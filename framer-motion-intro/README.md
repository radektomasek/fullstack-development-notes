# Introduction to Framer Motion

In this workshop, we are going to show basic capabilities of [Framer Motion](https://www.framer.com/motion), an amazing library that helps you to create beautiful motion animations in React.

With the concepts covered in this material, you should be comfortable creating your own animation very easily. However, there is always a [great documentation](https://www.framer.com/docs) and [other resources](https://www.framer.com/examples) that help you to go beyond.

## Pre-requisites

You should be pretty comfortable with React and its hooks API. A basic understanding of CSS might come handy as well.

## Project Initialization

We are going to initialize the project with **Snowpack** and its React Template.

To simplify things in this workshop, a **plain JavaScript is going to be used** (instead of **TypeScript**).

**Tailwind** (with **PostCSS**) is going to be used to make our project look nice.

To bootstrap the project, just type (in terminal):

```bash
npx create-snowpack-app framer-motion-intro --template @snowpack/app-template-react
```

### Installing dependencies

Another important step is to install **Framer Motion**. **Routing** might come handy for our examples too, therefore let's install **React Router** as well.

In the terminal (in the project folder), just type:

```bash
# install Framer Motion, and React Router for web.
npm install --save framer-motion react-router-dom
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

## Basic components and helpers

In this section, we are going to add some basic implementation for our application.

> Note: The application is going to be fully static (no HTTP API requests is going to be made).

This will be used as initial work for our further examples in order to save time (in next sections, we are going to add some actual functionality related to Framer Motion).

### Helper functions

In this part, we will add some placeholder data.

#### utils/mocks.js

```javascript
// src/utils/mocks.js
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
    image_url: 'https://images.punkapi.com/v2/keg.png',
  },
  {
    id: 277,
    name: 'Prototype Blonde Ale',
    tagline: 'Mellow Golden Ale.',
    description:
      'This Blonde Ale is a new recipe and uses a new yeast - so it is a true Prototype. Originally lined up as part of the 2017 Prototype Challenge it was released shortly afterwards. but hey, these things happen.',
    image_url: 'https://images.punkapi.com/v2/keg.png',
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

export const mocks = [...punkBeers, ...blondeBeers, ...pilsnerBeers];

export const getBeerList = () => mocks;

export const getBeerById = (beerId) =>
  mocks.find((beer) => beer.id === parseInt(beerId));

export const getListOfBeersByName = (phrase = '') =>
  mocks.filter((beer) =>
    beer.description.toLowerCase().includes(phrase.toLowerCase()),
  );
```

### Components

To be able to demonstrate the concepts, we need to add some basic components. At this point, no animation is going to be included.

#### components/BeerLogo.jsx

```javascript
// src/components/BeerLogo.jsx
/* 
  The SVG used in this component was downloaded from https://www.svgrepo.com
  For demo purposes during Framer Motion Demo only.
*/

import React from 'react';

export const BeerLogo = () => {
  return (
    <header className="flex flex-wrap justify-center w-full">
      <div className="w-20 mt-10 mb-5">
        <svg
          version="1.1"
          id="Beer"
          xmlns="http://www.w3.org/2000/svg"
          xmlnsXlink="http://www.w3.org/1999/xlink"
          x="0px"
          y="0px"
          viewBox="0 0 485 485"
          xmlSpace="preserve"
        >
          <g>
            <path
              d="M379.793,141.535l2.621-80.154h-16.163C359.26,26.423,328.34,0,291.357,0h-97.715c-36.983,0-67.903,26.423-74.894,61.381
		h-16.163l2.62,80.156c-9.567,12.064-14.77,26.915-14.77,42.474c0,16.985,6.367,33.374,17.657,45.898L116.431,485h252.132
		l8.341-255.088c11.292-12.524,17.66-28.914,17.66-45.901C394.563,168.452,389.361,153.601,379.793,141.535z M193.643,30h97.715
		c20.325,0,37.627,13.148,43.878,31.381h-185.47C156.016,43.148,173.317,30,193.643,30z M351.996,212.465l-4.677,4.266L339.527,455
		H145.466l-7.787-238.271l-4.676-4.266c-7.986-7.285-12.566-17.655-12.566-28.453c0-9.99,3.822-19.465,10.763-26.678l4.39-4.563
		l-2.006-61.39h217.834l-2.007,61.388l4.39,4.563c6.941,7.214,10.764,16.688,10.764,26.68
		C364.563,194.81,359.982,205.181,351.996,212.465z"
            />
          </g>
        </svg>
      </div>
    </header>
  );
};
```

#### components/BeerList.jsx

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <h2 className="text-4xl font-semibold text-black">
            Brewdog's Beer Selection
          </h2>
          <BeerLogo />
          <p className="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
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

#### components/BeerDetail.jsx

```javascript
// src/components/BeerDetail.jsx
import React from 'react';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../utils/mocks';

export const BeerDetail = ({ handleModalOpen }) => {
  const { id } = useParams();
  const history = useHistory();

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
    },
  );

  React.useEffect(() => {
    try {
      const beer = getBeerById(id);
      if (!beer) {
        throw new ReferenceError(`Beer with id: ${id} doesn't exist!`);
      }
      setState({ beer });
    } catch (error) {
      history.push('/');
    }
  }, [id]);

  const { beer } = state;

  return (
    <>
      <section className="text-gray-700 body-font overflow-hidden bg-white z-0">
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
              <h1 className="text-gray-900 text-3xl title-font font-medium mb-1 mt-2">
                {beer?.name}
              </h1>
              <p className="leading-relaxed mt-3">{beer?.description}</p>
              <div className="flex mt-4">
                <button
                  className="flex text-white bg-green-500 border-0 mr-5 py-2 px-6 focus:outline-none hover:bg-green-500 rounded"
                  onClick={handleModalOpen}
                >
                  Show a modal
                </button>
                <button
                  className="flex text-white bg-red-400 border-0 py-2 px-6 focus:outline-none hover:bg-red-400 rounded"
                  onClick={() => history.push('/')}
                >
                  Back
                </button>
              </div>
            </div>
          </div>
        </div>
      </section>
    </>
  );
};
```

#### components/BeerDetailModal.jsx

```javascript
// src/components/BeerDetailModal.jsx
import React from 'react';
import ReactDOM from 'react-dom';

export const BeerDetailModal = ({ showModal, handleModalClose }) =>
  showModal
    ? ReactDOM.createPortal(
        <div
          className="main-modal fixed w-full inset-0 z-50 overflow-hidden flex justify-center items-center animated fadeIn faster"
          style={{ background: 'rgba(0,0,0,.7)' }}
        >
          <div className="border border-blue-500 shadow-lg modal-container bg-white w-4/12 md:max-w-11/12 mx-auto rounded-xl shadow-lg z-50 overflow-y-auto">
            <div className="modal-content py-4 text-left px-6">
              <div className="flex justify-between items-center pb-3">
                <p className="text-2xl font-bold text-gray-500">
                  A simple modal
                </p>

                <div
                  className="modal-close cursor-pointer z-50"
                  onClick={handleModalClose}
                >
                  <svg
                    className="fill-current text-gray-500"
                    xmlns="http://www.w3.org/2000/svg"
                    width="18"
                    height="18"
                    viewBox="0 0 18 18"
                  >
                    <path d="M14.53 4.53l-1.06-1.06L9 7.94 4.53 3.47 3.47 4.53 7.94 9l-4.47 4.47 1.06 1.06L9 10.06l4.47 4.47 1.06-1.06L10.06 9z"></path>
                  </svg>
                </div>
              </div>

              <p className="text-gray-500">
                Let's see some advanced Framer Motion Capabalities in action!
              </p>

              <div className="flex justify-end pt-2 space-x-14">
                <button
                  className="px-4 bg-blue-500 p-3 ml-3 rounded-lg text-white hover:bg-teal-400"
                  onClick={handleModalClose}
                >
                  Close
                </button>
              </div>
            </div>
          </div>
        </div>,
        document.body,
      )
    : null;
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
+import { BeerDetailModal } from './components/BeerDetailModal';
+import { BeerDetail } from './components/BeerDetail';
+import { BeerList } from './components/BeerList';

