# Intro to React Query

> Note: This workshop handles the legacy React Query. In the foreseeable future, I would like to migrate it to [Tanstack Query for React](https://tanstack.com/query/latest/docs/framework/react/overview).

In this workshop, we are going to walk through a few simple concepts of [React Query](https://react-query.tanstack.com) which help you to **fetch**, **cache**, **synchronize** and **update** server state.

The examples here are going to cover the most common use-cases, however, if you are willing to learn these concepts more deeply, don't hesitate to visit the [official documentation](https://react-query.tanstack.com/overview) that has an amazing coverage of the whole API.

## Pre-requisites

You should be pretty comfortable with React and understand its limitation when it comes to handling HTTP requests.

Although there are some proven techniques and common best-practices, it's not easy to handle things like caching and data synchronization without of support of an external library.

**React Query** is here to help to make these things very straightforward.

## Project Initialization

We are going to initialize the project with **Snowpack** and its React Template.

Although, **React Query** was written in TypeScript and has an **amazing TypeScript support**, this material is going to use plain JavaScript to help us focusing more on the core React Query concepts.

**Tailwind** (with **PostCSS**) is going to be used to make our project look nice.

Because **React Query** is closely tight with server-side fetching and mutations, we are going to use some backend APIs during our demo.

- `Punk API` - Brewdog's [unofficial API](https://punkapi.com/documentation/v2) for listing favorite products from its brewery. We are going to use this API for most of the examples.
- `Deno Notes App` - Our Deno REST API [created for our workshops](https://github.com/ovotech/co_frontend/tree/master/deno-notes-app) will be used for mutations examples.

> Run the Deno API in a separated terminal by typing: `denon start`

To bootstrap the project, just type (in terminal):

```bash
npx create-snowpack-app react-query-intro --template @snowpack/app-template-react
```

### Installing dependencies

Another important step is to install **React Query**. Routing and some loading indications might come handy for our examples as well, therefore let's install **React Router** and **SVG Loaders React** as well.

Data fetching could be handled in various way (by `fetch` or any specific library). In our examples, we are going to use **Axios** for data fetching.

In the terminal (in the project folder), just type:

```bash
# install React Query, Axios, SVG Loader, and React Router for web.
npm install --save react-query axios react-router-dom svg-loaders-react
```

For the styling ([Tailwind](https://tailwindcss.com) + [PostCSS](https://postcss.org)), let's install a few more packages by typing the following:

```bash
# install tailwind, postcss and autoprefixer, @snowpack/plugin-postcss and cssnano@latest
npm install --save-dev tailwindcss postcss autoprefixer @snowpack/plugin-postcss cssnano
```

### Creating/Updating config files.

In this subsection, we are going to configure our CSS tooling.

The first step is to initialize a tailwind config. Let's do it typing the following command in your terminal:

```bash
npx tailwindcss init -p
```

This creates two files: `tailwind.config.js` and `postcss.config.js` files which will contain the following code:

```javascript
// tailwind.config.js
module.exports = {
  purge: [],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
};
```

```javascript
// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

In the `tailwind.config.js`, we can configure the **purge** option. This allows us to automatically remove unused styles in production.

```diff
--- a/tailwind.config.js
+++ b/tailwind.config.js
// tailwind.config.js
module.exports = {
- purge: [],
+ purge: ['./public/index.html', './src/**/*.{js,jsx}'],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
-   extend: {},
+   extend: {
+     opacity: ['disabled'],
+     backgroundColor: ['disabled'],
+     cursor: ['disabled'],
+   }
  },
  plugins: [],
}
```

The final version of the `tailwind.config.js` will look as the following:

```javascript
// tailwind.config.js
module.exports = {
  purge: ['./public/index.html', './src/**/*.{js,jsx}'],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {
      opacity: ['disabled'],
      backgroundColor: ['disabled'],
      cursor: ['disabled'],
    },
  },
  plugins: [],
};
```

Another change is going to happen in `postcss.config.js` where we are going to replace the original config with the following update.

```diff
--- a/postcss.config.js
+++ b/postcss.config.js
// postcss.config.js
+const tailwind = require('tailwindcss');
+const autoprefixer = require('autoprefixer');
+const cssnano = require('cssnano');
+
+const plugins =
+  process.env.NODE_ENV === 'production'
+    ? [tailwind, autoprefixer, cssnano]
+    : [tailwind, autoprefixer];
+
-module.exports = {
-  plugins: {
-    tailwindcss: {},
-    autoprefixer: {},
-  },
-}
+module.exports = { plugins };
```

And the final version of the `postcss.config.js` file now looks like the code below:

```javascript
// postcss.config.js
const tailwind = require('tailwindcss');
const autoprefixer = require('autoprefixer');
const cssnano = require('cssnano');

const plugins =
  process.env.NODE_ENV === 'production'
    ? [tailwind, autoprefixer, cssnano]
    : [tailwind, autoprefixer];

module.exports = { plugins };
```

We also need to tweak the `snowpack.config.mjs` and utilize the `@snowpack/plugin-postcss`. For the routing, we also need to uncomment the disabled support of routing fallback.

```diff
--- a/snowpack.config.mjs
+++ b/snowpack.config.mjs
// snowpack.config.mjs
/** @type {import("snowpack").SnowpackUserConfig } */
export default {
  mount: {
    public: { url: '/', static: true },
    src: { url: '/dist' },
  },
- plugins: ['@snowpack/plugin-react-refresh', '@snowpack/plugin-dotenv'],
+ plugins: [
+ '@snowpack/plugin-react-refresh',
+ '@snowpack/plugin-dotenv',
+ '@snowpack/plugin-postcss',
+],
  routes: [
    /* Enable an SPA Fallback in development: */
-   // {"match": "routes", "src": ".*", "dest": "/index.html"},
+   {"match": "routes", "src": ".*", "dest": "/index.html"},
  ],
  optimize: {
    /* Example: Bundle your final build: */
    // "bundle": true,
  },
  packageOptions: {
    /* ... */
  },
  devOptions: {
    /* ... */
  },
  buildOptions: {
    /* ... */
  },
};
```

The updated file will look as the following:

```javascript
// snowpack.config.mjs
/** @type {import("snowpack").SnowpackUserConfig } */
export default {
  mount: {
    public: { url: '/', static: true },
    src: { url: '/dist' },
  },
  plugins: [
    '@snowpack/plugin-react-refresh',
    '@snowpack/plugin-dotenv',
    '@snowpack/plugin-postcss',
  ],
  routes: [
    /* Enable an SPA Fallback in development: */
    { match: 'routes', src: '.*', dest: '/index.html' },
  ],
  optimize: {
    /* Example: Bundle your final build: */
    // "bundle": true,
  },
  packageOptions: {
    /* ... */
  },
  devOptions: {
    /* ... */
  },
  buildOptions: {
    /* ... */
  },
};
```

Now it is time to include `tailwind` in the main css. Let's update a file called `index.css` in the `src` folder and add the following content.

```diff
--- a/index.css
+++ b/index.css
+/* src/index.css */
-body {
-  margin: 0;
-  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
-    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
-    sans-serif;
-  -webkit-font-smoothing: antialiased;
-  -moz-osx-font-smoothing: grayscale;
-}
-
-code {
-  font-family: source-code-pro, Menlo, Monaco, Consolas, 'Courier New',
-    monospace;
-}
+
+@tailwind base;
+@tailwind components;
+@tailwind utilities;
```

So the final `index.css` file will look as the following:

```css
/* src/index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Now, the `React`, `Snowpack` and `Tailwind` are configured and we should be able to utilize it easily. Let's update the `App.jsx` and replace it by the following content:

```javascript
// App.jsx
import React from 'react';

function App() {
  return (
    <div className="App">
      <h1 className="p-4 m-4 bg-blue-300">Hello World</h1>
    </div>
  );
}

export default App;
```

> Feel free to remove the following additional files from `src` folder: `App.css` and `App.test.jsx` by typing `rm src/{App.css,App.test.jsx,logo.svg}` in your terminal. We don't need them for our examples

If you now run the app by `npm run start`, you should see a simple Hello World in a blue box.

## Add helpers and basic components

In this section, we are going to add some functionality for our application. This will be used as base for our example in order to save time (in next sections, we are going to add some functionality related to React Query).

Things we are going to add now are related to `http methods` and `components`.

### Helper functions

In this part, we are going to add our functions for manipulation with our APIs.

To be able to demonstrate all important parts of **React Query**, we need to be able to delay requests. For that reason, let's create a `src/api` folder and add `utils.js` into it with the following code:

```javascript
// src/api/utils.js
export const wait =
  (ms = 0) =>
  (fn) =>
    new Promise((resolve) => setTimeout(() => resolve(fn), ms));
```

The main helpers we are going to add are going to be related to the `Beer API` showing lots of Brewdog's products. They also contain pagination which might come handy later. Let's add `brewdog-punk-api.js` into `src/api` with the following code:

```javascript
// src/api/brewdog-punk-api.js
import axios from 'axios';
import { wait } from './utils';

const BASE_URL = 'https://api.punkapi.com/v2';

const instance = axios.create({
  baseURL: BASE_URL,
});

export const getBeerList =
  (delayInMs = 0) =>
  async (page, perPage = 25) =>
    instance
      .get(
        page
          ? `${BASE_URL}/beers?page=${page}&per_page=${perPage}`
          : `${BASE_URL}/beers`,
      )
      .then(wait(delayInMs));

export const getBeerById =
  (delayInMs = 0) =>
  async (beerId) =>
    instance.get(`${BASE_URL}/beers/${beerId}`).then(wait(delayInMs));

export const getListOfBeersByName =
  (delayInMs = 0) =>
  async (beerName = '') =>
    instance
      .get(`${BASE_URL}/beers?beer_name=${beerName.replace(' ', '_')}`)
      .then(wait(delayInMs));

/* 
  These mock data are coming from https://punkapi.com/documentation/v2.
*/

export const punkBeers = [
  {
    id: 63,
    name: 'Sunk Punk',
    tagline: 'Ocean Fermented Lager.',
    description:
      "It's rumoured just a drop can calm the fiercest of storms. A balance of sweet, salt and savoury, citrus, spruce and caramel. Fermented at the bottom of the North Sea, which just so happens to be the perfect temperature for lagers to ferment.",
    image_url: 'https://images.punkapi.com/v2/63.png',
  },
  {
    id: 95,
    name: 'Peroxide Punk',
    tagline: 'Zesty Pale Ale.',
    description:
      'A trashy blonde concession for those who felt that our original 6% Punk IPA recipe was too hard hitting. This was also the first time we experimented with dry hopping our beers, giving Peroxide Punk a depth of flavour that belies its modest ABV. Zesty, aromatic and thirst quenching.',
    image_url: 'https://images.punkapi.com/v2/keg.png',
  },
  {
    id: 98,
    name: 'Pumpkin King',
    tagline: 'Spicy Citrus Pumpkin Ale.',
    description:
      "We're turning Hallowe’en inside out and upside down. Pumpkin King is not your usual unctuous, cloyingly sweet Hallowe’en pumpkin ale. Sure, there’s a huge heady hit of pungent spice on the nose, but it’s followed with bright and zesty citrus flavours, and a light mouthfeel. Spicy and sweet autumnal favourites like toasted marshmallow and toffee apple are just some of the complex notes you’ll find in our twisted take on a pumpkin ale, which weighs in at 5.4% ABV.",
    image_url: 'https://images.punkapi.com/v2/98.png',
  },
];

const blondeBeers = [
  {
    id: 2,
    name: 'Trashy Blonde',
    tagline: "You Know You Shouldn't",
    description:
      'A titillating, neurotic, peroxide punk of a Pale Ale. Combining attitude, style, substance, and a little bit of low self esteem for good measure; what would your mother say? The seductive lure of the sassy passion fruit hop proves too much to resist. All that is even before we get onto the fact that there are no additives, preservatives, pasteurization or strings attached. All wrapped up with the customary BrewDog bite and imaginative twist.',
    image_url: 'https://images.punkapi.com/v2/2.png',
  },
  {
    id: 270,
    name: 'Blonde Export Stout',
    tagline: 'Beatnik Brewing Collective 2017.',
    description:
      'Our Equity Punk gypsy brewery - Beatnik Brewing Collective - voted for this recipe as their 2017 annual brew. A blonde stout brewed to export strength, the stout character comes from extra ingredients instead of dark malts.',
    image_url: null,
  },
  {
    id: 277,
    name: 'Prototype Blonde Ale',
    tagline: 'Mellow Golden Ale.',
    description:
      'This Blonde Ale is a new recipe and uses a new yeast - so it is a true Prototype. Originally lined up as part of the 2017 Prototype Challenge it was released shortly afterwards. but hey, these things happen.',
    image_url: null,
  },
];

const pilsnerBeers = [
  {
    id: 4,
    name: 'Pilsen Lager',
    tagline: 'Unleash the Yeast Series.',
    description:
      'Our Unleash the Yeast series was an epic experiment into the differences in aroma and flavour provided by switching up your yeast. We brewed up a wort with a light caramel note and some toasty biscuit flavour, and hopped it with Amarillo and Centennial for a citrusy bitterness. Everything else is down to the yeast. Pilsner yeast ferments with no fruity esters or spicy phenols, although it can add a hint of butterscotch.',
    image_url: 'https://images.punkapi.com/v2/4.png',
  },
  {
    id: 111,
    name: 'Vagabond Pilsner',
    tagline: 'Hoppy Pilsner.',
    description:
      'Take the thirst-quenching crispness of a German Pilsner and combine it with lemon and honey to produce a rewarding modern twist on a beer classic.',
    image_url: 'https://images.punkapi.com/v2/111.png',
  },
  {
    id: 301,
    name: 'Small Batch: Dry-hopped Pilsner',
    tagline: 'Dry-hopped lager.',
    description:
      "A BrewDog bar exclusive draft lager, brewed with Weihenstephan's lager yeast, and dry-hopped with the contemporary German variety Saphir; this lager has been lightly centrifuged and packaged at just under 28 days in tank.",
    image_url: 'https://images.punkapi.com/v2/keg.png',
  },
];
```

Still in `src/api` directory, let's create another file `notes.js` with the following content:

```javascript
// src/api/notes.js
import axios from 'axios';
import { wait } from './utils';

const BASE_URL = 'http://localhost:8000';

const instance = axios.create({
  baseURL: BASE_URL,
});

export const getNotes =
  (delayInMs = 0) =>
  async () =>
    instance.get(`${BASE_URL}/notes`).then(wait(delayInMs));

export const getNoteById =
  (delayInMs = 0) =>
  async (noteId) =>
    instance.get(`${BASE_URL}/notes/${noteId}`).then(wait(delayInMs));

export const addNote =
  (delayInMs = 0) =>
  async (note) =>
    instance
      .post(`${BASE_URL}/notes`, JSON.stringify(note), {
        headers: { 'Content-Type': 'application/json' },
      })
      .then(wait(delayInMs));

export const updateNote =
  (delayInMs = 0) =>
  async (note) =>
    instance
      .put(`${BASE_URL}/notes/${note.id}`, JSON.stringify(note), {
        headers: { 'Content-Type': 'application/json' },
      })
      .then(wait(delayInMs));

export const removeNote =
  (delayInMs = 0) =>
  async (id) =>
    instance
      .delete(`${BASE_URL}/notes/${id}`, {
        headers: { 'Content-Type': 'application/json' },
      })
      .then(wait(delayInMs));

export const mockNotes = [
  {
    id: 1,
    title: 'My first note',
    description: 'This is a description of the first note',
    color: 'green',
  },
  {
    id: 2,
    title: 'Second note',
    description: 'This is a description of the second note',
    color: 'red',
  },
  {
    id: 3,
    title: 'Third note',
    description: 'This is a description of the third note',
    color: 'yellow',
  },
];
```

### Basic Components

In this part, we are going to add component structure and a basic interactions. It will help us to save some time to just React Query functionality later as we won't need to write everything from scratch. These will go under `components` folder (need to create it first) within `src` directory.

#### Loading.jsx

```javascript
// src/components/Loading.jsx
import React from 'react';
import { Bars } from 'svg-loaders-react';

export const Loading = () => (
  <div className="flex items-center justify-center h-screen">
    <Bars stroke="#000000" fill="#000000" />
  </div>
);
```

#### Error.jsx

```javascript
// src/components/React.jsx
import React from 'react';

export const Error = ({ handleRetry, message }) => (
  <div className="bg-black text-white">
    <div className="flex h-screen">
      <div className="m-auto text-center">
        <div>
          <svg
            width="450"
            height="338"
            viewBox="0 0 450 338"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
          >
            <g clipPath="url(#clip0)">
              <path
                d="M321.749 231.118C321.749 261.706 304.982 272.387 284.299 272.387C263.616 272.387 246.849 261.706 246.849 231.118C246.849 200.529 284.299 161.616 284.299 161.616C284.299 161.616 321.749 200.529 321.749 231.118Z"
                fill="#E7D040"
              />
              <path
                d="M283.318 242.108L299.28 210.437L283.378 238.092L283.55 226.581L294.552 203.668L283.596 223.535L283.906 202.832L295.687 184.591L283.955 199.577L284.149 161.616L282.983 209.717L271.054 189.914L282.839 213.776L281.723 236.897L281.69 236.283L267.883 215.36L281.648 238.452L281.508 241.343L281.483 241.387L281.495 241.624L278.664 272.478H282.446L282.9 269.985L296.632 246.95L282.935 267.707L283.318 242.108Z"
                fill="#3F3D56"
              />
              <path
                d="M450 132.766C450 191.197 417.971 211.599 378.462 211.599C338.952 211.599 306.923 191.197 306.923 132.766C306.923 74.3342 378.462 0 378.462 0C378.462 0 450 74.3342 450 132.766Z"
                fill="#F2F2F2"
              />
              <path
                d="M375.855 202.661L376.587 153.759L407.079 93.2606L376.702 146.088L377.032 124.099L398.046 80.3294L377.119 118.281V118.281L377.712 78.7339L400.214 43.8875L377.804 72.5153L378.175 0L375.849 95.9971L376.04 92.0369L353.161 54.0563L375.673 99.6388L373.541 143.806L373.478 142.634L347.103 102.665L373.398 146.775L373.131 152.299L373.083 152.382L373.106 152.835L367.697 264.891H374.923L375.79 207.012L402.021 163.01L375.855 202.661Z"
                fill="#3F3D56"
              />
              <path
                d="M447.063 255.107C447.063 272.547 342.747 338 220.802 338C98.8564 338 0 304.126 0 286.685C0 269.245 96.43 289.975 218.375 289.975C340.321 289.975 447.063 237.667 447.063 255.107Z"
                fill="#3F3D56"
              />
              <path
                opacity="0.1"
                d="M447.063 255.107C447.063 272.547 342.747 338 220.802 338C98.8564 338 0 304.126 0 286.685C0 269.245 96.43 289.975 218.375 289.975C340.321 289.975 447.063 237.667 447.063 255.107Z"
                fill="black"
              />
              <path
                d="M447.063 255.107C447.063 272.547 342.747 318.264 220.802 318.264C98.8564 318.264 0 304.126 0 286.685C0 269.245 96.43 270.238 218.375 270.238C340.321 270.238 447.063 237.667 447.063 255.107Z"
                fill="#3F3D56"
              />
              <path
                opacity="0.1"
                d="M223.273 297.864C275.057 297.864 317.037 294.343 317.037 289.999C317.037 285.655 275.057 282.133 223.273 282.133C171.488 282.133 129.509 285.655 129.509 289.999C129.509 294.343 171.488 297.864 223.273 297.864Z"
                fill="black"
              />
              <path
                d="M148.992 132.382C148.992 132.382 176.442 132.934 171.359 138.171C166.276 143.409 147.213 138.171 147.213 138.171L148.992 132.382Z"
                fill="#A0616A"
              />
              <path
                d="M91.7996 68.7669C91.7996 68.7669 102.883 66.6204 106.736 76.3238C110.59 86.0273 118.504 113.701 118.504 113.701C118.504 113.701 139.683 128.905 143.536 128.406C147.388 127.906 153.973 128.925 152.765 133.483C151.557 138.041 152.094 143.21 149.313 143.843C146.532 144.477 142.109 138.43 138.724 141.342C135.339 144.255 105.171 133.617 103.234 131.413C101.297 129.209 85.2021 81.4807 85.2021 81.4807C85.2021 81.4807 85.846 68.6114 91.7996 68.7669Z"
                fill="#E7D040"
              />
              <path
                d="M115.57 273.656L116.84 285.233L106.166 288.266L100.065 285.233V275.034L115.57 273.656Z"
                fill="#A0616A"
              />
              <path
                d="M86.8481 281.099L88.1188 292.676L77.4439 295.708L71.3438 292.676V282.477L86.8481 281.099Z"
                fill="#A0616A"
              />
              <path
                d="M116.586 147.13L121.415 166.15L128.532 204.742C128.532 204.742 130.565 214.114 126.499 225.416C122.432 236.718 115.061 273.656 116.586 275.861C118.111 278.066 99.3028 279.72 99.5569 277.515C99.8111 275.31 102.353 244.988 102.353 244.988C102.353 244.988 105.911 218.249 107.69 214.941C109.469 211.633 99.5569 203.364 99.5569 203.364C99.5569 203.364 97.0153 235.064 93.2027 237.821C89.3902 240.577 89.8985 283.304 88.1193 283.855C86.3402 284.406 73.3776 286.612 71.8526 283.855C70.3276 281.099 71.8526 235.064 71.8526 235.064C71.8526 235.064 80.4943 200.056 78.7151 195.094C76.9359 190.132 77.9526 168.907 77.9526 168.907C77.9526 168.907 71.3442 152.643 74.9026 144.098L116.586 147.13Z"
                fill="#2F2E41"
              />
              <path
                d="M115.824 41.2784C115.824 41.2784 105.149 55.6125 105.657 61.9525C106.165 68.2926 88.8818 50.375 88.8818 50.375C88.8818 50.375 101.336 32.1818 101.336 28.3226C101.336 24.4634 115.824 41.2784 115.824 41.2784Z"
                fill="#A0616A"
              />
              <path
                d="M115.696 46.6537C124.961 46.6537 132.471 38.5083 132.471 28.4604C132.471 18.4126 124.961 10.2672 115.696 10.2672C106.431 10.2672 98.9209 18.4126 98.9209 28.4604C98.9209 38.5083 106.431 46.6537 115.696 46.6537Z"
                fill="#A0616A"
              />
              <path
                d="M73.421 136.818C72.6611 139.889 71.7181 143.37 71.8528 146.303C71.9393 148.202 76.588 149.448 83.0159 150.264C88.9762 151.022 96.4665 151.408 103.276 151.607C110.425 151.816 116.818 151.816 119.891 151.816C129.041 151.816 121.67 145.476 118.111 142.444C114.553 139.412 115.316 96.4095 115.57 90.8964C115.824 85.3832 113.282 69.3952 113.282 66.363C113.282 63.3308 107.302 55.4167 107.302 55.4167C107.302 55.4167 105.911 59.7473 100.32 53.9586C94.728 48.1698 87.3571 46.5159 87.3571 46.5159C81.7654 48.7211 72.3612 72.9788 71.8528 77.1136C71.6469 78.7923 71.8172 87.39 72.1629 97.2061C72.6636 111.554 73.538 128.515 74.1403 130.315C74.6156 131.735 74.0895 134.116 73.421 136.818Z"
                fill="#E7D040"
              />
              <path
                d="M100.573 152.919L102.352 159.81L95.4895 167.529L93.2021 156.778L100.573 152.919Z"
                fill="#A0616A"
              />
              <path
                d="M102.861 282.201C102.861 282.201 108.198 288.817 112.265 284.406C116.332 279.996 116.078 277.239 116.84 277.239C117.603 277.239 129.04 288.266 129.04 288.266C129.04 288.266 148.611 293.227 137.936 296.811C127.261 300.394 97.0149 297.914 97.0149 296.811C97.0149 295.708 96.2524 281.099 99.0483 281.099L102.861 282.201Z"
                fill="#2F2E41"
              />
              <path
                d="M74.1401 289.644C74.1401 289.644 79.4776 296.26 83.5443 291.849C87.611 287.439 87.3568 284.682 88.1193 284.682C88.8819 284.682 100.319 295.708 100.319 295.708C100.319 295.708 119.89 300.67 109.215 304.254C98.5402 307.837 68.2942 305.356 68.2942 304.254C68.2942 303.151 67.5317 288.541 70.3275 288.541L74.1401 289.644Z"
                fill="#2F2E41"
              />
              <path
                d="M96.8624 42.9832C96.3205 43.3629 95.615 43.7477 95.0724 43.3691C93.055 37.9126 91.5471 32.2505 90.5713 26.4677C90.2287 24.4365 89.9909 22.1784 91.0314 20.449C91.4418 19.767 92.0325 19.2092 92.3331 18.4629C92.7883 17.3327 92.4887 16.0244 92.6399 14.802C92.9567 12.2411 95.1889 10.4549 97.4334 9.58867C99.678 8.7224 102.116 8.45368 104.255 7.31697C106.287 6.23763 107.914 4.44759 109.798 3.08922C111.682 1.73085 114.095 0.802081 116.193 1.71997C118.049 2.53199 119.205 4.58363 120.959 5.62826C122.149 6.33715 123.531 6.53441 124.859 6.83871C128.552 7.6885 132.02 9.43558 135 11.9485C135.726 12.5096 136.351 13.2087 136.846 14.0119C139.097 18.0314 135.067 23.4962 136.845 27.7854L133.001 24.4956C131.974 23.5217 130.816 22.7237 129.567 22.1291C128.294 21.606 126.786 21.5429 125.629 22.3221C124.006 23.4156 123.566 25.7002 122.999 27.6793C122.432 29.6584 121.178 31.8531 119.267 31.8024C116.669 31.7335 115.761 27.8012 113.429 26.5563C111.909 25.7447 109.996 26.2792 108.7 27.4628C107.405 28.6464 106.627 30.3637 106.057 32.0937C105.701 33.1748 105.366 34.3935 104.734 35.3307C104.035 36.3642 103.143 36.4474 102.397 37.2192C100.456 39.2284 99.2631 41.3011 96.8624 42.9832Z"
                fill="#2F2E41"
              />
              <path
                opacity="0.1"
                d="M73.4202 136.818C76.5796 141.788 80.1405 146.915 83.0151 150.264C88.9754 151.022 96.4657 151.408 103.275 151.607C102.058 149.818 100.537 148.298 98.7939 147.13C95.4897 144.925 87.6105 118.738 87.6105 118.738C87.6105 118.738 97.2689 91.7233 99.8106 81.5241C102.352 71.3248 92.4397 65.5361 92.4397 65.5361C87.8647 61.4012 79.9854 71.0492 79.9854 71.0492C79.9854 71.0492 75.9645 84.2034 72.1621 97.2061C72.6628 111.554 73.5372 128.515 74.1395 130.315C74.6148 131.735 74.0887 134.116 73.4202 136.818Z"
                fill="black"
              />
              <path
                d="M91.6775 63.3308C91.6775 63.3308 101.59 69.1196 99.0484 79.3188C96.5067 89.5181 86.8483 116.532 86.8483 116.532C86.8483 116.532 94.7276 142.72 98.0317 144.925C101.336 147.13 105.911 152.367 102.353 155.124C98.7943 157.881 96.2526 162.291 93.7109 160.913C91.1692 159.535 91.1692 151.816 86.8483 151.816C82.5275 151.816 64.9898 123.148 64.7357 120.116C64.4815 117.084 79.2233 68.8439 79.2233 68.8439C79.2233 68.8439 87.1025 59.196 91.6775 63.3308Z"
                fill="#E7D040"
              />
              <path
                d="M297.797 138.675L294.938 135.574C293.27 133.766 291.008 132.749 288.649 132.749C286.291 132.749 284.029 133.766 282.361 135.574L226.899 195.725L171.437 135.574C169.769 133.766 167.507 132.749 165.148 132.749C162.79 132.749 160.527 133.766 158.86 135.574L156 138.675C155.174 139.571 154.519 140.634 154.072 141.805C153.626 142.975 153.396 144.229 153.396 145.496C153.396 146.762 153.626 148.016 154.072 149.187C154.519 150.357 155.174 151.42 156 152.316L211.462 212.466L156 272.617C155.174 273.513 154.519 274.576 154.072 275.746C153.626 276.917 153.396 278.171 153.396 279.437C153.396 280.704 153.626 281.958 154.072 283.128C154.519 284.299 155.174 285.362 156 286.257L158.86 289.359C160.527 291.167 162.79 292.184 165.148 292.184C167.507 292.184 169.769 291.167 171.437 289.359L226.899 229.208L282.361 289.359C284.029 291.167 286.291 292.184 288.649 292.184C291.008 292.184 293.27 291.167 294.938 289.359L297.797 286.257C299.465 284.449 300.402 281.995 300.402 279.437C300.402 276.879 299.465 274.426 297.797 272.617L242.335 212.466L297.797 152.316C299.465 150.507 300.402 148.054 300.402 145.496C300.402 142.937 299.465 140.484 297.797 138.675Z"
                fill="#FF6584"
              />
              <path
                opacity="0.1"
                d="M154.021 275.887L207.837 217.523L207.319 216.961L156.001 272.617C155.141 273.549 154.467 274.662 154.021 275.887Z"
                fill="black"
              />
              <path
                opacity="0.1"
                d="M155.234 140.631C156.902 138.822 159.164 137.806 161.522 137.806C163.881 137.806 166.143 138.822 167.811 140.631L223.273 200.782L278.735 140.631C280.403 138.822 282.665 137.806 285.024 137.806C287.382 137.806 289.644 138.822 291.312 140.631L294.172 143.732C295.398 145.063 296.239 146.753 296.591 148.596C296.943 150.438 296.79 152.352 296.151 154.102L297.798 152.316C298.624 151.42 299.279 150.357 299.726 149.187C300.173 148.016 300.403 146.762 300.403 145.496C300.403 144.229 300.173 142.975 299.726 141.804C299.279 140.634 298.624 139.571 297.798 138.675L294.938 135.574C293.271 133.765 291.009 132.749 288.65 132.749C286.291 132.749 284.029 133.765 282.361 135.574L226.899 195.725L171.437 135.574C169.769 133.765 167.507 132.749 165.149 132.749C162.79 132.749 160.528 133.765 158.86 135.574L156.001 138.675C155.141 139.607 154.467 140.72 154.021 141.946L155.234 140.631Z"
                fill="black"
              />
              <path
                opacity="0.1"
                d="M242.853 213.028L238.709 217.523L294.171 277.674C295.398 279.004 296.239 280.695 296.59 282.538C296.942 284.38 296.789 286.294 296.15 288.044L297.797 286.257C298.623 285.362 299.278 284.299 299.725 283.128C300.172 281.958 300.402 280.704 300.402 279.437C300.402 278.171 300.172 276.917 299.725 275.746C299.278 274.576 298.623 273.513 297.797 272.617L242.853 213.028Z"
                fill="black"
              />
              <path
                d="M64.5802 253.134C64.5802 278.539 50.6547 287.41 33.4766 287.41C16.2986 287.41 2.37305 278.539 2.37305 253.134C2.37305 227.729 33.4766 195.41 33.4766 195.41C33.4766 195.41 64.5802 227.729 64.5802 253.134Z"
                fill="#E7D040"
              />
              <path
                d="M32.6622 262.262L45.9195 235.958L32.7122 258.927L32.8555 249.366L41.9922 230.336L32.8934 246.836L33.1509 229.642L42.9346 214.492L33.1913 226.939L33.3522 195.41L32.3843 235.36L22.4768 218.913L32.2643 238.731L31.3377 257.934L31.3102 257.425L19.8428 240.047L31.2752 259.225L31.1593 261.627L31.1385 261.663L31.1481 261.86L28.7965 287.485H31.9385L32.3155 285.415L43.72 266.284L32.3439 283.523L32.6622 262.262Z"
                fill="#3F3D56"
              />
            </g>
            <defs>
              <clipPath id="clip0">
                <rect width="450" height="338" fill="white" />
              </clipPath>
            </defs>
          </svg>
        </div>
        <p className="text-sm md:text-base text-yellow-300 pt-5 mb-4">
          {message ? message : 'Oops, something went wrong!'}
        </p>
        {handleRetry && (
          <button
            onClick={handleRetry}
            className="bg-transparent hover:bg-yellow-300 text-yellow-300 hover:text-white rounded shadow hover:shadow-lg py-2 px-4 border border-yellow-300 hover:border-transparent"
          >
            Retry
          </button>
        )}
      </div>
    </div>
  </div>
);
```

#### BeerDetail.jsx

```javascript
// src/components/BeerDetail.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../api/brewdog-punk-api';

export const BeerDetail = () => {
  const { id } = useParams();
  const history = useHistory();

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
      error: null,
      status: 'pending',
    },
  );

  React.useEffect(() => {
    setState({ status: 'pending' });

    getBeerById(1000)(id)
      .then((response) =>
        setState({ beer: response.data[0], status: 'success' }),
      )
      .catch((error) => setState({ error: error.message, status: 'error' }));
  }, [id]);

  const { beer, error, status } = state;

  if (status === 'pending') {
    return <Loading />;
  }

  if (status === 'error') {
    return <Error message={error} />;
  }

  return (
    <section className="text-gray-700 body-font overflow-hidden bg-white">
      <div className="container px-5 py-24 mx-auto">
        <div className="lg:w-4/5 mx-auto flex flex-wrap">
          <img
            alt={beer?.name}
            className="w-40 h-80 object-cover object-center rounded border border-gray-200"
            src={beer?.image_url}
          />
          <div className="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
            <h2 className="text-sm title-font text-gray-500 tracking-widest">
              {beer?.tagline}
            </h2>
            <h1 className="text-gray-900 text-3xl title-font font-medium mb-1">
              {beer?.name}
            </h1>
            <p className="leading-relaxed">{beer?.description}</p>
            <div className="flex mt-4">
              <button
                className="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
                onClick={() => history.push('/')}
              >
                Back
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
};
```

#### BeerList.jsx

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: null,
      error: null,
      status: 'pending',
    },
  );

  const getAllBeers = async () => {
    setState({ status: 'pending' });
    return getBeerList(1000)()
      .then((response) => setState({ beers: response.data, status: 'success' }))
      .catch((error) => setState({ error: error.message, status: 'error' }));
  };

  const getBeerListByType = async (type) => {
    setState({ status: 'pending' });
    return getListOfBeersByName(1000)(type)
      .then((response) => setState({ beers: response.data, status: 'success' }))
      .catch((error) => setState({ error: error.message, status: 'error' }));
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers, error, status } = state;

  if (status === 'pending') {
    return <Loading />;
  }

  if (status === 'error') {
    return <Error message={error} />;
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

#### NoteForm.jsx

```javascript
// src/components/NoteForm.jsx
import React from 'react';

export const NoteForm = ({ onNoteAdd, isDisabled }) => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      id: Math.random().toString(36).substring(7),
      title: '',
      description: '',
      color: 'green',
    },
  );

  const handleSubmit = async (event) => {
    event.preventDefault();
    return onNoteAdd(state);
  };

  return (
    <div className="flex items-center bg-gray-50 dark:bg-gray-900">
      <div className="container mx-auto">
        <div className="max-w-md mx-auto my-10 bg-white p-5 rounded-md shadow-sm">
          <div className="text-center">
            <h1 className="my-3 text-3xl font-semibold text-gray-700 dark:text-gray-200">
              Add a new note
            </h1>
            <p className="text-gray-400 dark:text-gray-400">
              Fill up the form below to create a new note.
            </p>
          </div>
          <div className="m-7">
            <form onSubmit={handleSubmit}>
              <div className="mb-6">
                <label
                  htmlFor="name"
                  className="block mb-2 text-sm text-gray-600 dark:text-gray-400"
                >
                  Title
                </label>
                <input
                  id="name"
                  type="text"
                  name="name"
                  placeholder="John Doe"
                  className="w-full px-3 py-2 placeholder-gray-300 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-indigo-100 focus:border-indigo-300 dark:bg-gray-700 dark:text-white dark:placeholder-gray-500 dark:border-gray-600 dark:focus:ring-gray-900 dark:focus:border-gray-500"
                  onChange={(event) => setState({ title: event.target.value })}
                />
              </div>
              <div className="mb-6">
                <label
                  htmlFor="name"
                  className="block mb-2 text-sm text-gray-600 dark:text-gray-400"
                >
                  Color
                </label>
                <select
                  className="w-full border bg-white rounded px-3 py-2 outline-none"
                  onChange={(event) => setState({ color: event.target.value })}
                >
                  <option value="" disabled>
                    Select color
                  </option>
                  <option value="green">Green</option>
                  <option value="red">Red</option>
                  <option value="yellow">Yellow</option>
                </select>
              </div>
              <div className="mb-6">
                <label
                  htmlFor="message"
                  className="block mb-2 text-sm text-gray-600 dark:text-gray-400"
                >
                  Your Description
                </label>

                <textarea
                  rows="5"
                  name="message"
                  id="message"
                  placeholder="Your Message"
                  className="w-full px-3 py-2 placeholder-gray-300 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-indigo-100 focus:border-indigo-300 dark:bg-gray-700 dark:text-white dark:placeholder-gray-500 dark:border-gray-600 dark:focus:ring-gray-900 dark:focus:border-gray-500"
                  onChange={(event) =>
                    setState({ description: event.target.value })
                  }
                ></textarea>
              </div>
              <div className="mb-6">
                <button
                  type="submit"
                  className="w-full px-3 py-4 text-white bg-indigo-500 rounded-md focus:bg-indigo-600 focus:outline-none"
                >
                  Send Message
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  );
};
```

#### NoteList.jsx

```javascript
// src/components/NoteList.jsx
import React from 'react';
import { getNotes, addNote } from '../api/notes';
import { Loading } from './Loading';
import { Error } from './Error';
import { NoteForm } from './NoteForm';

