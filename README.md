# clojure-patterns
Some Patterns I use when writing clojure that help

## Re-Frame

### Routing 

Use Secretary and Accountant to help with routing. 

Routes can be defined like this:
```cljs
(defroute posts "/posts" []
  (rf/dispatch [:set-active-page :posts]))
  
(defroute post "/posts/:post-id" [post-id]
  (rf/dispatch [:set-active-page :post post-id]))
  
(defroute edit-post "/posts/:post-id/edit" [post-id]
  (rf/dispatch [:set-active-page :edit-post post-id]))
```

In this example the post-id is a value stored in the app db as the active post currently open. In order to link to the edit page from the view, the id must be known. To do this create a subscription to return the href for the active post.

```cljs
(rf/reg-sub
  :href/edit-post
  (fn [{:keys [:post-id]} _]
    (edit-post {:post-id post-id})))
```

And use that in the view.

```cljs
(defn edit-post-button []
  [:a.button {:href @(rf/subscribe [:href/edit-post])} "Edit Post"])
```

When no information is needed from the db for a link just reference the route directly in the view

```cljs
(defn all-post-button []
  [:a.button {:href (routes/posts)} "All Posts"])
```

### Buttons

When buttons are used for routing, use anchor html tags with a href. This allows the user to open the page in a new tab or window. A button with `on-click` does not have this behaviour

```cljs
(defn all-post-button []
  [:a.button {:href (routes/posts)} "All Posts"])
```
