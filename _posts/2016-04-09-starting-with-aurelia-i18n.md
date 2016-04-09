---
layout: post
title: "Starting with Aurelia i18n"
modified:
excerpt: "Posting about starting with Aurelia i18n."
tags: []
image:
  feature: sirhowy-river.jpg
  credit: Jim McSweeney
  creditlink: https://www.flickr.com/photos/73118846@N04/23900161566/in/photolist-4FYRxy-Eu9tGt-FjBR4n-CpYzn9-EptS4m-dkLb2H-4FYRvU-sHeR2-4FUF66-uEaH6G-tExfjY-vBaF5Z-tofp2U-tooXSt-6dSmd7-6dSqjm-EYwwaf-DWtw5e-6dSjfu-EHqWAX-bHMm6R-bHMwEn-wUBfuJ-6dShTS-dkLa77-Bc91ag-FczwFR-FaGaLF-EXKXXG-qAaeGA-B6mmoF-Ciygv5-fzTNgu-ot5gTM-yj64yS-xp5DDJ-yw9NZn-zGaWaw-oD7ZSq-AounKt-FgV3NZ-AnWc4p-CJasCH-sMWeQE-CpkHX3-bHMkd4-FggJXw-s437ED-Bfz5k7-wy7THM
comments: true
---

{% include _toc.html %}