export const NoteList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      notes: null,
      error: null,
      status: 'pending',
    },
  );

  const getAllNotes = async () => {
    setState({ status: 'pending' });
    return getNotes(1000)()
      .then((response) => setState({ notes: response.data, status: 'success' }))
      .catch((error) => setState({ error: error.message, status: 'error' }));
  };

  const addNewNote = async (note) => {
    setState({ status: 'pending' });
    return addNote(1000)(note)
      .then((response) =>
        setState((current) => ({
          notes: [...current, note],
          status: 'success',
        })),
      )
      .catch((error) => setState({ error: error.message, status: 'error' }));
  };

  const handleNoteAdd = async (note) => {
    await addNewNote(note);
    setState({ notes: [...state.notes, note], status: 'success' });
  };

  React.useEffect(() => {
    getAllNotes();
  }, []);

  const { status, error, notes } = state;

  if (status === 'pending') {
    return <Loading />;
  }

  if (status === 'error') {
    return <Error message={error} />;
  }

  return (
    <div className="w-11/12 md:w-4/5 lg:w-1/2 mx-auto">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full px-4 mt-4">
          <h2 className="text-4xl font-semibold text-black">
            Mutation test with Notes API
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's test basics of mutation with a simple API written in Deno.
          </p>
        </div>
      </div>

      <NoteForm onNoteAdd={handleNoteAdd} />

      <div className="bg-white shadow-md rounded my-6">
        <table className="text-left w-full border-collapse">
          <thead>
            <tr>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Id
              </th>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Title
              </th>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Color
              </th>
            </tr>
          </thead>
          <tbody>
            {notes.map((note) => (
              <tr className="hover:bg-grey-lighter" key={note.id}>
                <td className="py-4 px-6 border-b border-grey-light">
                  {note.id}
                </td>
                <td className="py-4 px-6 border-b border-grey-light">
                  {note.title}
                </td>
                <td className="py-4 px-6 border-b border-grey-light">
                  <span
                    className={`rounded py-1 px-3 text-xs font-bold bg-${note.color}-400`}
                  >
                    {note.color}
                  </span>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};
```

#### App.jsx

Once the components are added, we need to update the `App.jsx` within the `src` directory.

```diff
--- a/App.jsx
+++ b/App.jsx
// App.jsx
import React from 'react';
+import {
+  BrowserRouter as Router,
+  Switch,
+  Route,
+  Redirect,
+} from 'react-router-dom';
+import { BeerDetail } from './components/BeerDetail';
+import { BeerList } from './components/BeerList';
+import { NoteList } from './components/NoteList';

function App() {
  return (
-   <div className="App">
-     <h1 className="p-4 m-4 bg-blue-300">Hello World</h1>
-   </div>
+   <Router>
+     <Switch>
+       <Route exact path="/beers" component={BeerList} />
+       <Route path="/beers/:id" component={BeerDetail} />
+       <Route path="/notes" component={NoteList} />
+       <Redirect from="/" to="/beers" />
+     </Switch>
+   </Router>
  );
}

export default App;
```

The latest snapshot of the `App.jsx` looks as the following:

```javascript
// App.jsx
import React from 'react';
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Redirect,
} from 'react-router-dom';
import { BeerDetail } from './components/BeerDetail';
import { BeerList } from './components/BeerList';
import { NoteList } from './components/NoteList';

function App() {
  return (
    <Router>
      <Switch>
        <Route exact path="/beers" component={BeerList} />
        <Route path="/beers/:id" component={BeerDetail} />
        <Route path="/notes" component={NoteList} />
        <Redirect from="/" to="/beers" />
      </Switch>
    </Router>
  );
}

export default App;
```

## Basics (keys, data fetching, status/variables, dev tools, option params, core concepts)

In this section, we are going to cover the basic concepts. We learn how to **initialize React Query**, **set a key**, **fetch data** and **see the status** and **progress in dev tools**.

We also spend some time to discuss the core concepts (the difference between `fresh`, `fetching`, `stale` and `inactive`) and how we can customize the values and when data are re-fetched (e.g. during transition from inactive, during browser tab revisiting).

We need to **initialize React Query on a global level**. Let's also **add Dev tools** at the same time.

```diff
--- a/App.jsx
+++ b/App.jsx
import React from 'react';
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Redirect,
} from 'react-router-dom';
import { BeerDetail } from './components/BeerDetail';
import { BeerList } from './components/BeerList';
import { NoteList } from './components/NoteList';
+import { QueryClient, QueryClientProvider } from 'react-query';
+import { ReactQueryDevtools } from 'react-query/devtools';

+export const queryClient = new QueryClient();

function App() {
  return (
+   <QueryClientProvider client={queryClient}>
      <Router>
        <Switch>
          <Route exact path="/beers" component={BeerList} />
          <Route path="/beers/:id" component={BeerDetail} />
          <Route path="/notes" component={NoteList} />
          <Redirect from="/" to="/beers" />
        </Switch>
      </Router>
+     <ReactQueryDevtools />
+   </QueryClientProvider>
  );
}

export default App;
```

Now we can utilize the React Query functionality in our component first.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
+import { useQuery } from 'react-query';

export const BeerList = () => {
+ const [beerType, setBeerType] = React.useState('all');
+ const beerListResult = useQuery(
+   'beers',
+   () => getBeerList(1000)().then((result) => result.data)
+ );
- const [state, setState] = React.useReducer(
-   (state, action) => ({ ...state, ...action }),
-   {
-     beers: null,
-     error: null,
-     status: 'pending',
-   },
- );

  const getAllBeers = async () => {
+   setBeerType('all');
-   setState({ status: 'pending' });
-   return getBeerList(1000)()
-     .then((response) => setState({ beers: response.data, status: 'success' }))
-     .catch((error) => setState({ error: error.message, status: 'error' }));
  };

  const getBeerListByType = async (type) => {
+   setBeerType(type);
-   setState({ status: 'pending' });
-   return getListOfBeersByName(1000)(type)
-     .then((response) => setState({ beers: response.data, status: 'success' }))
-     .catch((error) => setState({ error: error.message, status: 'error' }));
  };

- React.useEffect(() => {
-   getAllBeers();
- }, []);

- const { beers, error, status } = state;

- if (status === 'pending') {
+ if (beerListResult.isLoading) {
    return <Loading />;
  }

- if (status === 'error') {
+ if (beerListResult.isError)
-   return <Error message={error} />;
+   return <Error message={beerListResult.error.toString()} />
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

> You can alternative use `status` instead of boolean indicators. `status === 'loading'` instead of `isLoading`.

The difference between `isLoading` and `isFetching` is cache. If there is no cached data, `isLoading` is triggered, otherwise `isFetching` takes a place.

There are some important concepts which are crucial to understand. `fresh`, `fetching`, `stale` and `inactive`.

> When data is fetched, they are marked as `fresh`. At that point, they can't be re-fetched until they reach `stale` state. `By default, the transition between fresh and stale is immediate`, as `staleTime is set to 0`. You can change it to a custom value, including `Infinity` which prevent to make such a transition.

`If the component with React Query is unmounted` (e.g. visiting a different page which doesn't use that query), the query transitions its state into `inactive` one and stays there until `cacheTime` kicks in (then it is garbage collected). You can set up `cacheTime` value as well. The value is in ms. The following code show some of the custom settings:

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    'beers',
-   () => getBeerList(1000)().then((result) => result.data)
+   () => getBeerList(1000)().then((result) => result.data),
+   {
+     staleTime: 2000,
+     cacheTime: 2000,
+   },
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

If you would like to understand this a bit more, I recommend to check out this article: [https://codemachine.dev/things-i-learned-while-using-react-query-part-1](https://codemachine.dev/things-i-learned-while-using-react-query-part-1) or the [official documentation](https://codemachine.dev/things-i-learned-while-using-react-query-part-1).

The latest state of the `App.jsx` and `BeerList.jsx` look as the following:

```javascript
// App.jsx
import React from 'react';
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Redirect,
} from 'react-router-dom';
import { BeerDetail } from './components/BeerDetail';
import { BeerList } from './components/BeerList';
import { NoteList } from './components/NoteList';
import { QueryClient, QueryClientProvider } from 'react-query';
import { ReactQueryDevtools } from 'react-query/devtools';

