class: middle, factory

# Migrating an app to Ember
## Component after Component

---

class: middle, factory

# Xavier Cambar
## @xcambar
.logo[<img src="./peopledoc.png"/>]

???

* PeopleDoc is an HR Company
* We do Business Software
* Our apps are **model- and processes-heavy**
* **Definitively** ambitious!

* We're hiring!
* Remotely!

* Manager: "Ember all the things!"
* Step 1: new project, *ideal* context
    * We built the app **by the book**
* Step 2: Major app in the company
    * **Server-side rendered** application
    * Python developers **own and hate** frontend code

* When you are asked to handle such an app, that is **alive and well**
* **you just can't start over**

---

class: middle, park

## In the real world
# You just can't
# start over

???

* Market is very demanding
* Features are constantly landed

* The product must move on
* **The show must go on**
* The team already has a fast paced development
* Considering to rewrite everything frm scratch and build a 1:1 Ember version of the existing is **madness**

* Plus, we wanted an **early release** of the expected features

### Epiphanies

* **Ember is hard to learn**, it's no big news the learning curve is steep
* Probably for this reason, **the documentation supposes a friendly environment**

Here are **a couple of golden rules** to remember for your migration

---

class: middle, dinos

## Epiphany no 2
# The outside world is hostile

???

* Hostile JS
* **No RunLoop**!
* Hostile DOM **that mutates unknowingly**
* Hostile build process
* You need a way to get an Ember app to deal with the outside world
* **Produce data for** and **consume data from** the host app

* Don't expose more than necessary!

---

class: middle, dinos

## Epiphany no 1
# Your app is just a component

???

You will start with **a slice of your app**, so:
* Limit your `application.hbs` to a single component instanciation
* You'll thank me later when the app will grow

With this consideration in mind:
* **Implement DDAU** between your host app and your Ember app
* This will be covered later in the talk
* DDAU enables **a safe environment**

**Limit your application.hbs to a single line as much as possible**

---

class: middle, factory2

# Key topics
### for your migration

???
Some considerations about **questions you will face**

1. What part of the app will we migrate first?
2. How to properly bootstrap the app?
3. What's the communication workflow?
4. How to integrate the build process?

---

class: middle, factory2

## topic no 1
# Integration strategies

???

### The most **repeated** component
* Good to DRY up the code
* Good for design-heavy, UX-heavy apps
* Examples:
    * Currency conversion
    * Addresses

### The most **critical** component
* More **control on sensitive operations**
* Example:
    * Tax form submission component

### The component **the users use the most**
  * Compose a tweet

### The most **complex** component
* Builds a safety net around complex features
* Example:
    * No example, because you will come up with a more complex one and mine will look foolish

## So what's the plan?
* We opted for the third strategy, because we're doing business software

---

class: middle, factory2
.center[<img src="Step 0.png"/>]

???

### Feature run down

* Facets
    * Dynamic lists
    * **Dependent fields**
    * Values are updated (or even available) based on already selected values
    * **Counters** are live
* Search
    * **Autocomplete** field
    * **Dependent on the facets**
* Table
    * Filtering, Sorting, Add/Remove columns, crazy rendering, **classic**...

We started with the **facets**.

---

class: middle, factory2

.center[<img src="Step 1.png"/>]

---

class: middle, factory2

.center[<img src="Step 2.png"/>]

---

class: middle, factory2

.center[<img src="Step 3.png"/>]

---

class: middle, factory2

.center[<img src="Step 4.png"/>]

???

Thanks to the `visit` API, you can **work with the router early**.

After you're done with the components
* of a single page
* on many pages

You can **take over the History**.

Note: "take over history" has quite a nice ring to it :)

In our strategy, this is when we will **consider close to Done** for the app.
Not there yet...

---

class: middle, factory2

## topic no 2
# Bootstrapping

???

Because of **the hostile environment** and because of Ember bootstrapping defaults,
you want to make sure that you bootstrap:

* At the **right time**
* At the **right place**
* With the **right data**

**Ember provides the tools for that**

* You most likely don't have an API at hand.
* Start your Ember APP with what you have
* Don't require more than you have
* **data === state === context**

---

class: center, middle, factory2

.center[<img src="Step 0.png"/>]

???

This is the app in its current form.

* Remove the code in the template that generates the UL/LI, SELECT...
* Turn that into a data structure
* **Static HTML becomes configuration**

---

class: middle, factory2

.center[<img src="Init - pre.png"/>]

???