As usual I wanted to jump right in the deep end with [Aurelia](http://aurelia.io). I'm in the processing building an app that'll rely quite heavily on translations so I wanted to test it out on a simply example.

So I found the not yet complete [i18n library](https://github.com/aurelia/i18n), but I also wanted to use it with [TypeScript](https://www.typescriptlang.org/) and [Webpack](https://webpack.github.io/). Sadly it's not that easy to do that as both of them are relatively new to the Aurelia community so there's not much support available. Although it's the end goal and I'll hopefully show how I got it working, when I get it working ;)

## Aurelia i18n on ES2016
Using [system.js](https://github.com/systemjs/systemjs) with [jspm](https://jspm.io/) is the most established way of working with Aurelia so I thought it would be best getting it working with that first before I tackle the stack I want to use.

To test it I came up with a basic plan

* Setup the latest ES2016 [skeleton navigation](https://github.com/aurelia/skeleton-navigation) repo
* Add a language switcher component
* Update the existing views and navigation to enable i18n translation
* Support 4 different translations, I went for
  * English - because I speak it
  * Welsh - because my daughters speaks a little
  * German - because it was the last language I tried to learn
  * Russian - because my wife knows a bit and it's uses a cyrillic script
  
If you want to skip to the final result it's in [my fork](https://github.com/mttmccb/skeleton-navigation/tree/i18n) of the skeleton navigation.

### Step 1 - Setup the latest ES2016 skeleton
The steps are in the README so I won't repeat them here but in summary you want to run the following

```shell
npm install
npm install -g gulp # (if you haven't done it already)
npm install -g jspm # (if you haven't done it already)
jspm install -y
```

It's worth checking things are working so run

```shell
gulp watch
```

And open [http://localhost:9000](http://localhost:9000) and make sure it works **before** you make any changes. Leave it running so we can see the changes as we make them.

### Step 2 - Add a language switcher component
At the moment we haven't created our language switcher component, we'll get to that shortly, it's useful to see how you'd use a component first.

Custom components in Aurelia are pretty handy they look like custom HTML elements but you build their structure and behaviour. I want the language switcher to be a `select` input in the nav bar so in the nav-bar.html I need to `require` it 

```html
<require from="language-switcher"></require>
```

and then use it.

```html
<language-switcher></language-switcher>
```

you can pass in information but the language switcher is self contained and uses the `EventAggregator` to interact with the rest of the application so we don't need anything else here. You can see the updated file below.

#### nav-bar.html (updated)
```html
<template bindable="router">
    <require from="language-switcher"></require>
    <nav class="navbar navbar-default navbar-fixed-top" role="navigation">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle" data-toggle="collapse" data-target="#skeleton-navigation-navbar-collapse">
                <span class="sr-only">Toggle Navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href="#">
                <i class="fa fa-home"></i>
                <span>${router.title}</span>
            </a>
        </div>

        <div class="collapse navbar-collapse" id="skeleton-navigation-navbar-collapse">
            <ul class="nav navbar-nav">
                <li repeat.for="row of router.navigation" class="${row.isActive ? 'active' : ''}">
                    <a data-toggle="collapse" data-target="#skeleton-navigation-navbar-collapse.in" href.bind="row.href" t.bind="row.settings.t">${row.title}</a>
                </li>
            </ul>
            <language-switcher></language-switcher>
            <ul class="nav navbar-nav navbar-right">
                <li class="loader" if.bind="router.isNavigating">
                    <i class="fa fa-spinner fa-spin fa-2x"></i>
                </li>
            </ul>
        </div>
    </nav>
</template>
```

The skeleton uses bootstrap so we need to use `navbar-form` to get it looking ok, I've also used `input-group` so we have a nice little globe icon next to the `select` input.

The Aurelia parts here are binding the value of the `select` input to `selectedLanguage` and building the options from the languages array. You'll see I've got a value and text, in this case the value will map to the language code.

#### language-switcher.html
```html
<template>
    <form class="navbar-form navbar-left" role="language">
        <div class="input-group">
            <span class="input-group-addon fa fa-globe"></span>
            <select value.bind="selectedLanguage" id="lg" class="form-control">
                <option repeat.for="language of languages" value.bind="language.value">${language.text}</option>
            </select>
        </div>
    </form>
</template>
```

At the moment there isn't much to the Javascript, it's just a class with a few properties which includes an array of the languages<sup>[1](#language-footnote)</sup> we're going to support and also the `selectedLanguage`, which we will make use of later.

#### language-switcher.js
```javascript
export class LanguageSwitcher {
    this.languages = [
        { value: 'en', text: 'English (Saesneg)'},
        { value: 'cy', text: 'Cymru (Welsh)'},
        { value: 'ru', text: 'Pусский (Russian)'},
        { value: 'de', text: 'Deutsche (German)'},
    ];
    this.selectedLanguage = 'en';
}
```

If you left your browser open and `gulp watch` running then you should now see a `select` input with 4 options on the left of the navigation menu.

### Step 3 - Add in Aurelia i18n
At this point we have just done standard Aurelia although we've added a *currently* pointless component.

Next we need to add i18n, refer to the [aurelia i18n repo](https://github.com/aurelia/i18n) for the latest way to do this but I did it like this

* ed the plugin via `jspm` with following command

```shell
    jspm install aurelia-i18n
```
* Updated to [manual bootstrapping](http://aurelia.io/docs#startup-and-configuration). This means updating the `index.html` adding this to the body tag:

```html
    <body aurelia-app="main">
```
* Created a `locales` folder in the project root
* For each locale created a new folder with it's name (e.g. `en`, `de`, ...)
* In those subfolders created a file named `translation.json` which contains your language specific translations. The en one looks like this, this contains all the strings that we're going to translate:

```javascript
    {
        "welcome": "Welcome",
        "github_users": "Github Users",
        "child_router": "Child Router",
        "title": "Welcome to the Aurelia Navigation App",
        "first_name": "First Name",
        "last_name": "Last Name",
        "full_name": "Full Name",
        "submit": "submit",
        "user_leaving_page": "Are you sure you want to leave",
        "contact": "Contact"
    }
```
* Installed the [XHR Plugin](https://github.com/i18next/i18next-xhr-backend) with the following command

```shell
    jspm install npm:i18next-xhr-backend
```
* Updated `main.js` in our `src` folder with following content:

#### main.js (updated)
```javascript
    import {I18N} from 'aurelia-i18n';
    import XHR from 'i18next-xhr-backend'; // <-- your previously installed backend plugin 

    export function configure(aurelia) {
        aurelia.use
            .standardConfiguration()
            .developmentLogging()
            .plugin('aurelia-i18n', (instance) => {
            // register backend plugin
            instance.i18next.use(XHR);
            
            // adapt options to your needs (see http://i18next.com/docs/options/)
            instance.setup({
                backend: {                                  // <-- configure backend settings
                loadPath: '/locales/{{lng}}/{{ns}}.json', // <-- XHR settings for where to get the files from
                },
                lng : 'en',
                attributes : ['t','i18n'],
                fallbackLng : 'cy',
                debug : false
            });
        });

        aurelia.start().then(a => a.setRoot());
    }
```

### Step 4 - Update the Welcome view
There isn't much involved in updating the Welcome view model, we've just extended from the `BaseI18N` class which takes care of injecting `i18n` and `EventAggregator`, setting up the `attached()` method to update the translation as well as subscribing to an event which does the same when the locale changes. It's definitely worth understanding what this does and understanding how to use a `constructor` if you need to. You'll need to use `super` but I haven't got my head around how this works but it's not needed below.

#### welcome.js (updated)
```javascript
import {BaseI18N} from 'aurelia-i18n';

export class Welcome extends BaseI18N {
    heading = 'Welcome to the Aurelia Navigation App!';
    firstName = 'John';
    lastName = 'Doe';
    previousValue = this.fullName;

    get fullName() {
        return `${this.firstName} ${this.lastName}`;
    }

    submit() {
        this.previousValue = this.fullName;
        alert(`${this.i18n.tr('welcome')}, ${this.fullName}!`);
    }

    canDeactivate() {
        if (this.fullName !== this.previousValue) {
            return confirm(`${this.i18n.tr('user_leaving_page')}?`);
        }
    }
}
```

Updating the HTML is involves add a `t` attribute with the translation key to all the relevant elements. Nothing too scary here.

#### welcome.html (updated)
```html
<template>  
    <section class="au-animate">
        <h2 t="title">Welcome to the Aurelia Navigation App!</h2>
        <form role="form" submit.delegate="submit()">
        <div class="form-group">
            <label for="fn" t="first_name">First Name</label>
            <input type="text" value.bind="firstName" class="form-control" id="fn" placeholder="first name">
        </div>
        <div class="form-group">
            <label for="ln" t="last_name">Last Name</label>
            <input type="text" value.bind="lastName" class="form-control" id="ln" placeholder="last name">
        </div>
        <div class="form-group">
            <label t="full_name">Full Name</label>
            <p class="help-block">${fullName | upper}</p>
        </div>
        <button type="submit" class="btn btn-default" t="submit">Submit</button>
        </form>
    </section>
</template>
```

### Step 5 - Update the User View Model
I took a slighty different approach for the Users view model in that I'm being the work of injecting `i18n`, I haven't setting up the events or attach method but as we'll see later it works, I'm not sure why yet.

#### users.js (updated)
```javascript
import {inject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';
import 'fetch';
import {I18N} from 'aurelia-i18n';

@inject(HttpClient, I18N)
export class Users {
    heading = 'Github Users';
    users = [];

    constructor(http, i18n) {
        this.i18n = i18n;
        http.configure(config => {
            config
                .useStandardConfiguration()
                .withBaseUrl('https://api.github.com/');
        });

        this.http = http;
    }

    activate() {
        return this.http.fetch('users')
            .then(response => response.json())
            .then(users => this.users = users);
    }
}
```

There isn't much to the HTML, I've just added the `t` attributes.

### Step 6 - Update the Routers
The routers were a little tricky initially but I was reminded by [K. Scott Allen](http://odetocode.com/blogs/scott/archive/2016/02/22/binding-aurelia-routing-rules.aspx) about using the `settings` property to `RouteConfig` entry. So that seemed like the obvious way to setup the translation too. Apart from that all we're doing is injecting `i18n`.

#### app.js (updated)
```javascript
import {I18N} from 'aurelia-i18n';
import {inject} from 'aurelia-framework';

@inject(I18N)
export class App {
    constructor(i18n) {
        this.i18n = i18n;
    }
    
    configureRouter(config, router) {
        config.title = 'Aurelia';
        config.map([
            { route: ['', 'welcome'], name: 'welcome',      moduleId: 'welcome',      nav: true, title: 'Welcome', settings: { t:'welcome' } },
            { route: 'users',         name: 'users',        moduleId: 'users',        nav: true, title: 'Github Users', settings: { t:'github_users' }},
            { route: 'child-router',  name: 'child-router', moduleId: 'child-router', nav: true, title: 'Child Router', settings: { t:'child_router' }}
        ]);
        
        this.router = router;
    }
}
```

Updating the HTML is pretty straightforward we just need to add `t.bind="row.settings.t"` to bind the value we've setup for each navigation item.

#### nav-bar.html (extract)
```html
...
<ul class="nav navbar-nav">
    <li repeat.for="row of router.navigation" class="${row.isActive ? 'active' : ''}">
        <a data-toggle="collapse" data-target="#skeleton-navigation-navbar-collapse.in" href.bind="row.href" t.bind="row.settings.t">${row.title}</a>
    </li>
</ul>
...
```

The `child-router` is almost identical so I won't repeat that here.

### Step 6 - Enable the language switcher
So far we've done all the donkey work to so when we do change locale all the relevent strings are translated. Now we need to inject `i18n` into the language switcher and setup a way to switch languages. As you can see from the code that's exactly what I've done, the `switchLanguage` method uses the i18n `setLocale` method to set it to the locale selected. In the background this will raise the 'changed locale' event which will be picked up by our view models.

#### language-switcher.js (updated)
```javascript
import {inject} from 'aurelia-framework';
import {I18N} from 'aurelia-i18n';

@inject(I18N)
export class LanguageSwitcher {
    constructor(i18n) {
        this.i18n = i18n;
    }
    
    this.languages = [
        { value:'en', text:'English (Saesneg)'},
        { value:'cy', text: 'Cymru (Welsh)'},
        { value:'ru', text: 'Pусский (Russian)'},
        { value:'de', text: 'Deutsche (German)'},
    ];
    
    selectedLanguage = 'en';

    switchLanguage() {
        this.i18n.setLocale(this.selectedLanguage);
    }
}
```

## Gotcha, Issues and workarounds
- Don't forget that adding a `t` attribute will replace the `textContent`, I forgot to include the punctuation and this was clear as it was in the translation file.
- Loading multiple language didn't work initially with aurelia-i18n 0.5.0 but this was fixed in 0.5.1, you can see the hack in a previous commit if you really want.
    
## Improvements
This only shows off a small part of i18n, I didn't get into number or date formats or different ways it can be implemented, when I make more use of those features I'll be sure to share it on this blog.

## Conclusion
I'm quite happy with how easy it was to get i18n setup in Aurelia, it doesn't overly complicate the skeleton, but I can see how it might in some cases. Where possible I'd try to use `BaseI18N` and inherit from that.

<small><a name="language-footnote">1</a>: If you're wondering why the language in brackets is translated into english except english which is translated into welsh it's because I started with the first too. Saesneg looks a lot like sassenach which is seem as derogatory but it just mean english in welsh or gaelic.</small>