function App() {
  return (
-   <div className="App">
-     <h1 className="p-4 m-4 bg-blue-300">Hello World</h1>
-   </div>
+   <>
+     <BeerDetailModal showModal={showModal} handleModalClose={closeModal} />
+     <Router>
+       <Route
+          render={({ location }) => (
+          <Switch>
+             <Route exact path="/beers">
+               <BeerList />
+             </Route>
+             <Route path="/beers/:id">
+               <BeerDetail handleModalOpen={openModal} />
+             </Route>
+             <Redirect from="/" to="/beers" />
+          </Switch>
+       )}
+      />
+     </Router>
+   </>
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
import { BeerDetailModal } from './components/BeerDetailModal';
import { BeerDetail } from './components/BeerDetail';
import { BeerList } from './components/BeerList';

function App() {
  const [showModal, setShowModal] = React.useState(false);

  const openModal = () => {
    setShowModal(true);
  };

  const closeModal = () => {
    setShowModal(false);
  };

  return (
    <>
      <BeerDetailModal showModal={showModal} handleModalClose={closeModal} />

      <Router>
        <Route
          render={({ location }) => (
            <Switch>
              <Route exact path="/beers">
                <BeerList />
              </Route>
              <Route path="/beers/:id">
                <BeerDetail handleModalOpen={openModal} />
              </Route>
              <Redirect from="/" to="/beers" />
            </Switch>
          )}
        />
      </Router>
    </>
  );
}

export default App;
```

## Basic Concepts

The first motion example we are going to cover is about importing the library into project, enabling animations for any HTML element with animate prop and tweaking CSS properties/convenient ones.

> Note: You can apply motion only on HTML elements directly.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
+import { motion } from 'framer-motion';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
-         <h2
+         <motion.h2
           className="text-4xl font-semibold text-black"
+          animate={{
+             color: 'lightcoral',
+             textDecoration: 'underline',
+             rotate: 5,
+             x: '5%',
+           }}
          >
            Brewdog's Beer Selection
+         </motion.h2>
-         </h2>
          <BeerLogo />
-         <p
+         <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
+           animate={{ x: 20, y: -10, scale: 1.2 }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
+         </motion.p>
-         </p>
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

The latest snapshot of the `BeerList.jsx` looks like as the following:

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            animate={{
              color: 'lightcoral',
              textDecoration: 'underline',
              rotate: 5,
              x: '5%',
            }}
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            animate={{ x: 20, y: -10, scale: 1.2 }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
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

## More props (initial + transition) and a few useful options

In this section, we are going to cover initial and transition props, various motion types (spring with stiffness, tween with duration) with some delays.

### Initial option

This option helps to setup the initial state of an element from which it animates.

```diff
+++ a/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
+           initial={{ x: '-100vw', color: 'lightblue', opacity: 0.4 }}
            animate={{
              color: 'lightcoral',
              textDecoration: 'underline',
              rotate: 5,
              x: '5%',
+             opacity: 1,
            }}
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
+           initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
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

### Transition

This option helps to control, what happens between the initial and final state of an animation. Things like easing an animation, or setting up a duration. Let's see that in action.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
-           initial={{ x: '-100vw', color: 'lightblue', opacity: 0.4 }}
+           initial={{ x: 0, color: 'lightblue', opacity: 0.4, rotate: 0 }}
            animate={{
              color: 'lightcoral',
              textDecoration: 'underline',
              rotate: 5,
              x: '5%',
              opacity: 1,
            }}
+           transition={{ delay: 2, duration: 5, type: 'tween' }}
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
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

> Note: Type **tween** is a default one, you can omit it and get the same result.

If you would like to experience a little bouncing back feedback during an animation, you can enable `type: spring`.

This option **duration** doesn't work with animation of type `spring`. However, you can enable **stiffness** which accepts an integer number. The higher the number is, the faster the animation you can experience. Let's see all of that in action.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            initial={{
              x: 0,
              color: 'lightblue',
              opacity: 0.4,
              rotate: 0,
            }}
            animate={{
              color: 'lightcoral',
              textDecoration: 'underline',
              rotate: 5,
              x: '5%',
              opacity: 1,
            }}
            transition={{
              delay: 2,
-             duration: 5,
-             type: 'tween',
+             type: 'spring',
+             stiffness: 1000,
            }}
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
+           transition={{
+             type: 'spring',
+             stiffness: 200,
+           }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
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

The latest snapshot of the `BeerList.jsx` looks like as the following:

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            initial={{
              x: 0,
              color: 'lightblue',
              opacity: 0.4,
              rotate: 0,
            }}
            animate={{
              color: 'lightcoral',
              textDecoration: 'underline',
              rotate: 5,
              x: '5%',
              opacity: 1,
            }}
            transition={{
              delay: 2,
              type: 'spring',
              stiffness: 1000,
            }}
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
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

## Drag & Drop

Enabling Drag & Drop capabilities is very easy and there are also a few useful options. Let's see that in action whist updating `BeerLogo.jsx` component.

```diff
--- a/BeerLogo.jsx
+++ b/BeerLogo.jsx
// src/components/BeerLogo.jsx
/*
  The SVG used in this component was downloaded from https://www.svgrepo.com
  For demo purposes during Framer Motion Demo only.
*/

import React from 'react';
+import { motion } from 'framer-motion';

export const BeerLogo = () => {
  return (
-   <header
+   <motion.header
      className="flex flex-wrap justify-center w-full"
+     drag
    >
      <div className="w-20 mt-10 mb-5">
        <svg
          version="1.1"
          id="Beer"
          xmlns="http://www.w3.org/2000/svg"
          xmlnsXlink="http://www.w3.org/1999/xlink"
          x="0px"
          y="0px"
          viewBox="0 0 485 485"
          xmlSpace="preserve"
        >
          <g>
            <path
              d="M379.793,141.535l2.621-80.154h-16.163C359.26,26.423,328.34,0,291.357,0h-97.715c-36.983,0-67.903,26.423-74.894,61.381
		h-16.163l2.62,80.156c-9.567,12.064-14.77,26.915-14.77,42.474c0,16.985,6.367,33.374,17.657,45.898L116.431,485h252.132
		l8.341-255.088c11.292-12.524,17.66-28.914,17.66-45.901C394.563,168.452,389.361,153.601,379.793,141.535z M193.643,30h97.715
		c20.325,0,37.627,13.148,43.878,31.381h-185.47C156.016,43.148,173.317,30,193.643,30z M351.996,212.465l-4.677,4.266L339.527,455
		H145.466l-7.787-238.271l-4.676-4.266c-7.986-7.285-12.566-17.655-12.566-28.453c0-9.99,3.822-19.465,10.763-26.678l4.39-4.563
		l-2.006-61.39h217.834l-2.007,61.388l4.39,4.563c6.941,7.214,10.764,16.688,10.764,26.68
		C364.563,194.81,359.982,205.181,351.996,212.465z"
            />
          </g>
        </svg>
      </div>
+   </motion.header>
-   </header>
  );
};
```

By just adding `drag` prop, the dragging capabilities start working automatically and **you can drag the icon freely around the page**.

If you would like to put a few restrictions, there are a few useful options ready for you. `dragConstraints` and `dragElastic`. The first one (**dragConstraints**) allows you to set boundaries, by how much you can move with the object in each direction. If you specify `0`, the object will always return to start position in that direction.

The latter (**dragElastic**) specify how easy and smooth the dragging experience is going to be. The higher the number is, the faster the dragging is.

Let's see both options in action.

```diff
+++ a/BeerLogo.jsx
// src/components/BeerLogo.jsx
/*
  The SVG used in this component was downloaded from https://www.svgrepo.com
  For demo purposes during Framer Motion Demo only.
*/

import React from 'react';
import { motion } from 'framer-motion';

export const BeerLogo = () => {
  return (
    <motion.header
      className="flex flex-wrap justify-center w-full"
      drag
+     dragConstraints={{ left: 150, top: 0, bottom: 0, right: 0 }}
+     dragElastic={0.3}
    >
      <div className="w-20 mt-10 mb-5">
        <svg
          version="1.1"
          id="Beer"
          xmlns="http://www.w3.org/2000/svg"
          xmlnsXlink="http://www.w3.org/1999/xlink"
          x="0px"
          y="0px"
          viewBox="0 0 485 485"
          xmlSpace="preserve"
        >
          <g>
            <path
              d="M379.793,141.535l2.621-80.154h-16.163C359.26,26.423,328.34,0,291.357,0h-97.715c-36.983,0-67.903,26.423-74.894,61.381
		h-16.163l2.62,80.156c-9.567,12.064-14.77,26.915-14.77,42.474c0,16.985,6.367,33.374,17.657,45.898L116.431,485h252.132
		l8.341-255.088c11.292-12.524,17.66-28.914,17.66-45.901C394.563,168.452,389.361,153.601,379.793,141.535z M193.643,30h97.715
		c20.325,0,37.627,13.148,43.878,31.381h-185.47C156.016,43.148,173.317,30,193.643,30z M351.996,212.465l-4.677,4.266L339.527,455
		H145.466l-7.787-238.271l-4.676-4.266c-7.986-7.285-12.566-17.655-12.566-28.453c0-9.99,3.822-19.465,10.763-26.678l4.39-4.563
		l-2.006-61.39h217.834l-2.007,61.388l4.39,4.563c6.941,7.214,10.764,16.688,10.764,26.68
		C364.563,194.81,359.982,205.181,351.996,212.465z"
            />
          </g>
        </svg>
      </div>
    </motion.header>
  );
};
```

The latest snapshot of the `BeerLogo.jsx` looks like as the following:

```javascript
// src/components/BeerLogo.jsx
/* 
  The SVG used in this component was downloaded from https://www.svgrepo.com
  For demo purposes during Framer Motion Demo only.
*/

import React from 'react';
import { motion } from 'framer-motion';

export const BeerLogo = () => {
  return (
    <motion.header
      className="flex flex-wrap justify-center w-full"
      drag
      dragConstraints={{ left: 150, top: 0, bottom: 0, right: 0 }}
      dragElastic={0.2}
    >
      <div className="w-20 mt-10 mb-5">
        <svg
          version="1.1"
          id="Beer"
          xmlns="http://www.w3.org/2000/svg"
          xmlnsXlink="http://www.w3.org/1999/xlink"
          x="0px"
          y="0px"
          viewBox="0 0 485 485"
          xmlSpace="preserve"
        >
          <g>
            <path
              d="M379.793,141.535l2.621-80.154h-16.163C359.26,26.423,328.34,0,291.357,0h-97.715c-36.983,0-67.903,26.423-74.894,61.381
		h-16.163l2.62,80.156c-9.567,12.064-14.77,26.915-14.77,42.474c0,16.985,6.367,33.374,17.657,45.898L116.431,485h252.132
		l8.341-255.088c11.292-12.524,17.66-28.914,17.66-45.901C394.563,168.452,389.361,153.601,379.793,141.535z M193.643,30h97.715
		c20.325,0,37.627,13.148,43.878,31.381h-185.47C156.016,43.148,173.317,30,193.643,30z M351.996,212.465l-4.677,4.266L339.527,455
		H145.466l-7.787-238.271l-4.676-4.266c-7.986-7.285-12.566-17.655-12.566-28.453c0-9.99,3.822-19.465,10.763-26.678l4.39-4.563
		l-2.006-61.39h217.834l-2.007,61.388l4.39,4.563c6.941,7.214,10.764,16.688,10.764,26.68
		C364.563,194.81,359.982,205.181,351.996,212.465z"
            />
          </g>
        </svg>
      </div>
    </motion.header>
  );
};
```

## While Hover

We can also trigger an animation while hovering on an element. There is special attribute called `whileHover` which allows us to do that. Let's see that in action.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            initial={{
              x: 0,
              color: 'lightblue',
              opacity: 0.4,
              rotate: 0,
            }}
            animate={{
              color: 'lightcoral',
              textDecoration: 'underline',
              rotate: 5,
              x: '5%',
              opacity: 1,
            }}
            transition={{
              delay: 2,
              type: 'spring',
              stiffness: 1000,
            }}
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
-           <button
+           <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
+             whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
-           </button>
-           <button
+           <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
+             whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
-           </button>
-           <button
+           <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
+             whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
-           </button>
-           <button
+           <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
+           </motion.button>
-           </button>
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
              <div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
              >
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

When we apply an transformation, some elements seem to have a bit strange position from where the animation takes a place. It depends on an element, but you can absolutely customize that behaviour by providing `origin` attribute for an element.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            initial={{
              x: 0,
              color: 'lightblue',
              opacity: 0.4,
              rotate: 0,
            }}
            animate={{
              color: 'lightcoral',
              textDecoration: 'underline',
              rotate: 5,
              x: '5%',
              opacity: 1,
            }}
            transition={{
              delay: 2,
              type: 'spring',
              stiffness: 1000,
            }}
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
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
-             <div
+             <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
+               whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
+             </motion.div>
-             </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

The latest snapshot of the `BeerList.jsx` looks like as the following:

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            initial={{
              x: 0,
              color: 'lightblue',
              opacity: 0.4,
              rotate: 0,
            }}
            animate={{
              color: 'lightcoral',
              textDecoration: 'underline',
              rotate: 5,
              x: '5%',
              opacity: 1,
            }}
            transition={{
              delay: 2,
              type: 'spring',
              stiffness: 1000,
            }}
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
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
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

## Variants

Variants are useful in three areas. They help to **extract individual React Motion elements into an object**, **propagate variant changes through the DOM** and **create a parent/child relationship by using orchestrating properties**. Let's have a closer look on each of these now.

### Extracting individual React Motion element into object

So far, we have been using React Motion by declaring motion objects in `initial`, `animate`, `transition` (and `whileHover`). We can simplify it by creating a motion object separately and using it with `variants` declaration and use the key value as references as shown in the example below.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

+const titleVariants = {
+ standard: {
+   x: 0,
+   color: 'lightblue',
+   opacity: 0.4,
+   rotate: 0,
+ },
+ rotated: {
+   color: 'lightcoral',
+   textDecoration: 'underline',
+   rotate: 5,
+   x: '5%',
+   opacity: 1,
+ },
+};

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
-           initial={{
-             x: 0,
-             color: 'lightblue',
-             opacity: 0.4,
-             rotate: 0,
-           }}
-           animate={{
-             color: 'lightcoral',
-             textDecoration: 'underline',
-             rotate: 5,
-             x: '5%',
-             opacity: 1,
-           }}
+           variants={titleVariants}
+           initial="standard"
+           animate="rotated"
            transition={{
              delay: 2,
              type: 'spring',
              stiffness: 1000,
            }}
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
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
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

You can reference most of the elements in the same way. One exception is `transition` which you can apply directly in the variant object definition on the element where you require to have that animation.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

const titleVariants = {
  standard: {
    x: 0,
    color: 'lightblue',
    opacity: 0.4,
    rotate: 0,
  },
  rotated: {
    color: 'lightcoral',
    textDecoration: 'underline',
    rotate: 5,
    x: '5%',
    opacity: 1,
+   transition: {
+     delay: 2,
+     type: 'spring',
+     stiffness: 1000,
+   },
  },
};

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            variants={titleVariants}
            initial="standard"
            animate="rotated"
-           transition={{
-             delay: 2,
-             type: 'spring',
-             stiffness: 1000,
-           }}
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
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
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

### Propagate variant changes through the DOM

When you have parent and child HTML elements, the parent can propagate the key values into the children, so in the child you can omit them. Let's see that in action.

```diff
--- a/BeerDetail.jsx
+++ b/BeerDetail.jsx
// src/components/BeerDetail.jsx
import React from 'react';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../utils/mocks';
+import { motion } from 'framer-motion';

+const parentVariants = {
+ hidden: {
+   x: '-100vh',
+   opacity: 0,
+ },
+ visible: {
+   x: 0,
+   opacity: 1,
+   transition: {
+     delay: 1,
+     type: 'spring',
+     stiffness: 1000,
+   },
+ },
+};
+
+const childVariants = {
+ hidden: {
+   opacity: 0,
+ },
+ visible: {
+   opacity: 1,
+   rotate: 2,
+ },
+};

export const BeerDetail = ({ handleModalOpen }) => {
  const { id } = useParams();
  const history = useHistory();

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
    },
  );

  React.useEffect(() => {
    try {
      const beer = getBeerById(id);
      if (!beer) {
        throw new ReferenceError(`Beer with id: ${id} doesn't exist!`);
      }
      setState({ beer });
    } catch (error) {
      history.push('/');
    }
  }, [id]);

  const { beer } = state;

  return (
    <>
      <section className="text-gray-700 body-font overflow-hidden bg-white z-0">
-       <div className="container px-5 py-24 mx-auto">
+       <motion.div
+         className="container px-5 py-24 mx-auto"
+         variants={parentVariants}
+         initial="hidden"
+         animate="visible"
+       >
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
              <h1 className="text-gray-900 text-3xl title-font font-medium mb-1 mt-2">
                {beer?.name}
              </h1>
              <p className="leading-relaxed mt-3">{beer?.description}</p>
-             <div className="flex mt-4">
+             <motion.div
+               className="flex mt-4"
+               variants={childVariants}
+               initial="hidden"
+               animate="visible"
+             >
                <button
                  className="flex text-white bg-green-500 border-0 mr-5 py-2 px-6 focus:outline-none hover:bg-green-500 rounded"
                  onClick={handleModalOpen}
                >
                  Show a modal
                </button>
                <button
                  className="flex text-white bg-red-400 border-0 py-2 px-6 focus:outline-none hover:bg-red-400 rounded"
                  onClick={() => history.push('/')}
                >
                  Back
                </button>
+             </motion.div>
-             </div>
            </div>
          </div>
+       </motion.div>
-       </div>
      </section>
    </>
  );
};
```

And because the keys are the same in both objects (`parentVariants`, `childVariants`), you can simplify the code as the following:

```diff
--- a/BeerDetail.jsx
// src/components/BeerDetail.jsx
import React from 'react';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../utils/mocks';
import { motion } from 'framer-motion';