* Pass it to your Ember app in the configuration

* As the app evolves, it **may become an API**
* Don't build a new API at first, **it takes too much time**

---

class: center, middle, factory2

.center[<img src="Step 1.png"/>]

---

class: middle, factory2

```js
let app = MyApp.create({ autoboot: false });
  let options = {
    location: 'none',
    rootElement: '#location',
    APP: {
      facets: { /*...*/ }
    }
  };
  App.visit('/', options).then((/* appInstance */) => {
    console.log('Smooth');
  });
```

---

class: middle, factory2

## A few words on
# Ember Islands

???

**Fantastic** ember-cli addon by **Mitch Lloyd**

* it allows to render an app into multiple **DOM islands**
* it's like **multiple root elements**

* There's a Global Meetup talk about it

---

class: middle, factory2

.center[<img src="Step 1.png"/>]

---

class: middle, factory2

.center[<img src="Step 2.png"/>]

???

In our case, adding the Search without taking over the Data Table was not possible.

Ember Islands came to the rescue!

---

class: middle, factory2

```html
// app/templates/application.hbs

{{ember-islands}}

// your_host_page.html

<div data-component='my-first-component'></div>
<!-- More HTML -->
<div data-component='my-other-component'></div>
```

### <small>*Hint: Keep configuration in Javascript*</small>

???

* You declare the **name of the component** where you want it.
* You can also pass configuration, but I'd **avoid doing passing configuration**, I prefermy **config in the bootstrap JS code**.

---

class: middle, factory2

## topic no 3
# Communication

???

* Your app is a component in your host APP
* you don't want ot expose the guts of your app
* Internals are for your eyes only
* You can extend/simulate *Data down, actions up* with:
    * events as actions
    * methods as a means to update the data

---

class: middle, factory2

.center[<img src="DDAU.png"/>]

???

### Services as state
* Services allow to have **A single point of entry for data manipulation**
* You can **ibject them** wherever you want
* Easily **replace configuration with an API** call when they're ready
* They're **the minimum surface of contact** you could have between your Ember app and the outside world

---

class: middle, factory2, top_title

## Actions Up with  events

```js
// app/app.js
Ember.Application.extend(Ember.Evented);
```

```js
// app/instance-initializers/application-as-emitter.js
export default {
  name: 'application-as-emitter',
  initialize(instance) {
    const service = instance.lookup('service:facets');
    service.on('didUpdateFacets', (facets)=> {
      instance.application.trigger('updateFacets', facets);
    });
  }
};
```

```js
// In the host application
theApp.on('updateFacets', reloadDataTable);
```

???

* What I think is the best way to teach the outside world

* A **Fire and Forget** strategy is great
* what to do after **is not the responsibility** of your Ember app.

---

class: middle, factory2, top_title

## Data down with a public API

```js
// app/app.js
Ember.Application.extend({
  /*
    @returns Promise
  */
  resetFacets() {
    return this.__container__.lookup('service:facets').reset();
  }
});
```

```js
// In the host application
theApp.resetFacets().then(showFlashMessage);
```

???

* Minimal surface API
* Equivalent of `sendAction` and `{{action}}` for non-Ember things
* Control flow is maintained if you use Promises as return values

---

class: middle, factory2

## topic no 4
# Build process

???

* It's a hard topic
* Very dependent on host app

---

class: middle, center, factory2

# Automation<br>is hard
### Watch your content hooks in ember-cli

???

### ember-cli is oriented towards full-page Ember apps.

* Content hooks are hard to work with in this context
* Flesh out `index.html`?
* How to have multiple index.html?
* How to separate content hooks into multiple files?

### What we want:

* We try to avoid to build the Ember app locally
* We want to deliver the app to the host as a dependency
* Making a release for each PR is very heavy
* SemVer saves the day, but it remains tedious

* Copy/Paste is the current way to go.

### At the end of the day

* I should invest more in ember-cli
* I much prefer to work with ember-cli even if it's hard
* **ember-cli is awesome nonetheless**

---

class: middle, fernsehturm

## Conclusion
# Migrate to Ember!

### You have all the tools!

???


### you can migrate your already ambitious app to Ember
* **The tools for an easy migration are already there**
* Great opportunity to see Ember in more companies and projects

A lot of talks about migrations this year. **Let's write awesome documentation!**

---

class: middle, blue, fernsehturm

# Thank you!
## Xavier Cambar - @xcambar
.logo[![](./peopledoc.png)]

<!-- github twitter -->
