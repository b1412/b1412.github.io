---
layout: post
title: "Clojure practice in 'Clojure Web Admin' - Dasherize Korma"
date: 2015-12-28 22:23:19 +0800
comments: true
categories: Clojure Korma
keywords: Clojure  Korma
---




By default, Korma follows database convention using underscores  while Clojurians prefer kebab-case(lisp-case).

In this post, the function/macro used in [Clojure Web Admin](https://github.com/b1412/clojure-web-admin "https://github.com/b1412/clojure-web-admin") are dasherized by
[`alert-var-root`](https://clojuredocs.org/clojure.core/alter-var-root "https://clojuredocs.org/clojure.core/alter-var-root")

For others functions/macros, it could be done by code snippets in this post with minor changes.

Code in this post is [here](https://gist.github.com/b1412/f5d90e604a72ef88318f "https://gist.github.com/b1412/f5d90e604a72ef88318f")

<!--more-->

##defentity & select &  insert


```clojure
(def table->entity (atom {}))

(def underscore-kw (comp keyword #(clojure.string/replace % "-" "_") name))

(def dasherize-kw  (comp keyword #(clojure.string/replace % "_" "-") name))

(def map-keys (fn [f m] (reduce-kv #(assoc %1 (f %2) %3) {} m)))

(defmacro defent [table & body]
  `(do (declare ~table)
       (defentity ~table
         (prepare   (partial map-keys underscore-kw))
         (transform (partial map-keys dasherize-kw))
         (table (-> (:name (meta (var ~table)))
                      (clojure.string/replace #"-" "_")
                      (keyword))))))
```

```clojure
;;; Map  the table 'role_resource' in DB, get rid of `(table :role-resource :role_resource)`
(defent role-resource)

;;; Use kebab-case in where conditon and get column names with kebab-case keyword.
(select role-resource (limit 1))

; =>({:id 1, :role-id 1, :resource-id 1, :scope "system", :valid "yes", :fingerprint "f465893674d0f121bddb96698d6c55b5"})
```

##where

```clojure
(alter-var-root
 (var k/where*)
 (fn [f]
   #(f %1
       (walk/prewalk
        (fn [e]
          (if (:korma.sql.utils/generated e)
            (update e :korma.sql.utils/generated
                    (fn [k] (str/replace k "-" "_")))
            e)) %2))))
```

```
;;; Use kebab-case column names in where conditon
(select role-resource (where {:role-id 2}) (limit 1))
; =>({:id 72, :role-id 2, :resource-id 58, :scope "org", :valid "yes", :fingerprint "f20bafaae6285a3443cb6710421da306"})
```




##References


* http://camdez.com/blog/2015/07/21/dasherize-korma-columns
