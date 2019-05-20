# clojure-patterns
Some Patterns I use when writing clojure that help

## NOTICE:
These are just guidelines. Do not treat them as hard rules. There will always be reasons to deviate away from this advice for one reason or another. Also take this with a grain of salt. I'm relatively inexperienced and these are just my thought of what has worked for be so far or what I believe will work. I garentee that some of this is incorrect, inacurrate and poor advice. Use your judgement. Only use this advice if it makes sense to you and don't just blindly follow the advice of some random person on the internet without knowing why.

TODO: Something about how good code is a good fit for semi experimental projects

TODO: Something about who this is targeted at

TODO: Something about what aspects of software are expensive.

TODO: Maybe highlight what is important to who

## General App Design

The software built will change often and sometimes in big ways. Avoid writing code that doesn't change easily. You will need to balance what you write so as not to over engineer and waste time writing code that will never change or be resused. But also still write code to a high enough standard to avoid wasting time maintaining and improving poor quality code.

Below are some things that's I've learned that I believe save me time.

### Reduce validation/typing

This applies for services of infrastructure, functions or modules of code. Any encapsulated unit of logic that has data piped though it.

Often you'll writing be modules/functions or services that have data piped through them sequentially. For an exagerated example ` Web Form -> API Gateway -> Authorization -> Primary Service (Backend) -> Auxiliary Service (notifications) -> Database.` Avoid validating the payload at each step. If you write and include validation (Even shared code) for each service and often codebase (multiple repos) then making an alteration to the validation this requires a change on every service. It's easy to forget to update one of the many services. Instead validate/sanitize the data between the untrusted service (such as frontend) and the trusted service only. Assume that data will be sent correctly from trusted services and have been validated at a prior stage. The benefits from rigerous validation are not worth dealing with the rigidity and fragility of an ever changing code base. You will likely deviate away from this when the code stops changing so frequently and you're sick of dealing with errors that could have been caught with validation. However if you prematurely introduce this you'll quickly get sick of making a single validation change across multiple services.

### Reduce automated testing

Reduce but do not eliminate. There are many forms of automated testing each with their own cost to implement and payoff. IMO unit testing is the cheapest form of automated testing and also provides the most value. I'd say unit test most things. A unit test either: a) invokes a pure function and asserts the result or b) mocks out side effects of an impure function making in pure, then invokes and asserts. Unit test as much as you can, and apply TDD. It is worth it in the short and long term. 

However one caveate. Only unit test an entrypoint/parent function. If `A` invokes `B` and `B` invokes `C` then only test `A`. `B` & `C` will already have test coverage from a well writted `A` unit test. Testing all three requires fixing many broken tests when making a single change. One change should really only break the unit tests for that function.

Avoid Integration, DB, e2e ui and all other forms of testing until you are bug fixing more than you are writing features for a project. All have their benefits but the trade off is not worth it during early stages unless usless reliablility take more precidence over time to implement.

### Avoid mapping data.

Allow your data to be as dynamic as possible. Avoid/reduce static rules for your data. TODO

### Avoid renaming

Get names right the first time TODO

### Don't change, add
TODO

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
