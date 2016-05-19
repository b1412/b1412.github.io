---
layout: post
title: "Clojure Practice in 'Clojure Web Admin' - React-related projects in ClojureScript"
date: 2016-01-25 17:15:12 +0800
comments: true
categories: Clojure React Reagent 
---

[CWA](https://github.com/b1412/clojure-web-admin)(Clojure Web Admin) uses React and Bootstrap3  as a view layer with Reagent & Re-frame.

All the projects used in CWA are shwon as bellow:

- [Reagent](https://reagent-project.github.io/index.html "https://reagent-project.github.io/index.html")

  Reagent provides a minimalistic interface between ClojureScript and React.

- [Re-frame](https://github.com/Day8/re-frame "https://github.com/Day8/re-frame")

   Re-frame is a pattern for writing SPAs in ClojureScript, using Reagent with workflow as bellow:

```
app-db  -->  components  -->  Hiccup  -->  Reagent  -->  VDOM  -->  React  -->  DOM
 ^                                                                              |
 |                                                                              v
 handlers <-----  router  <-----------------------  (dispatch [event-id  event params])
```

- [React-Bootstrap](https://react-bootstrap.github.io "https://react-bootstrap.github.io")

    Bootstrap 3 components built with React.

- [Reagent-forms](https://github.com/reagent-project/reagent-forms "https://github.com/reagent-project/reagent-forms")

   A ClojureScript library to provide form data bindings for Reagent.
   In CWA, we have create some high-level abstract field types to make your life easier including Image ,Attachment, Enum, Lookup,etc. 

```clojurescript
(defmethod init-field :image
  [[_ {:keys [field id] :as attrs} :as component] {:keys [doc] :as opts}]
  (let [upload-path (reagent/atom
                     ((keyword (str (name id) ".path")) @doc))]
    (fn []
      [:div.form-group
            [:img {:src @upload-path :class "img-responsive"}]
            [show-on-click
             :trigger
             [Button {:bs-style "primary"}
              [FaIcon "upload"] ]
             :body [file-form
                    :image upload-path
                    :column-name id
                    :form-data doc]]])))

(defmethod bind :image
  [{:keys [field id fmt]} {:keys [get save! doc]}]
  {:value (let [value (or (get id) "")]
            value)
   :on-change #(save! id (->> % (value-of) (format-type field)))})

```


ClojureScript is basicly the same as Clojure, functional concepts in ClojureScript are a very good match for React's vision of components and immutability. And reagent could be your good choice to use React as a view layer in ClojureScript




Recently Bootstrap has been the bread and butter of rapid web prototyping, we better  make most of the Bootstrap ecosystem. Funtunately, React-Bootstrap has rewritten most Bootstrap components with React. In addition, we can easily wrap the react-bootstrap with Reagent and have access to all the goodies and  we can wrap any other js libraries as well.

```clojurescript
(def Button (reagent.core/adapt-react-class (aget js/ReactBootstrap "Button")))
(def TreeView (reagent.core/adapt-react-class js/TreeView))
```



With all the projects metioned above,front-end programming could become straightforward and fun!


