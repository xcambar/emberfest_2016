class: middle, factory

# Migrating an app to Ember
## Component after Component

???

What I mean by that is "how not to do a big bang rewrite of an existing application",
and "how not to lose your sanity catching up with the existing features".

Because building features is cool, but rewriting features not so much.

And you (or your company) can not alwayd afford a rewrite.

A bit about me and how we got there.

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

* Manager: "Can you save our frontend?"
* **(Because of the context)** Me: "Yeah, sure, Ember all the things!"
* Step 1: new project, *ideal* context (**apetizers**)
    * We built the app **by the book**
    * it worked perfectly
* Step 2: Major app in the company
    * 7 years old
    * mostly **Server-side rendered** application
    * tons of features
    * Python developers **own and hate** frontend code

* After a **very quick** discussion with the product team, it was very clear that
* **you just can't start over**

---

class: middle, park

## Unfortunately
# You can't always
# start over

???

In our case,
* Market is very demanding
* Features are constantly landed

* The product must move on
* **The show must go on**
* The team already has a fast paced development
* Considering to rewrite everything frm scratch and build a 1:1 Ember version of the existing is **madness**

* Plus, we wanted an **early release** of the expected features

* So we needed another way to get Ember into this app, as there was no way it would be replaced.
* The thing is, it's not always obvious how you should proceed.

The following of this talk will provide some hopefully helpful tips
if you ever want or need to migrate an existing app to Ember.

### Epiphanies

Here are **a couple of golden rules** to remember for your migration.

* If we had those thought clear before we started, it would have guided us.
* Keep those epiphanies in mind when you need Ember to interact with the outside world
* (Works for addons as well)

---

class: middle, dinos

## Epiphany no 1
# The outside world is hostile

???

* Hostile JS
* **No RunLoop**! (we're civilized, aren't we?)
* Hostile DOM **that mutates unknowingly**
* Hostile build process
* You need a way to get an Ember app to deal with the outside world

* It's all about producing data and consuming data, right?
* **Produce data for** and **consume data from** the host app

* Don't expose more of your app than necessary!

---

class: middle, dinos

## Epiphany no 2
# Your app is just a component

???

You go component after component, so it's only natural that at first,
your app is just a component.

How do we handle data flow in an Ember app? **DDAU**

With this consideration in mind:
* **Implement DDAU** between your host app and your Ember app
* This will be covered later in the talk
* DDAU enables **a safe environment**

* You will start with **a slice of your app**, so:

**Limit your application.hbs to a single line as much as possible, and as long as you can**

---

class: middle, factory2

# Key topics
### for your migration

???

There are some questions that you will have to face while when performing a slow migration.

1. What part of the app will we migrate first?
2. How to properly bootstrap the app?
3. What's the communication workflow?
4. How to integrate the build process?

---

class: middle, factory2

## topic no 1
# Integration strategies

???

**What component should I start with?**

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

**We opted for the third strategy, because we're doing business software**

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

## A note on working on multiple pages

Thanks to the `visit` API, you can **work with the router early**.

---

class: middle, factory2

.center[<img src="Step 1.png"/>]

---

class: middle, factory2

.center[<img src="Step 2.png"/>]

---

class: middle, factory2

.center[<img src="Step 3.png"/>]

???

Data tables are hard, we kept then last

---

class: middle, factory2

.center[<img src="Step 4.png"/>]

???

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

Fortunately, **Ember provides the tools to bootstrap your app as specifically as needed**

* You can pass options and make it your state
* you can start programmatically
* you can route to a specific page

---

class: center, middle, factory2

.center[<img src="Step 0.png"/>]

???

This is the app in its initial form.
Worst case scenario: you don't have an API, you only have static HTML.
Let's consider that.

* Remove the code in the template that generates the UL/LI, SELECT...
* Turn that into a data structure
* **Static HTML becomes configuration** (selected facets...)

---

class: middle, factory2

.center[<img src="Init - pre.png"/>]

???

* Pass the configuration to your Ember app so it bootstraps with the correct state and voilà!

About static configuration VS APIs:

* bootstrap configuration is a temporary solution, because you don't have an API not the router ready yet.
* As the app evolves, it **may become an API** (and hopefully will).

Don't think too much about APIs or Router at first, it's a lot of work to handle and you already have your previous app doing that for you.
DRY! Reuse the existing, turn it upside down, make it yours!

Just build and use data.

---

class: center, middle, factory2

.center[<img src="Step 1.png"/>]

???

...And your app is started.

The green part is your Ember app. But how did it work, specifically?

---

class: middle, factory2, bigger1

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

???

location deactivates the Router

rootElement

APP is an object which data you can consume in services, components...

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

class: middle, factory2, bigger2

```js
// app/templates/application.hbs
{{ember-islands}}
```

```js
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

Your app starts in the correct state, but this state is more than likely going to change.

If your Ember app changes its state, it may want to inform the host app.

If the host app requires a change in your Ember app's state, it needs a way to do it.

Keep in mind that:

* Your app is a component in your host APP
* You want to expose a public API as small as possible

Let's try to DDAU this layer in our app.

---

class: middle, factory2

.center[<img src="DDAU.png"/>]

???

Actions up:
* events as actions
* A **Fire and Forget** strategy is great
* What I think is the best way to teach the outside world
* what to do after **is not the responsibility** of your Ember app.

Data down:
* methods as a means to update the data
* Minimal surface API
* Equivalent of `sendAction` and `{{action}}` for non-Ember things
* Control flow is maintained if you use Promises as return values



### Services as state
* Services allow to have **A single point of entry for data manipulation**
* You can **inject them** wherever you want
* Easily **replace configuration with an API** call when they're ready
* They're **the minimum surface of contact** you could have between your Ember app and the outside world

---

class: middle, factory2, top_title, bigger3

## Actions Up with events

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

---

class: middle, factory2, top_title, bigger0

## Data down with a public API

```js
// app/app.js
Ember.Application.extend({
  /*
    @returns Promise
  */
  resetFacets() {
    const container = this.__container__;
    return container.lookup('service:facets').reset();
  }
});
```

```js
// In the host application
theApp.resetFacets().then(showFlashMessage);
```

???

### Control flow preserved!

---

class: middle, factory2

## topic no 4
# Build process

???

* It's a hard topic
* Very dependent on host app

* We don't have a definitive answer.

General tip: Be careful.

---

class: middle, center, factory2

# Automation<br>is hard
### <small>*Watch your content hooks in ember-cli*</small>

???

### Ember-cli is amazing

* it makes a sdifficult task easy
* it's still open
* it's interoperable
* can be used with the CLI or programmatically via an API

### ember-cli is oriented towards single-page apps.

* Content hooks are hard to work with in this context
* Flesh out `index.html`?
* How to have multiple index.html?
* How to separate content hooks into multiple files?

### tl;dr;

* Find your own way in your own context
* If you have answers, come find me!
* ember-cli makes our lives so much easier
* I should invest more in ember-cli, hey Stef!

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

I'm very glad about the "Learning core team". If you have experience about migrations,
we should gather and write amazing documentation (Hey Ray, Hey Jade)!

---

class: middle, blue, fernsehturm

# Thank you!
## Xavier Cambar - @xcambar
.logo[![](./peopledoc.png)]

<!-- github twitter -->