const parentVariants = {
  hidden: {
    x: '-100vh',
    opacity: 0,
  },
  visible: {
    x: 0,
    opacity: 1,
    transition: {
      delay: 1,
      type: 'spring',
      stiffness: 1000,
    },
  },
};

const childVariants = {
  hidden: {
    opacity: 0,
  },
  visible: {
    opacity: 1,
    rotate: 2,
  },
};

export const BeerDetail = ({ handleModalOpen }) => {
  const { id } = useParams();
  const history = useHistory();

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
    },
  );

  React.useEffect(() => {
    try {
      const beer = getBeerById(id);
      if (!beer) {
        throw new ReferenceError(`Beer with id: ${id} doesn't exist!`);
      }
      setState({ beer });
    } catch (error) {
      history.push('/');
    }
  }, [id]);

  const { beer } = state;

  return (
    <>
      <section className="text-gray-700 body-font overflow-hidden bg-white z-0">
        <motion.div
          className="container px-5 py-24 mx-auto"
          variants={parentVariants}
          initial="hidden"
          animate="visible"
        >
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
              <h1 className="text-gray-900 text-3xl title-font font-medium mb-1 mt-2">
                {beer?.name}
              </h1>
              <p className="leading-relaxed mt-3">{beer?.description}</p>
              <motion.div
                className="flex mt-4"
                variants={childVariants}
-               initial="hidden"
-               animate="visible"
              >
                <button
                  className="flex text-white bg-green-500 border-0 mr-5 py-2 px-6 focus:outline-none hover:bg-green-500 rounded"
                  onClick={handleModalOpen}
                >
                  Show a modal
                </button>
                <button
                  className="flex text-white bg-red-400 border-0 py-2 px-6 focus:outline-none hover:bg-red-400 rounded"
                  onClick={() => history.push('/')}
                >
                  Back
                </button>
              </motion.div>
            </div>
          </div>
        </motion.div>
      </section>
    </>
  );
};
```

And you should get the same result.

### Create a parent/child relationship by using orchestrating properties

In the previous example, we introduced a parent/child animation. However, there is a little problem. They both happen at the same time which is a problem when you would like to create some sort of a dependency. Fortunately, variants allow you to achieve that quite easily by adding `when: beforeChildren` into the parent variant.

> Note: You can also use `afterChildren`, which triggers the animation once the child one is completed.

```diff
// src/components/BeerDetail.jsx
+++ a/BeerDetail.jsx
import React from 'react';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../utils/mocks';
import { motion } from 'framer-motion';

