---
layout: post
title: "Clojure practice in 'Clojure Web Admin' - Incremental updates"
date: 2015-12-28 23:14:12 +0800
comments: true
categories: Clojure Korma
keywords: Clojure Korma
---
When assign resources to a role, multiple resources should be edited in one http request.


In back-end, we must tell  the new role-resources to create, the exist role-resources to update and the unuse role-resources to delete, which  could make create functiontoo complex.Acturally,it is a incremental updates solution, in [Clojure Web Admin]() we add fingerprint column in tables in which need 。 Besides, fingerprint can be used in big data update.

When using fingerprint, time of creating would increase due to the time of  fingerprint calculation, however time of updating would decrease because it only compare fingerprint not all the columns.

<!--more-->

```clojure
(defn assign-resources-to-role
  "Renew the permissions of the role:
     1 Get all the resources of current role.
     2 Delete the resources whose id is not in  params.
     3 Update the scope whose id is in db-resources  but scope is different.
     4 Create new resources in parmas whose id is not in the resources  for the role
  "
  [id param]
  (let [param (dissoc param :current-user :s-scope)
        db-ress (->> (-> (k/select* role-resource)
                         (k/where {:role_id id})
                         (k/select))
                     (map (juxt :resource-id :scope)))
        db-ids (->> db-ress
                    (map (fn [[r-id s]] r-id))
                    (set))
        param-ids (->> param
                       (map (fn [[r-id s]] (Integer/parseInt r-id)))
                       (set))
        del-data  (->> db-ress
                       (remove (fn [[r-id s]] (param-ids r-id))))
        exist-data (->> param
                        (filter (fn [[r-id s]]
                                  (let [curr (->> db-ress
                                                  (filter (fn [[db-r-id db-s]]
                                                            (= (Integer/parseInt r-id) db-r-id)))
                                                  (first))]
                                    (and (db-ids (Integer/parseInt r-id))
                                         (not= s (second curr)) )))))
        new-data (->> param
                      (remove (fn [[r-id s]] (db-ids (Integer/parseInt r-id)))))]
    (->> del-data
         (map (fn [[r-id s]] (-> (k/delete* role-resource)
                                 (k/where {:role_id id :resource_id r-id :scope s})
                                 (k/delete))))
         (doall))

    (->> new-data
         (map (fn [[r-id s]] [:resource-id r-id :scope s :role-id id]))
         (doall)
         (map (partial apply hash-map))
         (#(-> (k/insert* role-resource)
               (k/values %)
               (k/insert))))

    (->> exist-data
         (map (fn [[r-id s]]
                (-> (k/update* role-resource)
                    (k/set-fields {:scope s})
                    (k/where {:role_id id
                              :resource_id r-id})
                    (k/update))))
         (doall))))
```
```clojure
(defn assign-resources-to-role
  [id param]
  (fingerprint/init! role-resource {:role_id id})
  (->> (dissoc param :current-user :s-scope)
       (map (fn [[res-id s]] {:resource-id res-id :scope s :role-id id}))
       (map (partial create-entity role-resource))
       (doall))
  (fingerprint/remove-expired! role-resource))
```

Import code snippets in `create-entity`, see complete code [here](https://github.com/b1412/clojure-web-admin/blob/master/src/clj/clojure_web/common/crud.clj#L214)

If this entity has fingerprint column, invoke create function in fingerprint.clj, otherwise call Korma directly.
```clojure
(defn create-entity [entity params]
  (log/debug  "create " (:name entity) "  " params)
  (let [ create (fn[params](-> (k/insert* entity)
                     (k/values params)
                     (k/insert)))]

    (if (contains-column? entity :fingerprint)
      (fingerprint/create entity params create)
      (create params))))
```

For example

![DNS configuration](/images/role-resources-before.png)

![DNS configuration](/images/role-resources-after.png)


##Scenario



##Solution
