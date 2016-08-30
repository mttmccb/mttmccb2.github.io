---
title: "Using i18next language detection with Aurelia"
excerpt: "How to get setup with this handy plugin"
tags: [aurelia, i18n, i18next]
header:
  image: radio-detector-rover-min.jpg
  caption: "Photo credit: [**Pascal**](https://www.flickr.com/photos/pasukaru76/6832318941/in/photolist-bpKqor-9mPY7C-ou57Yj-ou74rZ-orZ3oo-oeZLXg-ow7JCb-owc2pD-ow8nVX-odX69N-JCnf79-ocMguU-fqoPCH-ocu9Un-ocFZUs-ou633z-fqoKgX-a1QS9b-otZhRL-dEruTv-otREqc-dEwTU7-odoa6y-ayaiAx-otM5Qs-a1MYSB-odK2gR-otDyzZ-oupkQU-oydLKn-ocAZpu-os69Ab-odiHy9-6XgE13-ouePDT-owb56T-otZj7G-odGE35-otNYc3-otJJhK-ou6S3g-otZpTb-orZ7w3-dCfw4H-otUWff-otMipW-ocxs3p-ocw6Sj-otEq2p-ocseZN)"
---

I'd been trying to add language detection to a multi-lingual [Aurelia](http://aurelia.io) project I'm working on. I'm already using [aurelia-i18n](https://github.com/aurelia/i18n) and have it setup like this...

```javascript
import 'some.css';
import {I18N} from 'aurelia-i18n';
import Backend from 'i18next-xhr-backend';

export async function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()

  aurelia.use.plugin('aurelia-i18n', (instance) => {
    instance.i18next.use(Backend);

    return instance.setup({
      backend: { 
        loadPath: './locales/{{lng}}/{{ns}}.json',
      },
      lng : 'cy',
      attributes : ['t','i18n'],
      fallbackLng : 'en-GB',
      debug : false
    });
  });

  await aurelia.start();
  aurelia.setRoot('shell/shell');
}
```

All this is doing is setting up the `Backend` to handle pulling in the relevant translation, at no point is there anything to detect the users preferred language so in this case it loads it up with Welsh (cy), you can then change it English but if you refresh the app it will revert back to Welsh.

So we have 3 main requirements

* Detect the users preferred language
* Set the users preferred language
* Save their choices

## First attempt

I started by replacing `lng: 'cy'` with the `getLanguage()` function below, because browsers are annoying, they don't store language preferences in the same way, some hold them in an array, others simply store them as a property in the `navigator` object. This might be standardised at some point but right now something like this would do the job.

```javascript
function getLanguage(){
  return window.navigator.languages ? window.navigator.languages[0] : (window.navigator.language || window.navigator.userLanguage) || 'en-GB';
}
```

But this didn't feel like a very good way to do it, so I [opened an issue](https://github.com/aurelia/i18n/issues/148) on the aurelia i18n repo as I couldn't find an obvious to do what I was looking for.

## Enter i18next-browser-languagedetector
[zewa666](https://github.com/zewa666) kindly pointed out that i18next has an [extensive plugin ecosystem](http://i18next.com/docs/ecosystem/), plugins are great, it's quickly becoming a feature I look for when selecting a library or tool.

Getting browser detection setup is ridiculously simple. Firstly, you need to add it as a dependency to your project, I use npm so this will do it for me:

```shell
npm install i18next-browser-languagedetector --save
```

Import it

```javascript
import LngDetector from 'i18next-browser-languagedetector';
```

And `use` it

```javascript
instance.i18next
      .use(Backend)
      .use(LngDetector);
```

aurelia-i18n then uses a `setup` rather than `init` to pass in the options, these options below at doing a few things, first they're setting the backend loadpath, where to load the translation json files from. `lng` is the language to select, which can be dropped as we will be using language detection. We can leave `attributes` and `debug`, but we want `fallbackLng` to be a language we have available.

```javascript
var options = {
  backend: { 
    loadPath: './locales/{{lng}}/{{ns}}.json',
  },
  lng : 'cy',
  attributes : ['t','i18n'],
  fallbackLng : 'en-GB',
  debug : false
};
```

Within these options we also need to setup detection, I'll be doing this by adding the following property to the options:

```javascript
detection: {
  order: ['localStorage', 'cookie', 'navigator'],
  lookupCookie: 'i18next',
  lookupLocalStorage: 'i18nextLng',
  caches: ['localStorage', 'cookie']
}
```

This setup means I'm using `localStorage` or a `cookie` to cache which language has been set. When it's first loaded there won't be anything in `localStorage` or a `cookie` so it will get this from the `navigator`, which is essentially the users browser settings.

The final setup looks like this and fulfills all of our original requirements. Do you still want to write it yourself?

```javascript
import 'some.css';
import {I18N} from 'aurelia-i18n';
import Backend from 'i18next-xhr-backend';
import LngDetector from 'i18next-browser-languagedetector';

export async function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()

  aurelia.use.plugin('aurelia-i18n', (instance) => {

    instance.i18next
      .use(Backend)
      .use(LngDetector);
        
    return instance.setup({
      backend: {
        loadPath: './locales/{{lng}}/{{ns}}.json',
      },
      detection: {
        order: ['localStorage', 'cookie', 'navigator'],
        lookupCookie: 'i18next',
        lookupLocalStorage: 'i18nextLng',
        caches: ['localStorage', 'cookie']
      },
      attributes : ['t','i18n'],
      fallbackLng : 'en-GB',
      debug : false
    });
  });

  await aurelia.start();
  aurelia.setRoot('shell/shell');

}
```

## Slight aside
While I wasn't expecting my first effort to write a language detection function to be perfect I wasn't far off, here's how `i18n-browser-languagedetector` does it. There's some better error checking and they get a standard interface as they're using different 'detectors'.

It's pretty clear if I continued I'd have reinventing the wheel and it might have been a bit oval.

```javascript
export default {
  name: 'navigator',

  lookup(options) {
    let found = [];

    if (typeof navigator !== 'undefined') {
      if (navigator.languages) { // chrome only; not an array, so can't use .push.apply instead of iterating
        for (let i=0; i < navigator.languages.length; i++) {
          found.push(navigator.languages[i]);
        }
      }
      if (navigator.userLanguage) {
        found.push(navigator.userLanguage);
      }
      if (navigator.language) {
        found.push(navigator.language);
      }
    }

    return found.length > 0 ? found : undefined;
  }
};
```