const parentVariants = {
  hidden: {
    x: '-100vh',
    opacity: 0,
  },
  visible: {
    x: 0,
    opacity: 1,
    transition: {
      delay: 1,
      type: 'spring',
      stiffness: 1000,
+     when: 'beforeChildren',
    },
  },
};

const childVariants = {
  hidden: {
    opacity: 0,
  },
  visible: {
    opacity: 1,
    rotate: 2,
  },
};

export const BeerDetail = ({ handleModalOpen }) => {
  const { id } = useParams();
  const history = useHistory();

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
    },
  );

  React.useEffect(() => {
    try {
      const beer = getBeerById(id);
      if (!beer) {
        throw new ReferenceError(`Beer with id: ${id} doesn't exist!`);
      }
      setState({ beer });
    } catch (error) {
      history.push('/');
    }
  }, [id]);

  const { beer } = state;

  return (
    <>
      <section className="text-gray-700 body-font overflow-hidden bg-white z-0">
        <motion.div
          className="container px-5 py-24 mx-auto"
          variants={parentVariants}
          initial="hidden"
          animate="visible"
        >
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
              <h1 className="text-gray-900 text-3xl title-font font-medium mb-1 mt-2">
                {beer?.name}
              </h1>
              <p className="leading-relaxed mt-3">{beer?.description}</p>
              <motion.div className="flex mt-4" variants={childVariants}>
                <button
                  className="flex text-white bg-green-500 border-0 mr-5 py-2 px-6 focus:outline-none hover:bg-green-500 rounded"
                  onClick={handleModalOpen}
                >
                  Show a modal
                </button>
                <button
                  className="flex text-white bg-red-400 border-0 py-2 px-6 focus:outline-none hover:bg-red-400 rounded"
                  onClick={() => history.push('/')}
                >
                  Back
                </button>
              </motion.div>
            </div>
          </div>
        </motion.div>
      </section>
    </>
  );
};
```

If you animation type is `spring`, you can also utilize two additional attributes: `mass` and `damping`. The first one (`mass`) is about speed of the spring animation. The lower the number is, the faster the spring effect is.

The option `damping` is about the oscillation part. If you set it to `0`, the oscillation is going to be indefinite. The higher the number is in this case, the less opposing force during the oscillation you can expect.

```diff
--- a/BeerDetail.jsx
+++ b/BeerDetail.jsx
import React from 'react';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../utils/mocks';
import { motion } from 'framer-motion';

const parentVariants = {
  hidden: {
    x: '-100vh',
    opacity: 0,
  },
  visible: {
    x: 0,
    opacity: 1,
    transition: {
-     delay: 1,
+     mass: 0.4,
+     damping: 6,
      type: 'spring',
      stiffness: 1000,
      when: 'beforeChildren',
    },
  },
};

const childVariants = {
  hidden: {
    opacity: 0,
  },
  visible: {
    opacity: 1,
    rotate: 2,
  },
};