export const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Router>
        <Switch>
          <Route exact path="/beers" component={BeerList} />
          <Route path="/beers/:id" component={BeerDetail} />
          <Route path="/notes" component={NoteList} />
          <Redirect from="/" to="/beers" />
        </Switch>
      </Router>
      <ReactQueryDevtools />
    </QueryClientProvider>
  );
}

export default App;
```

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    'beers',
    () => getBeerList(1000)().then((result) => result.data),
    {
      staleTime: 2000,
      cacheTime: 2000,
    },
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return <Error message={beerListResult.error.toString()} />;
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

## React Query Keys in more details

In this section we are going to have a closer look at the query key mechanism. You have probably spotted a `number associated with you key` within your dev tools. No matter how much queries are going to make, the number won't change as long as the query key is not unique. Let's see that in action.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    'beers',
    () => getBeerList(1000)().then((result) => result.data),
    {
      staleTime: 2000,
      cacheTime: 2000,
    },
  );

+ useQuery(
+   'beers',
+   () => getBeerList(1000)().then((result) => result.data),
+   {
+     staleTime: 2000,
+     cacheTime: 2000,
+   },
+ );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

If you make the change shown above, you won't see any difference. To trigger another query, you need to make a tweak into a query. Let's make the following change:

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    'beers',
    () => getBeerList(1000)().then((result) => result.data),
    {
      staleTime: 2000,
      cacheTime: 2000,
    },
  );

 useQuery(
-  'beers',
+  'beers-new',
   () => getBeerList(1000)().then((result) => result.data),
   {
     staleTime: 2000,
     cacheTime: 2000,
   },
 );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

This will trigger another request and associate it with the new key. You can make these queries more robust after putting them into square brackets. This creates a composed key and it's especially useful, when you are working with variables.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    'beers',
    () => getBeerList(1000)().then((result) => result.data),
    {
      staleTime: 2000,
      cacheTime: 2000,
    },
  );

 useQuery(
  'beers-new',
   () => getBeerList(1000)().then((result) => result.data),
   {
     staleTime: 2000,
     cacheTime: 2000,
   },
 );

+ useQuery(
+   ['beers', beerType, null],
+   () => getBeerList(1000)().then((result) => result.data),
+   {
+     staleTime: 2000,
+     cacheTime: 2000,
+   },
+ );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

Instead of `null`, you can easily put `undefined` as well. Both are converted into `null`, so you will get some consistency.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    'beers',
    () => getBeerList(1000)().then((result) => result.data),
    {
      staleTime: 2000,
      cacheTime: 2000,
    },
  );

 useQuery(
  'beers-new',
   () => getBeerList(1000)().then((result) => result.data),
   {
     staleTime: 2000,
     cacheTime: 2000,
   },
 );

 useQuery(
   ['beers', beerType, null],
   () => getBeerList(1000)().then((result) => result.data),
   {
     staleTime: 2000,
     cacheTime: 2000,
   },
  );

+ useQuery(
+   ['beers', beerType, undefined],
+   () => getBeerList(1000)().then((result) => result.data),
+   {
+     staleTime: 2000,
+     cacheTime: 2000,
+   },
+ );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

However, for our next example we can remove it. And `keep just intial query in place with a beerType variable`. This will be useful later during query invalidation.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
-   'beers',
+   ['beers', beerType],
-   () => getBeerList(1000)().then((result) => result.data),
+   () => getBeerList(1000)().then((result) => result.data)
-   {
-     staleTime: 2000,
-     cacheTime: 2000,
-   },
  );

- useQuery(
-  'beers-new',
-   () => getBeerList(1000)().then((result) => result.data),
-   {
-     staleTime: 2000,
-     cacheTime: 2000,
-   },
- );

- useQuery(
-   ['beers', beerType, null],
-   () => getBeerList(1000)().then((result) => result.data),
-   {
-     staleTime: 2000,
-     cacheTime: 2000,
-   },
- );

- useQuery(
-   ['beers', beerType, undefined],
-   () => getBeerList(1000)().then((result) => result.data),
-   {
-     staleTime: 2000,
-     cacheTime: 2000,
-   },
- );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

Now, we can modify the `BeerDetail.jsx` and fetch data via React Query.

```diff
--- a/BeerDetail.jsx
+++ b/BeerDetail.jsx
// src/components/BeerDetail.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../api/brewdog-punk-api';
+import { useQuery } from 'react-query';

