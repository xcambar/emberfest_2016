class: middle

# Migrating an app to Ember

### Component After Component

---

class: middle

# Xavier Cambar

# PeopleDoc

???

* 18 months in the company
* HR processes
* Business Software
* Model-heavy applications
* Processes- and workflow-heavy applications
* **Definitively** ambitious!

---

class: middle

# Backstory

???

* Allow me a bit of a back story.
* Where I was
* Where the company was
* What overall was the situation

---

class: middle

# Manager: "Ember all the things!"

---

class: middle

# Step 1: brand new app

### aka "The appetizers"

* Brand new project
* Ideal Ember context

???

* as we know and love
* follow the docs and rock

* Project was built
* pushed to production
* successfullly released
* and then...

---

class: middle

# Step 2: Biggest app in the company

### aka "OMFG"

* Market is very demanding
* Features are constantly landed

???

* Already added Ember to our stack
* 2 frontend developers
* Server-side rendered application
* Python developers hate frontend code
* For good reasons: frontend tooling in Django was awful (another story)

* Too many changes to be made
* Should we adapt?
* Should we rewrite?

* When it comes to the main/biggest app in the company

---

class: middle

# ...you just can't start over

### Zeno's Achilles paradox is real

.center[<img width="100%" src="Zeno_tortoise.png"/>]

???

* The product will move on
* No time to catch up
* Market is very sensitive to change
* No Big Bang rewrites
* Feature-full ETA: Too big to tell
* We wanted an early release

---

class: middle

# Epiphanies (from the trenches)

???

* Radical change from the typical (documented) Ember context
* There's a steep learning curve, but documentation supposes a friendly environment
* Difference between theory and practice

---

class: middle

# Epiphany Nr 1

## Components are the best, and your app is one

???

* Limit your `application.hbs` to a single component instanciation
* You'll thank me later when the app will grow
* Implement DDAU between your host app and your Ember app

---

class: middle

# Epiphany Nr 2

## Your Ember app lives in a hostile world

???

* We needed a way to get an Ember app to deal with the outside world
* Produce data for and consume data from the host app
* Hostile JS that doesn't care about the runloop (What the hell!)
* Hostile DOM
* Hostile build process

* Don't expose more than necessary!

---

class: middle

# Considerations for an easier life

1. What part of the app will we migrate first?
1. How to properly bootstrap the app?
1. What's the communication workflow?
1. How to integrate the build process?

<!--- 1. When to let the app manage data? --->
<!--- 1. When to migrate the API? --->
<!--- 1. How to build? --->

???

* Biggest consideration: Smooth transition
* Move fast, don't break
* Don't break things

---

class: middle

# What part of the app will we migrate first?

* The most repeated component?
* The most significant component?

???

* Good to DRY up the code
* Good for design-heavy, UX-heavy apps

* Good for critical parts of the application
* Good for most used parts of the application

* We opted for the second strategy, because we're doing business software

---

class: middle

# What's the plan?

.center[<img src="Step 0.png"/>]

???

At this point, your Ember App is nothing more than a component
in the host application.

It should be treated that way.

`application.hbs` should be a single line of code, calling your "main" component.

unless you're using Ember Islands, see later.

---

class: middle

# What's the plan?

.center[<img src="Step 1.png"/>]

---

class: middle

# What's the plan?

.center[<img src="Step 2.png"/>]

---

class: middle

# What's the plan?

.center[<img src="Step 3.png"/>]

---

class: middle

# What's the plan?

.center[<img src="Step 4.png"/>]

---

class: middle

# How to properly bootstrap the app?

* at the right time
* at the right place
* with the right data

???

* You most likely don't have an API at hand.
* Start your Ember APP with what you have
* Don't require more than you have
* Let the host application control the application flow
* data === context

---

class: center,middle

.center[<img src="Init - pre.png"/>]

---

class: center,middle

.center[<img src="Step 1.png"/>]

???

* Remove the code in the template that generates the UL/LI, SELECT...
* Turn that into a data structure
* Pass it to your Ember app in the configuration

* No New API required!

---

class: middle

.red[**Warning:**] Ember ≥ 2.3

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

class: middle

# Going further

.center[<img src="Step 1.png"/>]

---

class: middle

# Going further

.center[<img src="Step 2.png"/>]

---

class: middle

# Going further: Ember Islands

```html
// app/templates/application.hbs

{{ember-islands}}

// your_host_page.html

<div data-component='my-first-component'></div>
<!-- More HTML -->
<div data-component='my-other-component'></div>
```

???

* RTFM
* It's very quickly a requirement
* See Ember Meetup Global
* I prefer to leave the configuration to API and initial configuration

---

class: middle

# What's the communication workflow?

???

* Your app is a component in your host APP
* you don't want ot expose the guts of your app
* Internals are for your eyes only
* Data down, actions up

---

class: middle

# Data Down, Actions Up

.center[<img src="DDAU.png"/>]

---

class: middle

# Use Services for shared data

```js
// app/services/facets.js
export default Ember.Service.extend({
  all() {
    return this._allFromConfig() || this._allFromAPI();
  },
  /* ... */
});
```

???

* A single point of entry for data manipulation
* A minimum contact surface between your Ember app and the outside world
* Get rid of it as soon as possible

---

class: middle

# Actions Up: Emit events

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
* that your app did something
* Fire and forget, it's not your responsibility

---

class: middle

# Data down: Expose a simple API

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
* Keeps control flow at hand

---

class: middle

# (WIP) How to integrate the build process?

* Content hooks are amazing...ly binding
* Integrate ember-cli in the build pipeline of the host app?
* Don't build, only integrate the `/dist` folder?

???

* We try to avoid to build the Ember app locally
* Making a release for each PR is very heavy
* SemVer saves the day, but it remains tedious
* Copy/Paste is the current way to go.

* No silver lining

---

class: middle

# Ember integrates very well in existing apps

* Great opportunity for more teams and companies to give Ember a try
* Obvious use case for companies, yet non-obvious to the community
* Yet, considering the variability of contexts, how to make it more visible and accessible?


???

* Tooling and APIs are moving in the right direction
* Lack of documentation is on the community (aka, me)