export const BeerDetail = ({ handleModalOpen }) => {
  const { id } = useParams();
  const history = useHistory();

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
    },
  );

  React.useEffect(() => {
    try {
      const beer = getBeerById(id);
      if (!beer) {
        throw new ReferenceError(`Beer with id: ${id} doesn't exist!`);
      }
      setState({ beer });
    } catch (error) {
      history.push('/');
    }
  }, [id]);

  const { beer } = state;

  return (
    <>
      <section className="text-gray-700 body-font overflow-hidden bg-white z-0">
        <motion.div
          className="container px-5 py-24 mx-auto"
          variants={parentVariants}
          initial="hidden"
          animate="visible"
        >
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
              <h1 className="text-gray-900 text-3xl title-font font-medium mb-1 mt-2">
                {beer?.name}
              </h1>
              <p className="leading-relaxed mt-3">{beer?.description}</p>
              <motion.div className="flex mt-4" variants={childVariants}>
                <button
                  className="flex text-white bg-green-500 border-0 mr-5 py-2 px-6 focus:outline-none hover:bg-green-500 rounded"
                  onClick={handleModalOpen}
                >
                  Show a modal
                </button>
                <button
                  className="flex text-white bg-red-400 border-0 py-2 px-6 focus:outline-none hover:bg-red-400 rounded"
                  onClick={() => history.push('/')}
                >
                  Back
                </button>
              </motion.div>
            </div>
          </div>
        </motion.div>
      </section>
    </>
  );
};
```

The latest snapshots of the `BeerList.jsx` and `BeerDetail.jsx` look like as the following:

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

const titleVariants = {
  standard: {
    x: 0,
    color: 'lightblue',
    opacity: 0.4,
    rotate: 0,
  },
  rotated: {
    color: 'lightcoral',
    textDecoration: 'underline',
    rotate: 5,
    x: '5%',
    opacity: 1,
    transition: {
      delay: 2,
      type: 'spring',
      stiffness: 1000,
    },
  },
};

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            variants={titleVariants}
            initial="standard"
            animate="rotated"
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
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
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
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
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../utils/mocks';
import { motion } from 'framer-motion';

const parentVariants = {
  hidden: {
    x: '-100vh',
    opacity: 0,
  },
  visible: {
    x: 0,
    opacity: 1,
    transition: {
      mass: 0.4,
      damping: 6,
      type: 'spring',
      stiffness: 1000,
      when: 'beforeChildren',
    },
  },
};

const childVariants = {
  hidden: {
    opacity: 0,
  },
  visible: {
    opacity: 1,
    rotate: 2,
  },
};

export const BeerDetail = ({ handleModalOpen }) => {
  const { id } = useParams();
  const history = useHistory();

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
    },
  );

  React.useEffect(() => {
    try {
      const beer = getBeerById(id);
      if (!beer) {
        throw new ReferenceError(`Beer with id: ${id} doesn't exist!`);
      }
      setState({ beer });
    } catch (error) {
      history.push('/');
    }
  }, [id]);

  const { beer } = state;

  return (
    <>
      <section className="text-gray-700 body-font overflow-hidden bg-white z-0">
        <motion.div
          className="container px-5 py-24 mx-auto"
          variants={parentVariants}
          initial="hidden"
          animate="visible"
        >
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
              <h1 className="text-gray-900 text-3xl title-font font-medium mb-1 mt-2">
                {beer?.name}
              </h1>
              <p className="leading-relaxed mt-3">{beer?.description}</p>
              <motion.div className="flex mt-4" variants={childVariants}>
                <button
                  className="flex text-white bg-green-500 border-0 mr-5 py-2 px-6 focus:outline-none hover:bg-green-500 rounded"
                  onClick={handleModalOpen}
                >
                  Show a modal
                </button>
                <button
                  className="flex text-white bg-red-400 border-0 py-2 px-6 focus:outline-none hover:bg-red-400 rounded"
                  onClick={() => history.push('/')}
                >
                  Back
                </button>
              </motion.div>
            </div>
          </div>
        </motion.div>
      </section>
    </>
  );
};
```

## Key Frames

If you need to repeat certain animation multiple times, `keyframes` are a very useful technique. Let's see that on a simple example.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

const titleVariants = {
  standard: {
    x: 0,
    color: 'lightblue',
    opacity: 0.4,
    rotate: 0,
  },
  rotated: {
    color: 'lightcoral',
    textDecoration: 'underline',
    rotate: 5,
    x: '5%',
    opacity: 1,
    transition: {
      delay: 2,
      type: 'spring',
      stiffness: 1000,
    },
  },
};

+const imageVariants = {
+ hover: {
+   rotate: [20, -60, 20],
+ },
+};

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            variants={titleVariants}
            initial="standard"
            animate="rotated"
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
-             <div className="col-span-2 sm:col-span-1 xl:col-span-1">
+             <motion.div
+               className="col-span-2 sm:col-span-1 xl:col-span-1"
+               variants={imageVariants}
+               whileHover="hover"
+             >
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
-             </div>
+             </motion.div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

When you hover over an image, it triggers a sequence define in the array. But handling things in this way is a bit more difficult than it should be. You can make a few tweaks.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

const titleVariants = {
  standard: {
    x: 0,
    color: 'lightblue',
    opacity: 0.4,
    rotate: 0,
  },
  rotated: {
    color: 'lightcoral',
    textDecoration: 'underline',
    rotate: 5,
    x: '5%',
    opacity: 1,
    transition: {
      delay: 2,
      type: 'spring',
      stiffness: 1000,
    },
  },
};

const imageVariants = {
  hover: {
-   rotate: [20, -60, 20],
+   rotate: 20,
+   transition: {
+     repeat: 20, // or Infinity
+     repeatType: 'mirror',
+   },
  },
};

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            variants={titleVariants}
            initial="standard"
            animate="rotated"
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                variants={imageVariants}
                whileHover="hover"
              >
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </motion.div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

The latest snapshot of the `BeerList.jsx` looks like as the following:

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

const titleVariants = {
  standard: {
    x: 0,
    color: 'lightblue',
    opacity: 0.4,
    rotate: 0,
  },
  rotated: {
    color: 'lightcoral',
    textDecoration: 'underline',
    rotate: 5,
    x: '5%',
    opacity: 1,
    transition: {
      delay: 2,
      type: 'spring',
      stiffness: 1000,
    },
  },
};

const imageVariants = {
  hover: {
    rotate: 20,
    transition: {
      repeat: 10, // or Infinity
      repeatType: 'mirror',
    },
  },
};

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            variants={titleVariants}
            initial="standard"
            animate="rotated"
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                variants={imageVariants}
                whileHover="hover"
              >
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </motion.div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

## Animate Presence

Animate Presence enables the `exit` option which defines what happens, when UI changes after an element is removed from DOM (e.g. when a route is changed).

### Removing element from DOM

Before we jump on route changes, let's just create a quick example. There is a special High-Order Component `Animate Presence` you need tom import from `framer-motion`.

You can wrap any motion element which would enable an exit option where you can put your motion definition, e.g. `<AnimatePresence><motion.X exit={{}}></motion.X></AnimatePresence>`. It will be triggered when that element is going to be removed from DOM. Let's see that in action.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
-import { motion } from 'framer-motion';
+import { motion, AnimatePresence } from 'framer-motion';

const titleVariants = {
  standard: {
    x: 0,
    color: 'lightblue',
    opacity: 0.4,
    rotate: 0,
  },
  rotated: {
    color: 'lightcoral',
    textDecoration: 'underline',
    rotate: 5,
    x: '5%',
    opacity: 1,
    transition: {
      delay: 2,
      type: 'spring',
      stiffness: 1000,
    },
  },
};

const imageVariants = {
  hover: {
    rotate: 20,
    transition: {
      repeat: 10, // or Infinity
      repeatType: 'mirror',
    },
  },
};

export const BeerList = () => {
+ const [showComponent, setShowComponent] = React.useState(true);
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
+   setTimeout(() => setShowComponent(false), 2000);
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            variants={titleVariants}
            initial="standard"
            animate="rotated"
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
+         <AnimatePresence>
+           {showComponent && (
              <motion.p
                className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
                initial={{ y: 50, scale: 0.2 }}
                animate={{ x: 20, y: -10, scale: 1.2 }}
                transition={{
                  type: 'spring',
                  stiffness: 200,
                }}
+               exit={{ scale: 0.5 }}
              >
                Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
                capabalities with sample mocks based on an unofficial Brewdog's API.
              </motion.p>
+           )}
+         </AnimatePresence>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                variants={imageVariants}
                whileHover="hover"
              >
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </motion.div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

This would remove the wrapped paragraph after 2 seconds, together with a simple transformation.

### Route Changing

The previous example was useful, but not very practical. A better way how to utilize the `AnimatePresence` is during route changes. Let's achieve it in a few steps with the first one redoing the changes from the last step.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
-import { motion, AnimatePresence } from 'framer-motion';
+import { motion } from 'framer-motion';

const titleVariants = {
  standard: {
    x: 0,
    color: 'lightblue',
    opacity: 0.4,
    rotate: 0,
  },
  rotated: {
    color: 'lightcoral',
    textDecoration: 'underline',
    rotate: 5,
    x: '5%',
    opacity: 1,
    transition: {
      delay: 2,
      type: 'spring',
      stiffness: 1000,
    },
  },
};

const imageVariants = {
  hover: {
    rotate: 20,
    transition: {
      repeat: 10, // or Infinity
      repeatType: 'mirror',
    },
  },
};

export const BeerList = () => {
- const [showComponent, setShowComponent] = React.useState(true);
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
-   setTimeout(() => setShowComponent(false), 2000);
  }, []);

  const { beers } = state;

  return (
    <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            variants={titleVariants}
            initial="standard"
            animate="rotated"
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
-         <AnimatePresence>
-           {showComponent && (
              <motion.p
                className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
                initial={{ y: 50, scale: 0.2 }}
                animate={{ x: 20, y: -10, scale: 1.2 }}
                transition={{
                  type: 'spring',
                  stiffness: 200,
                }}
-               exit={{ scale: 0.5 }}
              >
                Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
                capabalities with sample mocks based on an unofficial Brewdog's API.
              </motion.p>
-           )}
-         </AnimatePresence>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                variants={imageVariants}
                whileHover="hover"
              >
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </motion.div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};
```