export const BeerDetail = () => {
  const { id } = useParams();
  const history = useHistory();

+ const beerListResult = useQuery(['beers', id], () =>
+   getBeerById(1000)(id).then((result) => result.data[0]),
+ );

- const [state, setState] = React.useReducer(
-   (state, action) => ({ ...state, ...action }),
-   {
-     beer: null,
-     error: null,
-     status: 'pending',
-   },
- );
-
- React.useEffect(() => {
-   setState({ status: 'pending' });
-
-   getBeerById(1000)(id)
-     .then((response) =>
-       setState({ beer: response.data[0], status: 'success' }),
-     )
-     .catch((error) => setState({ error: error.message, status: 'error' }));
- }, [id]);
-
- const { beer, error, status } = state;
+ const { data: beer, status, error } = beerListResult;

  if (status === 'pending') {
    return <Loading />;
  }

  if (status === 'error') {
    return <Error message={error.toString()}  />;
  }

  return (
    <section className="text-gray-700 body-font overflow-hidden bg-white">
      <div className="container px-5 py-24 mx-auto">
        <div className="lg:w-4/5 mx-auto flex flex-wrap">
          <img
            alt={beer?.name}
            className="w-40 h-80 object-cover object-center rounded border border-gray-200"
            src={beer?.image_url}
          />
          <div className="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
            <h2 className="text-sm title-font text-gray-500 tracking-widest">
              {beer?.tagline}
            </h2>
            <h1 className="text-gray-900 text-3xl title-font font-medium mb-1">
              {beer?.name}
            </h1>
            <p className="leading-relaxed">{beer?.description}</p>
            <div className="flex mt-4">
              <button
                className="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
                onClick={() => history.push('/')}
              >
                Back
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
};
```

The latest versions of the components look as the following:

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(['beers', beerType], () =>
    getBeerList(1000)().then((result) => result.data),
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return <Error message={beerListResult.error.toString()} />;
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

```javascript
// src/components/BeerDetail.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerDetail = () => {
  const { id } = useParams();
  const history = useHistory();
  const beerListResult = useQuery(['beers', id], () =>
    getBeerById(1000)(id).then((result) => result.data[0]),
  );

  const { data: beer, status, error } = beerListResult;

  if (status === 'loading') {
    return <Loading />;
  }

  if (status === 'error') {
    return <Error message={error.toString()} />;
  }

  return (
    <section className="text-gray-700 body-font overflow-hidden bg-white">
      <div className="container px-5 py-24 mx-auto">
        <div className="lg:w-4/5 mx-auto flex flex-wrap">
          <img
            alt={beer?.name}
            className="w-40 h-80 object-cover object-center rounded border border-gray-200"
            src={beer?.image_url}
          />
          <div className="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
            <h2 className="text-sm title-font text-gray-500 tracking-widest">
              {beer?.tagline}
            </h2>
            <h1 className="text-gray-900 text-3xl title-font font-medium mb-1">
              {beer?.name}
            </h1>
            <p className="leading-relaxed">{beer?.description}</p>
            <div className="flex mt-4">
              <button
                className="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
                onClick={() => history.push('/')}
              >
                Back
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
};
```

## Retries

By default, there is a retry mechanism. It tries to re-fetch three times until it throws an error. You can override this behaviour and either set a different value or disable this mechanism completely. Let's see that in action for `BeerDetail.jsx`.

```diff
--- a/BeerDetail.jsx
+++ b/BeerDetail.jsx
// src/components/BeerDetail.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerDetail = () => {
  const { id } = useParams();
  const history = useHistory();
  const beerListResult = useQuery(['beers', id], () =>
    getBeerById(1000)(id).then((result) => result.data[0]),
+   {
+     retry: 1,
+   }
  );

  const { data: beer, status, error } = beerListResult;

  if (status === 'loading') {
    return <Loading />;
  }

  if (status === 'error') {
    return <Error message={error.toString()}  />;
  }

  return (
    <section className="text-gray-700 body-font overflow-hidden bg-white">
      <div className="container px-5 py-24 mx-auto">
        <div className="lg:w-4/5 mx-auto flex flex-wrap">
          <img
            alt={beer?.name}
            className="w-40 h-80 object-cover object-center rounded border border-gray-200"
            src={beer?.image_url}
          />
          <div className="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
            <h2 className="text-sm title-font text-gray-500 tracking-widest">
              {beer?.tagline}
            </h2>
            <h1 className="text-gray-900 text-3xl title-font font-medium mb-1">
              {beer?.name}
            </h1>
            <p className="leading-relaxed">{beer?.description}</p>
            <div className="flex mt-4">
              <button
                className="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
                onClick={() => history.push('/')}
              >
                Back
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
};
```

Now, if you try to open a detail with an invalid id, you will trigger that mechanism (feel free to reject the promise in any other way).

> As mentioned earlier, you can also disable the retry by settings the value to `false`.

```diff
--- a/BeerDetail.jsx
+++ b/BeerDetail.jsx
// src/components/BeerDetail.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerDetail = () => {
  const { id } = useParams();
  const history = useHistory();
  const beerListResult = useQuery(['beers', id], () =>
    getBeerById(1000)(id).then((result) => result.data[0]),
    {
-     retry: 1,
+     retry: false,
    }
  );

  const { data: beer, status, error } = beerListResult;

  if (status === 'loading') {
    return <Loading />;
  }

  if (status === 'error') {
    return <Error message={error.toString()}  />;
  }

  return (
    <section className="text-gray-700 body-font overflow-hidden bg-white">
      <div className="container px-5 py-24 mx-auto">
        <div className="lg:w-4/5 mx-auto flex flex-wrap">
          <img
            alt={beer?.name}
            className="w-40 h-80 object-cover object-center rounded border border-gray-200"
            src={beer?.image_url}
          />
          <div className="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
            <h2 className="text-sm title-font text-gray-500 tracking-widest">
              {beer?.tagline}
            </h2>
            <h1 className="text-gray-900 text-3xl title-font font-medium mb-1">
              {beer?.name}
            </h1>
            <p className="leading-relaxed">{beer?.description}</p>
            <div className="flex mt-4">
              <button
                className="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
                onClick={() => history.push('/')}
              >
                Back
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
};
```

However, let's return some standard value there.

```diff
--- a/BeerDetail.jsx
+++ b/BeerDetail.jsx
// src/components/BeerDetail.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerDetail = () => {
  const { id } = useParams();
  const history = useHistory();
  const beerListResult = useQuery(['beers', id], () =>
    getBeerById(1000)(id).then((result) => result.data[0]),
    {
-     retry: false,
+     retry: 3,
    }
  );

  const { data: beer, status, error } = beerListResult;

  if (status === 'loading') {
    return <Loading />;
  }

  if (status === 'error') {
    return <Error message={error.toString()}  />;
  }

  return (
    <section className="text-gray-700 body-font overflow-hidden bg-white">
      <div className="container px-5 py-24 mx-auto">
        <div className="lg:w-4/5 mx-auto flex flex-wrap">
          <img
            alt={beer?.name}
            className="w-40 h-80 object-cover object-center rounded border border-gray-200"
            src={beer?.image_url}
          />
          <div className="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
            <h2 className="text-sm title-font text-gray-500 tracking-widest">
              {beer?.tagline}
            </h2>
            <h1 className="text-gray-900 text-3xl title-font font-medium mb-1">
              {beer?.name}
            </h1>
            <p className="leading-relaxed">{beer?.description}</p>
            <div className="flex mt-4">
              <button
                className="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
                onClick={() => history.push('/')}
              >
                Back
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
};
```

The final snapshot of the component looks as the following:

```javascript
// src/components/BeerDetail.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerDetail = () => {
  const { id } = useParams();
  const history = useHistory();
  const beerListResult = useQuery(
    ['beers', id],
    () => getBeerById(1000)(id).then((result) => result.data[0]),
    {
      retry: 3,
    },
  );

  const { data: beer, status, error } = beerListResult;

  if (status === 'loading') {
    return <Loading />;
  }

  if (status === 'error') {
    return <Error message={error.toString()} />;
  }

  return (
    <section className="text-gray-700 body-font overflow-hidden bg-white">
      <div className="container px-5 py-24 mx-auto">
        <div className="lg:w-4/5 mx-auto flex flex-wrap">
          <img
            alt={beer?.name}
            className="w-40 h-80 object-cover object-center rounded border border-gray-200"
            src={beer?.image_url}
          />
          <div className="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
            <h2 className="text-sm title-font text-gray-500 tracking-widest">
              {beer?.tagline}
            </h2>
            <h1 className="text-gray-900 text-3xl title-font font-medium mb-1">
              {beer?.name}
            </h1>
            <p className="leading-relaxed">{beer?.description}</p>
            <div className="flex mt-4">
              <button
                className="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
                onClick={() => history.push('/')}
              >
                Back
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
};
```

## Parallel + Dependent Queries

When queries are triggered, they all start fetching data in parallel as shown in the example below.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(['beers', beerType], () =>
-   getBeerList(1000)().then((result) => result.data),
+   getBeerList(5000)().then((result) => result.data),
  );

+ useQuery(['beers', 'pilsner'], () =>
+   getListOfBeersByName(1000)('pilsner').then((result) => result.data),
+   {
+     enabled: false,
+   },
+ );
+
+ useQuery(['beers', 'lager'], () =>
+   getListOfBeersByName(1000)('lager').then((result) => result.data),
+   {
+     enabled: false,
+   },
+ );
+
+ useQuery(['beers', 'ale'], () =>
+   getListOfBeersByName(1000)('ale').then((result) => result.data),
+   {
+     enabled: false,
+   },
+ );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

This is fine, however, sometimes it's handy to have a mechanism for managing the dependencies. Let's imagine you have an id returned as a result of one query and you need to pass it as an input for another.

For such a use-case, you might be interested in learning how to handle dependent queries. Let's see that mechanism in action.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(['beers', beerType], () =>
    getBeerList(5000)().then((result) => result.data),
  );

  useQuery(['beers', 'pilsner'], () =>
    getListOfBeersByName(1000)('pilsner').then((result) => result.data),
    {
-     enabled: false,
+     enabled: beerListResult.isSuccess && beerListResult.data[0].id === 1,
    },
  );

  useQuery(['beers', 'lager'], () =>
    getListOfBeersByName(1000)('lager').then((result) => result.data),
    {
-     enabled: false,
+     enabled: beerListResult.isSuccess && beerListResult.data[0].id === 1,
    },
  );

  useQuery(['beers', 'ale'], () =>
   getListOfBeersByName(1000)('ale').then((result) => result.data),
   {
-     enabled: false,
+     enabled: beerListResult.isSuccess && beerListResult.data[0].id === 1,
   },
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

Let's make one more refactoring, which simplify the logic a bit.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
+ const beerListResult = useQuery(
+   ['beers', beerType],
+   beerType === 'all'
+     ? () => getBeerList(1000)().then((result) => result.data)
+     : () =>
+         getListOfBeersByName(1000)(beerType).then((result) => result.data),
- const beerListResult = useQuery(['beers', beerType], () =>
-   getBeerList(5000)().then((result) => result.data),
- );
-
- useQuery(['beers', 'pilsner'], () =>
-   getListOfBeersByName(1000)('pilsner').then((result) => result.data),
-   {
-    enabled: beerListResult.isSuccess && beerListResult.data[0].id === 1,
-   },
- );
-
- useQuery(['beers', 'lager'], () =>
-   getListOfBeersByName(1000)('lager').then((result) => result.data),
-   {
-    enabled: beerListResult.isSuccess && beerListResult.data[0].id === 1,
-   },
- );
-
- useQuery(['beers', 'ale'], () =>
-  getListOfBeersByName(1000)('ale').then((result) => result.data),
-  {
-    enabled: beerListResult.isSuccess && beerListResult.data[0].id === 1,
-  },
- );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

The final version of the `BeerList.jsx` looks as the following:

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType],
    beerType === 'all'
      ? () => getBeerList(1000)().then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return <Error message={beerListResult.error.toString()} />;
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

## Initial vs Placeholder Data

If you would like to allow React Query behaves as if it already has a data, you can consider using `initial` or `placeholder` data option. The main between them is that `placeholder` data is not cached and it is better to use it when you work with mock for instance vs using an actual initial data with the other option. Let's see both option in action.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
-import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
+import {
+ getBeerList,
+ getListOfBeersByName,
+ punkBeers,
+} from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType],
    beerType === 'all'
      ? () => getBeerList(3000)().then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
+   {
+     initialData: punkBeers,
+     staleTime: 2000,
+   },
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

If you run the open the DevTools, you are going to see that data is used initially (no HTTP call is made) and stays active until re-sync is going to happen. **This is the main reason why mocks shouldn't be used in this case as it could be confusing for the users**.

If you would like to have just a simple placeholder in place, the other option **Placeholder data** is a better choice. This just displays some data until a fetch is completed.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import {
  getBeerList,
  getListOfBeersByName,
  punkBeers,
} from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType],
    beerType === 'all'
      ? () => getBeerList(3000)().then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
    {
-     initialData: punkBeers,
+     placeholderData: punkBeers,
      staleTime: 2000,
    },
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

It's very useful and the actual usage depends on your use-case. `Initial data` works well, when you have cached data which would like to use it for pre-population of other queries. We are going to have a look at `pre-fetching` in the next section which work with use-cases like this. Before we do that, let's remove our latest changes as the following:

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
-import {
-  getBeerList,
-  getListOfBeersByName,
-  punkBeers,
-} from '../api/brewdog-punk-api';
+import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType],
    beerType === 'all'
      ? () => getBeerList(3000)().then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
-   {
-     placeholderData: punkBeers,
-     staleTime: 2000,
-   },
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

So, the latest snapshot of the `BeerList` component will turn into this:

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType],
    beerType === 'all'
      ? () => getBeerList(3000)().then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return <Error message={beerListResult.error.toString()} />;
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

## Pre-fetching Data

Pre-fetching is very useful technique for improving the general user experience. Let's have a look at one useful use-case.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
-import { getBeerList, getListOfBeersByName } from '../api/brewdog-punk-api';
+import {
+ getBeerList,
+ getListOfBeersByName,
+ getBeerById,
+} from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';
+import { queryClient } from '../App';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType],
    beerType === 'all'
      ? () => getBeerList(3000)().then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link to={`/beers/${beer.id}`} className="underline"
+                  onMouseOver={async () => {
+                   await queryClient.prefetchQuery(
+                     ['beers', beer.id.toString()],
+                     () =>
+                       getBeerById()(beer.id).then((result) => result.data[0]),
+                    );
+                  }}
                  >
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

You can also add `staleTime` which make sure only data older than specified time are fetched.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import {
  getBeerList,
  getListOfBeersByName,
  getBeerById,
} from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';
import { queryClient } from '../App';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType],
    beerType === 'all'
      ? () => getBeerList(3000)().then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link
                  to={`/beers/${beer.id}`}
                  className="underline"
                  onMouseOver={async () => {
                    await queryClient.prefetchQuery(
                      ['beers', beer.id.toString()],
                      () =>
                        getBeerById()(beer.id).then((result) => result.data[0]),
+                     {
+                       staleTime: 15000,
+                     },
                    );
                  }}
                >
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

The latest snapshot of the `BeerList.jsx` looks as the following code:

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import {
  getBeerList,
  getListOfBeersByName,
  getBeerById,
} from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';
import { queryClient } from '../App';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType],
    beerType === 'all'
      ? () => getBeerList(3000)().then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return <Error message={beerListResult.error.toString()} />;
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link
                  to={`/beers/${beer.id}`}
                  className="underline"
                  onMouseOver={async () => {
                    await queryClient.prefetchQuery(
                      ['beers', beer.id.toString()],
                      () =>
                        getBeerById()(beer.id).then((result) => result.data[0]),
                      {
                        staleTime: 15000,
                      },
                    );
                  }}
                >
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

## Query Invalidation

Invalidating queries is another useful technique which helps you to fetch data before they get stale. Let's see an example:

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import {
  getBeerList,
  getListOfBeersByName,
  getBeerById,
} from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';
import { queryClient } from '../App';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType],
    beerType === 'all'
      ? () => getBeerList(3000)().then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
+   queryClient.invalidateQueries(['beers', type]);
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link
                  to={`/beers/${beer.id}`}
                  className="underline"
                  onMouseOver={async () => {
                    await queryClient.prefetchQuery(
                      ['beers', beer.id.toString()],
                      () =>
                        getBeerById()(beer.id).then((result) => result.data[0]),
                      {
                        staleTime: 15000,
                      },
                    );
                  }}
                >
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

This will invalidate just specific React Query. If you click on `Pilsner`, `Lager` or `Ale`, new data is going to be always re-fetched. But `All beers` keeps working as usual.

Here is the full snapshot of the component:

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import {
  getBeerList,
  getListOfBeersByName,
  getBeerById,
} from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';
import { queryClient } from '../App';

export const BeerList = () => {
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType],
    beerType === 'all'
      ? () => getBeerList(3000)().then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    queryClient.invalidateQueries(['beers', type]);
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return <Error message={beerListResult.error.toString()} />;
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={true}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link
                  to={`/beers/${beer.id}`}
                  className="underline"
                  onMouseOver={async () => {
                    await queryClient.prefetchQuery(
                      ['beers', beer.id.toString()],
                      () =>
                        getBeerById()(beer.id).then((result) => result.data[0]),
                      {
                        staleTime: 15000,
                      },
                    );
                  }}
                >
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

## Paginated Queries

Pagination is a common technique on a frontend application. It's relatively simple to do it with React Query.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import {
  getBeerList,
  getListOfBeersByName,
  getBeerById,
} from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';
import { queryClient } from '../App';

+const PAGE_SIZE = 50;

export const BeerList = () => {
+ const [page, setPage] = React.useState(1);
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType, page],
    beerType === 'all'
-     ? () => getBeerList(3000)().then((result) => result.data)
+     ? () => getBeerList(1000)(page, PAGE_SIZE).then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    queryClient.invalidateQueries(['beers', type]);
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
-             disabled={true}
+             disabled={page === 1}
+             onClick={() => setPage((old) => Math.max(old - 1, 0))}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
-             disabled={true}
+             disabled={beerListResult.data.length < PAGE_SIZE}
+             onClick={() => setPage((old) => old + 1)}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link
                  to={`/beers/${beer.id}`}
                  className="underline"
                  onMouseOver={async () => {
                    await queryClient.prefetchQuery(
                      ['beers', beer.id.toString()],
                      () =>
                        getBeerById()(beer.id).then((result) => result.data[0]),
                      {
                        staleTime: 15000,
                      },
                    );
                  }}
                >
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

This is working relatively fine, but you can see a small problem. Every time we click on a next button, a loading indicator is triggered as we are fetching new data. If we would like to tweak this behaviour, we can use specific attribute called `keepPreviousData`, which literally does what is says. It keeps the old data until the new ones are fetched.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import {
  getBeerList,
  getListOfBeersByName,
  getBeerById,
} from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';
import { queryClient } from '../App';

const PAGE_SIZE = 50;

export const BeerList = () => {
  const [page, setPage] = React.useState(1);
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType, page],
    beerType === 'all'
      ? () => getBeerList(1000)(page, PAGE_SIZE).then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
    { keepPreviousData: true },
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    queryClient.invalidateQueries(['beers', type]);
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={page === 1}
              onClick={() => setPage((old) => Math.max(old - 1, 0))}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={beerListResult.data.length < PAGE_SIZE}
              onClick={() => setPage((old) => old + 1)}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link
                  to={`/beers/${beer.id}`}
                  className="underline"
                  onMouseOver={async () => {
                    await queryClient.prefetchQuery(
                      ['beers', beer.id.toString()],
                      () =>
                        getBeerById()(beer.id).then((result) => result.data[0]),
                      {
                        staleTime: 15000,
                      },
                    );
                  }}
                >
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

There is a little problem. You can click multiple times on the next button, which triggers multiple queries. There is a special field called `isPreviousData` in the React Query result. It helps you with controlling the logic a bit more.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import {
  getBeerList,
  getListOfBeersByName,
  getBeerById,
} from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';
import { queryClient } from '../App';

const PAGE_SIZE = 50;

export const BeerList = () => {
  const [page, setPage] = React.useState(1);
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType, page],
    beerType === 'all'
      ? () => getBeerList(1000)(page, PAGE_SIZE).then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
    { keepPreviousData: true },
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    queryClient.invalidateQueries(['beers', type]);
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return (
      <Error message={beerListResult.error.toString()}  />
    );
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={page === 1}
              onClick={() => setPage((old) => Math.max(old - 1, 0))}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={
+               beerListResult.isPreviousData ||
                beerListResult.data.length < PAGE_SIZE
              }
              onClick={() => setPage((old) => old + 1)}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link
                  to={`/beers/${beer.id}`}
                  className="underline"
                  onMouseOver={async () => {
                    await queryClient.prefetchQuery(
                      ['beers', beer.id.toString()],
                      () =>
                        getBeerById()(beer.id).then((result) => result.data[0]),
                      {
                        staleTime: 15000,
                      },
                    );
                  }}
                >
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

The latest snapshot of the `BeerList.jsx` component looks like as the following now:

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Loading } from './Loading';
import { Error } from './Error';
import { Link } from 'react-router-dom';
import {
  getBeerList,
  getListOfBeersByName,
  getBeerById,
} from '../api/brewdog-punk-api';
import { useQuery } from 'react-query';
import { queryClient } from '../App';

const PAGE_SIZE = 50;

export const BeerList = () => {
  const [page, setPage] = React.useState(1);
  const [beerType, setBeerType] = React.useState('all');
  const beerListResult = useQuery(
    ['beers', beerType, page],
    beerType === 'all'
      ? () => getBeerList(1000)(page, PAGE_SIZE).then((result) => result.data)
      : () =>
          getListOfBeersByName(1000)(beerType).then((result) => result.data),
    { keepPreviousData: true },
  );

  const getAllBeers = async () => {
    setBeerType('all');
  };

  const getBeerListByType = async (type) => {
    queryClient.invalidateQueries(['beers', type]);
    setBeerType(type);
  };

  if (beerListResult.isLoading) {
    return <Loading />;
  }

  if (beerListResult.isError) {
    return <Error message={beerListResult.error.toString()} />;
  }

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-24 lg:pb-64">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing React Query
            capabalities of an unofficial Brewdog's API.
          </p>
          <div className="flex justify-center">
            <button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              All Beers
            </button>
            <button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Pilsner
            </button>
            <button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Lager
            </button>
            <button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
            >
              Ale
            </button>
          </div>

          <div className="flex justify-center mt-5">
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={page === 1}
              onClick={() => setPage((old) => Math.max(old - 1, 0))}
            >
              Prev
            </button>
            <button
              className="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 mr-2 rounded-l disabled:font-bold disabled:py-2 disabled:px-4 disabled:rounded disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={
                beerListResult.isPreviousData ||
                beerListResult.data.length < PAGE_SIZE
              }
              onClick={() => setPage((old) => old + 1)}
            >
              Next
            </button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beerListResult.data.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <Link
                  to={`/beers/${beer.id}`}
                  className="underline"
                  onMouseOver={async () => {
                    await queryClient.prefetchQuery(
                      ['beers', beer.id.toString()],
                      () =>
                        getBeerById()(beer.id).then((result) => result.data[0]),
                      {
                        staleTime: 15000,
                      },
                    );
                  }}
                >
                  Detail
                </Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

## Mutations

Mutations are useful when you would like to make some modifications on the server (e.g. submit data) and have those changes reflected on the frontend. Let's see that in action.

The first step, however, is to add `useQuery` for data fetching. You can absolutely use `mutations separately`, but there are some nice benefits if used together.

```diff
--- a/NoteList.jsx
+++ b/NoteList.jsx
// src/components/NoteList.jsx
import React from 'react';
import { getNotes, addNote } from '../api/notes';
import { Loading } from './Loading';
import { Error } from './Error';
import { NoteForm } from './NoteForm';
+import { useQuery } from 'react-query';

export const NoteList = () => {
-  const [state, setState] = React.useReducer(
-    (state, action) => ({ ...state, ...action }),
-    {
-      notes: null,
-      error: null,
-      status: 'pending',
-    },
-  );

-  const getAllNotes = async () => {
-   setState({ status: 'pending' });
-   return getNotes(1000)()
-     .then((response) => setState({ notes: response.data, status: 'success' }))
-     .catch((error) => setState({ error: error.message, status: 'error' }));
-   };

+  const notesListResult = useQuery(['notes'], () =>
+    getNotes(2000)().then((result) => result.data),
+  );

  const addNewNote = async (note) => {
    setState({ status: 'pending' });
    return addNote(1000)(note)
      .then((response) =>
        setState((current) => ({
          notes: [...current, note],
          status: 'success',
        })),
      )
      .catch((error) => setState({ error: error.message, status: 'error' }));
  };

  const handleNoteAdd = async (note) => {
    await addNewNote(note);
    setState({ notes: [...state.notes, note], status: 'success' });
  };

- React.useEffect(() => {
-   getAllNotes();
- }, []);

- const { status, error, notes } = state;

- if (status === 'pending') {
+ if (notesListResult.isLoading) {
    return <Loading />;
  }

- if (notesListResult.error) {
+ if (notesListResult.isError) {
-   return <Error message={error} />
+   return <Error message={notesListResult.error.toString()} />;
  }

  return (
    <div className="w-11/12 md:w-4/5 lg:w-1/2 mx-auto">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full px-4 mt-4">
          <h2 className="text-4xl font-semibold text-black">
            Mutation test with Notes API
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's test basics of mutation with a simple API written in Deno.
          </p>
        </div>
      </div>

      <NoteForm onNoteAdd={handleNoteAdd} />

      <div className="bg-white shadow-md rounded my-6">
        <table className="text-left w-full border-collapse">
          <thead>
            <tr>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Id
              </th>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Title
              </th>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Color
              </th>
            </tr>
          </thead>
          <tbody>
-           {notes.map((note) => (
+           {notesListResult.data.map((note) => (
              <tr className="hover:bg-grey-lighter" key={note.id}>
                <td className="py-4 px-6 border-b border-grey-light">
                  {note.id}
                </td>
                <td className="py-4 px-6 border-b border-grey-light">
                  {note.title}
                </td>
                <td className="py-4 px-6 border-b border-grey-light">
                  <span
                    className={`rounded py-1 px-3 text-xs font-bold bg-${note.color}-400`}
                  >
                    {note.color}
                  </span>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};
```

The list of the notes is now handled via React Query. Let's add some mutations.

```diff
--- a/NoteList.jsx
+++ b/NoteList.jsx
// src/components/NoteList.jsx
import React from 'react';
import { getNotes, addNote } from '../api/notes';
import { Loading } from './Loading';
import { Error } from './Error';
import { NoteForm } from './NoteForm';
-import { useQuery } from 'react-query';
+import { useQuery, useMutation } from 'react-query';

export const NoteList = () => {
  const notesListResult = useQuery(['notes'], () =>
    getNotes(2000)().then((result) => result.data),
  );

+ const mutation = useMutation((note) => addNote(1000)(note));

  const addNewNote = async (note) => {
-   setState({ status: 'pending' });
-   return addNote(1000)(note)
-     .then((response) =>
-       setState((current) => ({
-         notes: [...current, note],
-         status: 'success',
-       })),
-     )
-     .catch((error) => setState({ error: error.message, status: 'error' }));
+      mutation.mutate(note);
  };

  const handleNoteAdd = async (note) => {
    await addNewNote(note);
-   setState({ notes: [...state.notes, note], status: 'success' });
  };

- if (notesListResult.isLoading) {
+ if (notesListResult.isLoading || mutation.isLoading) {
    return <Loading />;
  }

  if (notesListResult.isError) {
    return <Error message={notesListResult.error.toString()} />;
  }

+ if (mutation.isError) {
+   return <Error message={mutation.error.toString()} />;
+ }

  return (
    <div className="w-11/12 md:w-4/5 lg:w-1/2 mx-auto">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full px-4 mt-4">
          <h2 className="text-4xl font-semibold text-black">
            Mutation test with Notes API
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's test basics of mutation with a simple API written in Deno.
          </p>
        </div>
      </div>

      <NoteForm onNoteAdd={handleNoteAdd} />

      <div className="bg-white shadow-md rounded my-6">
        <table className="text-left w-full border-collapse">
          <thead>
            <tr>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Id
              </th>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Title
              </th>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Color
              </th>
            </tr>
          </thead>
          <tbody>
            {notesListResult.data.map((note) => (
              <tr className="hover:bg-grey-lighter" key={note.id}>
                <td className="py-4 px-6 border-b border-grey-light">
                  {note.id}
                </td>
                <td className="py-4 px-6 border-b border-grey-light">
                  {note.title}
                </td>
                <td className="py-4 px-6 border-b border-grey-light">
                  <span
                    className={`rounded py-1 px-3 text-xs font-bold bg-${note.color}-400`}
                  >
                    {note.color}
                  </span>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};
```

Right now, it works. But the user experience is not the best. We need to wait until data is re-fetched to be shown. We can use mutation side-effects to make some improvements.

```diff
--- a/NoteList.jsx
+++ b/NoteList.jsx
// src/components/NoteList.jsx
import React from 'react';
import { getNotes, addNote } from '../api/notes';
import { Loading } from './Loading';
import { Error } from './Error';
import { NoteForm } from './NoteForm';
import { useQuery, useMutation } from 'react-query';
import { queryClient } from '../App';

export const NoteList = () => {
  const notesListResult = useQuery(['notes'], () =>
    getNotes(2000)().then((result) => result.data),
  );

- const mutation = useMutation((note) => addNote(1000)(note));
+ const mutation = useMutation((note) => addNote(1000)(note), {
+   onMutate: (variables) => {
+     queryClient.cancelQueries('notes');
+
+      const oldNotes = queryClient.getQueryData('notes');
+      queryClient.setQueryData('notes', (oldNotes) => {
+        return [
+          ...oldNotes,
+          {
+            ...variables,
+            id: Math.random().toString(36).substring(7),
+          },
+        ];
+      });
+
+      return oldNotes;
+    },
+    onError: (error, variables, context) => {
+      queryClient.setQueryData('notes', context);
+    },
+    onSuccess: (data, variables, context) => {
+      queryClient.invalidateQueries('notes');
+    },
+    onSettled: (data, error, variables, context) => {},
+  });

  const addNewNote = async (note) => {
    mutation.mutate(note);
  };

  const handleNoteAdd = async (note) => {
    await addNewNote(note);
  };

  if (notesListResult.isLoading || mutation.isLoading) {
    return <Loading />;
  }

  if (notesListResult.isError) {
    return <Error message={notesListResult.error.toString()} />;
  }

  if (mutation.isError) {
-   return <Error message={mutation.error.toString()} />;
+   // return <Error message={mutation.error.toString()} />;
  }

  return (
    <div className="w-11/12 md:w-4/5 lg:w-1/2 mx-auto">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full px-4 mt-4">
          <h2 className="text-4xl font-semibold text-black">
            Mutation test with Notes API
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's test basics of mutation with a simple API written in Deno.
          </p>
        </div>
      </div>

      <NoteForm onNoteAdd={handleNoteAdd} />

      <div className="bg-white shadow-md rounded my-6">
        <table className="text-left w-full border-collapse">
          <thead>
            <tr>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Id
              </th>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Title
              </th>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Color
              </th>
            </tr>
          </thead>
          <tbody>
            {notesListResult.data.map((note) => (
              <tr className="hover:bg-grey-lighter" key={note.id}>
                <td className="py-4 px-6 border-b border-grey-light">
                  {note.id}
                </td>
                <td className="py-4 px-6 border-b border-grey-light">
                  {note.title}
                </td>
                <td className="py-4 px-6 border-b border-grey-light">
                  <span
                    className={`rounded py-1 px-3 text-xs font-bold bg-${note.color}-400`}
                  >
                    {note.color}
                  </span>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};
```

The latest state of the `NoteList.jsx` looks as the following:

```javascript
// src/components/NoteList.jsx
import React from 'react';
import { getNotes, addNote } from '../api/notes';
import { Loading } from './Loading';
import { Error } from './Error';
import { NoteForm } from './NoteForm';
import { useQuery, useMutation } from 'react-query';
import { queryClient } from '../App';

export const NoteList = () => {
  const notesListResult = useQuery(['notes'], () =>
    getNotes(2000)().then((result) => result.data),
  );

  const mutation = useMutation((note) => addNote(1000)(note), {
    onMutate: (variables) => {
      queryClient.cancelQueries('notes');

      const oldNotes = queryClient.getQueryData('notes');
      queryClient.setQueryData('notes', (oldNotes) => {
        return [
          ...oldNotes,
          {
            ...variables,
            id: Math.random().toString(36).substring(7),
          },
        ];
      });

      return oldNotes;
    },
    onError: (error, variables, context) => {
      queryClient.setQueryData('notes', context);
    },
    onSuccess: (data, variables, context) => {
      queryClient.invalidateQueries('notes');
    },
    onSettled: (data, error, variables, context) => {},
  });

  const addNewNote = async (note) => {
    mutation.mutate(note);
  };

  const handleNoteAdd = async (note) => {
    await addNewNote(note);
  };

  if (notesListResult.isLoading || mutation.isLoading) {
    return <Loading />;
  }

  if (notesListResult.isError) {
    return <Error message={notesListResult.error.toString()} />;
  }

  if (mutation.isError) {
    // return <Error message={mutation.error.toString()} />;
  }

  return (
    <div className="w-11/12 md:w-4/5 lg:w-1/2 mx-auto">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full px-4 mt-4">
          <h2 className="text-4xl font-semibold text-black">
            Mutation test with Notes API
          </h2>
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's test basics of mutation with a simple API written in Deno.
          </p>
        </div>
      </div>

      <NoteForm onNoteAdd={handleNoteAdd} />

      <div className="bg-white shadow-md rounded my-6">
        <table className="text-left w-full border-collapse">
          <thead>
            <tr>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Id
              </th>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Title
              </th>
              <th className="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
                Color
              </th>
            </tr>
          </thead>
          <tbody>
            {notesListResult.data.map((note) => (
              <tr className="hover:bg-grey-lighter" key={note.id}>
                <td className="py-4 px-6 border-b border-grey-light">
                  {note.id}
                </td>
                <td className="py-4 px-6 border-b border-grey-light">
                  {note.title}
                </td>
                <td className="py-4 px-6 border-b border-grey-light">
                  <span
                    className={`rounded py-1 px-3 text-xs font-bold bg-${note.color}-400`}
                  >
                    {note.color}
                  </span>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};
```

## Other interesting topics

There is a lot more [React Query](https://react-query.tanstack.com/ can offer.

I personally recommend to have a look at the following two topics:

- Infinite Queries - [https://react-query.tanstack.com/guides/infinite-queries](https://react-query.tanstack.com/guides/infinite-queries)
- Query Cancellation - [https://react-query.tanstack.com/guides/query-cancellation]

## Conclusion

Thanks for following this workshop material. Any feedback appreciated (radek.tomasek@gmail.com).