Another step is to update `App.jsx` where our Routing definition is situated.

```diff
--- a/App.jsx
+++ b/App.jsx
// App.jsx
import React from 'react';
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Redirect,
} from 'react-router-dom';
import { BeerDetailModal } from './components/BeerDetailModal';
import { BeerDetail } from './components/BeerDetail';
import { BeerList } from './components/BeerList';
+import { AnimatePresence } from 'framer-motion';

function App() {
  const [showModal, setShowModal] = React.useState(false);

  const openModal = () => {
    setShowModal(true);
  };

  const closeModal = () => {
    setShowModal(false);
  };

  return (
    <>
      <BeerDetailModal showModal={showModal} handleModalClose={closeModal} />
      <Router>
        <Route
          render={({ location }) => (
+           <AnimatePresence exitBeforeEnter>
-             <Switch>
+             <Switch location={location} key={location.pathname}>
                <Route exact path="/beers">
                  <BeerList />
                </Route>
                <Route path="/beers/:id">
                  <BeerDetail handleModalOpen={openModal} />
                </Route>
                <Redirect from="/" to="/beers" />
              </Switch>
+           </AnimatePresence>
          )}
        />
      </Router>
    </>
  );
}

export default App;
```

After you wrap your Switch component, you can define your `exit` objects in the each individual child.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

const titleVariants = {
  standard: {
    x: 0,
    color: 'lightblue',
    opacity: 0.4,
    rotate: 0,
  },
  rotated: {
    color: 'lightcoral',
    textDecoration: 'underline',
    rotate: 5,
    x: '5%',
    opacity: 1,
    transition: {
      delay: 2,
      type: 'spring',
      stiffness: 1000,
    },
  },
};

const imageVariants = {
  hover: {
    rotate: 20,
    transition: {
      repeat: 10, // or Infinity
      repeatType: 'mirror',
    },
  },
};

+const containerVariants = {
+  exit: {
+    rotate: 40,
+  },
+};

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
-   <div id="menu" className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0">
+   <motion.div
+     id="menu"
+     className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0"
+     variants={containerVariants}
+     exit="exit"
+   >
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            variants={titleVariants}
            initial="standard"
            animate="rotated"
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                variants={imageVariants}
                whileHover="hover"
              >
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </motion.div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
            </React.Fragment>
          ))}
        </div>
      </div>
+   </motion.div>
-   </div>
  );
};
```

```diff
--- a/BeerDetail.jsx
+++ b/BeerDetail.jsx
// src/components/BeerDetail.jsx
import React from 'react';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../utils/mocks';
import { motion } from 'framer-motion';

const parentVariants = {
  hidden: {
    x: '-100vh',
    opacity: 0,
  },
  visible: {
    x: 0,
    opacity: 1,
    transition: {
      mass: 0.4,
      damping: 6,
      type: 'spring',
      stiffness: 1000,
      when: 'beforeChildren',
    },
  },
};

const childVariants = {
  hidden: {
    opacity: 0,
  },
  visible: {
    opacity: 1,
    rotate: 2,
  },
};

+const containerVariants = {
+  exit: {
+    rotate: 40,
+  },
+};

export const BeerDetail = ({ handleModalOpen }) => {
  const { id } = useParams();
  const history = useHistory();

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
    },
  );

  React.useEffect(() => {
    try {
      const beer = getBeerById(id);
      if (!beer) {
        throw new ReferenceError(`Beer with id: ${id} doesn't exist!`);
      }
      setState({ beer });
    } catch (error) {
      history.push('/');
    }
  }, [id]);

  const { beer } = state;

  return (
    <>
-     <section className="text-gray-700 body-font overflow-hidden bg-white z-0">
+     <motion.section
+       className="text-gray-700 body-font overflow-hidden bg-white z-0"
+       variants={containerVariants}
+       exit="exit"
+     >
        <motion.div
          className="container px-5 py-24 mx-auto"
          variants={parentVariants}
          initial="hidden"
          animate="visible"
        >
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
              <h1 className="text-gray-900 text-3xl title-font font-medium mb-1 mt-2">
                {beer?.name}
              </h1>
              <p className="leading-relaxed mt-3">{beer?.description}</p>
              <motion.div className="flex mt-4" variants={childVariants}>
                <button
                  className="flex text-white bg-green-500 border-0 mr-5 py-2 px-6 focus:outline-none hover:bg-green-500 rounded"
                  onClick={handleModalOpen}
                >
                  Show a modal
                </button>
                <button
                  className="flex text-white bg-red-400 border-0 py-2 px-6 focus:outline-none hover:bg-red-400 rounded"
                  onClick={() => history.push('/')}
                >
                  Back
                </button>
              </motion.div>
            </div>
          </div>
        </motion.div>
+     </motion.section>
-     </section>
    </>
  );
};
```

### Modal Screens

We can also animate modal windows. However, there is also a special option for `AnimatePresence` which allows to handle some extra use-cases. At the moment, when you open a modal window and click on the back button in the browser, the modal stays open. You can use `onExitComplete` option in the `AnimatePresence` which is triggered after the exit transition is completed.

```diff
--- a/App.jsx
+++ b/App.jsx
// App.jsx
import React from 'react';
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Redirect,
} from 'react-router-dom';
import { BeerDetailModal } from './components/BeerDetailModal';
import { BeerDetail } from './components/BeerDetail';
import { BeerList } from './components/BeerList';
import { AnimatePresence } from 'framer-motion';

function App() {
  const [showModal, setShowModal] = React.useState(false);

  const openModal = () => {
    setShowModal(true);
  };

  const closeModal = () => {
    setShowModal(false);
  };

  return (
    <>
      <BeerDetailModal showModal={showModal} handleModalClose={closeModal} />
      <Router>
        <Route
          render={({ location }) => (
-           <AnimatePresence exitBeforeEnter>
+           <AnimatePresence exitBeforeEnter onExitComplete={closeModal}>
              <Switch location={location} key={location.pathname}>
                <Route exact path="/beers">
                  <BeerList />
                </Route>
                <Route path="/beers/:id">
                  <BeerDetail handleModalOpen={openModal} />
                </Route>
                <Redirect from="/" to="/beers" />
              </Switch>
            </AnimatePresence>
          )}
        />
      </Router>
    </>
  );
}

export default App;
```

This will make sure the modal is always closed when route changes.

The latest snapshots of the `App.jsx`, `BeerList.jsx` and `BeerDetail.jsx` look like as the following:

```javascript
// App.jsx
import React from 'react';
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Redirect,
} from 'react-router-dom';
import { BeerDetailModal } from './components/BeerDetailModal';
import { BeerDetail } from './components/BeerDetail';
import { BeerList } from './components/BeerList';
import { AnimatePresence } from 'framer-motion';

function App() {
  const [showModal, setShowModal] = React.useState(false);

  const openModal = () => {
    setShowModal(true);
  };

  const closeModal = () => {
    setShowModal(false);
  };

  return (
    <>
      <BeerDetailModal showModal={showModal} handleModalClose={closeModal} />
      <Router>
        <Route
          render={({ location }) => (
            <AnimatePresence exitBeforeEnter onExitComplete={closeModal}>
              <Switch location={location} key={location.pathname}>
                <Route exact path="/beers">
                  <BeerList />
                </Route>
                <Route path="/beers/:id">
                  <BeerDetail handleModalOpen={openModal} />
                </Route>
                <Redirect from="/" to="/beers" />
              </Switch>
            </AnimatePresence>
          )}
        />
      </Router>
    </>
  );
}

export default App;
```

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion } from 'framer-motion';

const titleVariants = {
  standard: {
    x: 0,
    color: 'lightblue',
    opacity: 0.4,
    rotate: 0,
  },
  rotated: {
    color: 'lightcoral',
    textDecoration: 'underline',
    rotate: 5,
    x: '5%',
    opacity: 1,
    transition: {
      delay: 2,
      type: 'spring',
      stiffness: 1000,
    },
  },
};

const imageVariants = {
  hover: {
    rotate: 20,
    transition: {
      repeat: 10, // or Infinity
      repeatType: 'mirror',
    },
  },
};

const containerVariants = {
  exit: {
    rotate: 40,
  },
};

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <motion.div
      id="menu"
      className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0"
      variants={containerVariants}
      exit="exit"
    >
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            variants={titleVariants}
            initial="standard"
            animate="rotated"
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                variants={imageVariants}
                whileHover="hover"
              >
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </motion.div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </motion.div>
  );
};
```

```javascript
// src/components/BeerDetail.jsx
import React from 'react';
import { useParams, useHistory } from 'react-router-dom';
import { getBeerById } from '../utils/mocks';
import { motion } from 'framer-motion';

const parentVariants = {
  hidden: {
    x: '-100vh',
    opacity: 0,
  },
  visible: {
    x: 0,
    opacity: 1,
    transition: {
      mass: 0.4,
      damping: 6,
      type: 'spring',
      stiffness: 1000,
      when: 'beforeChildren',
    },
  },
};

const childVariants = {
  hidden: {
    opacity: 0,
  },
  visible: {
    opacity: 1,
    rotate: 2,
  },
};

const containerVariants = {
  exit: {
    rotate: 40,
  },
};

export const BeerDetail = ({ handleModalOpen }) => {
  const { id } = useParams();
  const history = useHistory();

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
    },
  );

  React.useEffect(() => {
    try {
      const beer = getBeerById(id);
      if (!beer) {
        throw new ReferenceError(`Beer with id: ${id} doesn't exist!`);
      }
      setState({ beer });
    } catch (error) {
      history.push('/');
    }
  }, [id]);

  const { beer } = state;

  return (
    <>
      <motion.section
        className="text-gray-700 body-font overflow-hidden bg-white z-0"
        variants={containerVariants}
        exit="exit"
      >
        <motion.div
          className="container px-5 py-24 mx-auto"
          variants={parentVariants}
          initial="hidden"
          animate="visible"
        >
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
              <h1 className="text-gray-900 text-3xl title-font font-medium mb-1 mt-2">
                {beer?.name}
              </h1>
              <p className="leading-relaxed mt-3">{beer?.description}</p>
              <motion.div className="flex mt-4" variants={childVariants}>
                <button
                  className="flex text-white bg-green-500 border-0 mr-5 py-2 px-6 focus:outline-none hover:bg-green-500 rounded"
                  onClick={handleModalOpen}
                >
                  Show a modal
                </button>
                <button
                  className="flex text-white bg-red-400 border-0 py-2 px-6 focus:outline-none hover:bg-red-400 rounded"
                  onClick={() => history.push('/')}
                >
                  Back
                </button>
              </motion.div>
            </div>
          </div>
        </motion.div>
      </motion.section>
    </>
  );
};
```

## React Motion and 

We can easily apply motion animation to SVG elements (`svg`, `path`). Let's have a look at a simple example how to do that.

```diff
--- a/BeerLogo.jsx
+++ b/BeerLogo.jsx
// src/components/BeerLogo.jsx
/*
  The SVG used in this component was downloaded from https://www.svgrepo.com
  For demo purposes during Framer Motion Demo only.
*/

import React from 'react';
import { motion } from 'framer-motion';

+const svgVariants = {
+ hidden: { x: '-100vw' },
+ visible: {
+   x: 0,
+   transition: {
+     duration: 1,
+   },
+ },
+};
+
+const pathVariants = {
+ hidden: {
+   opacity: 0,
+   pathLength: 0,
+ },
+ visible: {
+   opacity: 1,
+   pathLength: 1,
+   transition: {
+     duration: 5,
+   },
+ },
+};

export const BeerLogo = () => {
  return (
    <motion.header
      className="flex flex-wrap justify-center w-full"
      drag
      dragConstraints={{ left: 150, top: 0, bottom: 0, right: 0 }}
      dragElastic={0.2}
    >
      <div className="w-20 mt-10 mb-5">
-       <svg
+       <motion.svg
          version="1.1"
          id="Beer"
          xmlns="http://www.w3.org/2000/svg"
          xmlnsXlink="http://www.w3.org/1999/xlink"
          x="0px"
          y="0px"
          viewBox="0 0 485 485"
          xmlSpace="preserve"
+         variants={svgVariants}
+         initial="hidden"
+         animate="visible"
        >
          <g>
-           <path
+           <motion.path
              d="M379.793,141.535l2.621-80.154h-16.163C359.26,26.423,328.34,0,291.357,0h-97.715c-36.983,0-67.903,26.423-74.894,61.381
		h-16.163l2.62,80.156c-9.567,12.064-14.77,26.915-14.77,42.474c0,16.985,6.367,33.374,17.657,45.898L116.431,485h252.132
		l8.341-255.088c11.292-12.524,17.66-28.914,17.66-45.901C394.563,168.452,389.361,153.601,379.793,141.535z M193.643,30h97.715
		c20.325,0,37.627,13.148,43.878,31.381h-185.47C156.016,43.148,173.317,30,193.643,30z M351.996,212.465l-4.677,4.266L339.527,455
		H145.466l-7.787-238.271l-4.676-4.266c-7.986-7.285-12.566-17.655-12.566-28.453c0-9.99,3.822-19.465,10.763-26.678l4.39-4.563
		l-2.006-61.39h217.834l-2.007,61.388l4.39,4.563c6.941,7.214,10.764,16.688,10.764,26.68
		C364.563,194.81,359.982,205.181,351.996,212.465z"
+             variants={pathVariants}
            />
          </g>
-       </svg>
+       </motion.svg>
      </div>
    </motion.header>
  );
};
```

The latest snapshot of the `BeerLogo.jsx` looks like as the following:

```javascript
// src/components/BeerLogo.jsx
/* 
  The SVG used in this component was downloaded from https://www.svgrepo.com
  For demo purposes during Framer Motion Demo only.
*/

import React from 'react';
import { motion } from 'framer-motion';

const svgVariants = {
  hidden: { x: '-100vw' },
  visible: {
    x: 0,
    transition: {
      duration: 1,
    },
  },
};

const pathVariants = {
  hidden: {
    opacity: 0,
    pathLength: 0,
  },
  visible: {
    opacity: 1,
    pathLength: 1,
    transition: {
      duration: 5,
    },
  },
};

export const BeerLogo = () => {
  return (
    <motion.header
      className="flex flex-wrap justify-center w-full"
      drag
      dragConstraints={{ left: 150, top: 0, bottom: 0, right: 0 }}
      dragElastic={0.2}
    >
      <div className="w-20 mt-10 mb-5">
        <motion.svg
          version="1.1"
          id="Beer"
          xmlns="http://www.w3.org/2000/svg"
          xmlnsXlink="http://www.w3.org/1999/xlink"
          x="0px"
          y="0px"
          viewBox="0 0 485 485"
          xmlSpace="preserve"
          variants={svgVariants}
          initial="hidden"
          animate="visible"
        >
          <g>
            <motion.path
              d="M379.793,141.535l2.621-80.154h-16.163C359.26,26.423,328.34,0,291.357,0h-97.715c-36.983,0-67.903,26.423-74.894,61.381
		h-16.163l2.62,80.156c-9.567,12.064-14.77,26.915-14.77,42.474c0,16.985,6.367,33.374,17.657,45.898L116.431,485h252.132
		l8.341-255.088c11.292-12.524,17.66-28.914,17.66-45.901C394.563,168.452,389.361,153.601,379.793,141.535z M193.643,30h97.715
		c20.325,0,37.627,13.148,43.878,31.381h-185.47C156.016,43.148,173.317,30,193.643,30z M351.996,212.465l-4.677,4.266L339.527,455
		H145.466l-7.787-238.271l-4.676-4.266c-7.986-7.285-12.566-17.655-12.566-28.453c0-9.99,3.822-19.465,10.763-26.678l4.39-4.563
		l-2.006-61.39h217.834l-2.007,61.388l4.39,4.563c6.941,7.214,10.764,16.688,10.764,26.68
		C364.563,194.81,359.982,205.181,351.996,212.465z"
              variants={pathVariants}
            />
          </g>
        </motion.svg>
      </div>
    </motion.header>
  );
};
```

## Using useCycle hook

The last thing we are going to cover here is `useCycle` hook which is useful for connecting multiple motion objects together. It accepts string parameters - identifiers of variants that have to be grouped together in single variant object. Hook returns an animation object and setter that allows you to swap between the variants (e.g. with **onClick** action). Let's see that in action.

```diff
--- a/BeerList.jsx
+++ b/BeerList.jsx
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
-import { motion } from 'framer-motion';
+import { motion, useCycle } from 'framer-motion';

const titleVariants = {
  standard: {
    x: 0,
    color: 'lightblue',
    opacity: 0.4,
    rotate: 0,
  },
  rotated: {
    color: 'lightcoral',
    textDecoration: 'underline',
    rotate: 5,
    x: '5%',
    opacity: 1,
    transition: {
      delay: 2,
      type: 'spring',
      stiffness: 1000,
    },
  },
};

const imageVariants = {
- hover: {
+ imageVariantsOne: {
    rotate: 20,
    transition: {
      repeat: 10, // or Infinity
      repeatType: 'mirror',
    },
  },
+ imageVariantsTwo: {
+   rotate: 180,
+   transition: {
+     repeat: 30,
+     repeatType: 'mirror',
+   }
+ }
};

const containerVariants = {
  exit: {
    rotate: 40,
  },
};

export const BeerList = () => {
+ const [animation, cycleAnimation] = useCycle(
+   'imageVariantsOne',
+   'imageVariantsTwo',
+ );
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <motion.div
      id="menu"
      className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0"
      variants={containerVariants}
      exit="exit"
    >
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            variants={titleVariants}
            initial="standard"
            animate="rotated"
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                variants={imageVariants}
-               whileHover="hover"
+               whileHover={animation}
+               onClick={cycleAnimation}
              >
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </motion.div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </motion.div>
  );
};
```

Now we should be able to see an animation while hovering over the image and swap the effect when clicked on it. The latest snapshot of the `BeerList.jsx` looks like as the following:

```javascript
// src/components/BeerList.jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { BeerLogo } from './BeerLogo';
import { getBeerList, getListOfBeersByName } from '../utils/mocks';
import { motion, useCycle } from 'framer-motion';

const titleVariants = {
  standard: {
    x: 0,
    color: 'lightblue',
    opacity: 0.4,
    rotate: 0,
  },
  rotated: {
    color: 'lightcoral',
    textDecoration: 'underline',
    rotate: 5,
    x: '5%',
    opacity: 1,
    transition: {
      delay: 2,
      type: 'spring',
      stiffness: 1000,
    },
  },
};

const imageVariants = {
  imageVariantsOne: {
    rotate: 20,
    transition: {
      repeat: 10, // or Infinity
      repeatType: 'mirror',
    },
  },

  imageVariantsTwo: {
    rotate: 180,
    transition: {
      repeat: 30,
      repeatType: 'mirror',
    },
  },
};

const containerVariants = {
  exit: {
    rotate: 40,
  },
};

export const BeerList = () => {
  const [animation, cycleAnimation] = useCycle(
    'imageVariantsOne',
    'imageVariantsTwo',
  );
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: [],
    },
  );

  const getAllBeers = async () => {
    const beers = getBeerList();
    setState({ beers });
  };

  const getBeerListByType = async (type) => {
    const beers = getListOfBeersByName(type);
    setState({ beers });
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers } = state;

  return (
    <motion.div
      id="menu"
      className="container mx-auto px-4 lg:pt-10 lg:pb-10 z-0"
      variants={containerVariants}
      exit="exit"
    >
      <div className="flex flex-wrap text-center justify-center">
        <div className="w-full lg:w-6/12 px-4">
          <motion.h2
            className="text-4xl font-semibold text-black"
            variants={titleVariants}
            initial="standard"
            animate="rotated"
          >
            Brewdog's Beer Selection
          </motion.h2>
          <BeerLogo />
          <motion.p
            className="text-lg leading-relaxed mt-4 mb-4 text-gray-500"
            initial={{ y: 50, scale: 0.2 }}
            animate={{ x: 20, y: -10, scale: 1.2 }}
            transition={{
              type: 'spring',
              stiffness: 200,
            }}
          >
            Let's enjoy Brewdog's selection of beer whilst testing Framer Motion
            capabalities with sample mocks based on an unofficial Brewdog's API.
          </motion.p>
          <div className="flex justify-center">
            <motion.button
              onClick={getAllBeers}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              All Beers
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('pilsner')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Pilsner
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('lager')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Lager
            </motion.button>
            <motion.button
              onClick={() => getBeerListByType('ale')}
              className="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
              whileHover={{ y: -10 }}
            >
              Ale
            </motion.button>
          </div>
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                variants={imageVariants}
                whileHover={animation}
                onClick={cycleAnimation}
              >
                <img
                  alt={beer.title}
                  src={beer.image_url}
                  className="h-20 w-8 rounded  mx-auto"
                />
              </motion.div>
              <div className="col-span-2 sm:col-span-4 xl:col-span-4">
                <h3 className="font-semibold text-black">{beer.name}</h3>
                <p>
                  {beer.description.slice(0, beer.description.indexOf('.'))}.
                </p>
              </div>
              <motion.div
                className="col-span-2 sm:col-span-1 xl:col-span-1"
                whileHover={{ scale: 1.2, originX: 0 }}
              >
                <Link to={`/beers/${beer.id}`} className="underline">
                  Detail
                </Link>
              </motion.div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </motion.div>
  );
};
```

## Conclusion

Thanks for following this workshop material. Any feedback appreciated (radek.tomasek@gmail.com).
