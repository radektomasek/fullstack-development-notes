# Intro to React Location 

> Note: This workshop handles the legacy React Location. In the foreseeable future, I would like to migrate it to [Tanstack Router](https://tanstack.com/router/latest/docs/framework/react/migrate-from-react-location).

When it comes to UI routing in React, a common choice is to use [React Router](https://reactrouter.com) which, without a doubt is an amazing and proven solution and its recent release of [v6](https://reactrouter.com/docs/en/v6) confirms that.

On the other hand, it is not always easy to configure it properly and sometimes, an upgrade to its newer versions can cause a bit of a headache.

Tanner Lindsey from [TanStack](https://tanstack.com) maintains a project called [React Location](https://react-location.tanstack.com/) which also focuses on React Routing, but where things are managed in more declarative way.

In my option it's very easy to learn and use it. This tutorial helps to teach the basics concepts whilst building a simple application.

## Project Initialization

The principles are going to be demonstrated on the application built for [React Query Intro](https://github.com/ovotech/co_frontend/tree/master/react-query-intro) - the part using the [Brewdog Beer Punk API](https://punkapi.com/documentation/v2).

The boilerplate of the app is going to be setup with [Create-MF-app](https://www.npmjs.com/package/create-mf-app) which is a very cool tool allowing us creating a simple prototype based on almost any JS framework, whilst you can choose between JavaScript/TypeScript and CSS/Tailwind for building simple web apps, API servers or libraries.

As a first step let's initialize the project by typing the following:

```bash
npx create-mf-app
```

There are going to be a set of prompt questions. Let's set them as the following:

- **react-location-intro** (Name of the app)
- **Application** (Project type)
- **8080** (Port Number)
- **React** (Framework)
- **Typescript** (Language)
- **Tailwind** (CSS)

Once the project is initialized, we need to install the modules. You can choose either `yarn` or `npm` , based on your preference. In this example, I stick to the latter.

Let's install the dependencies:

```bash
# if you prefer yarn, feel free to use it instead
cd react-location-intro
npm install
```

As a next step, we need to install `React Location`. It's a pretty straightforward operation, just type the following:

> Note: There is no need to install external types for TypeScript, as the TS support is ready for us out of the box.

```bash
npm install react-location --save
```

We can also add a prettier config with some simple configuration:

```bash
touch .prettierrc
```

```json
{
  "singleQuote": true
}
```

Now, let's create two directories **api** and **pages** by typing:

```bash
mkdir src/{api,pages}
```

**In the api folder**, let's create an `index.ts` file with the following content:

```typescript
// src/api/index.ts
const BASE_URL = 'https://api.punkapi.com/v2';

export interface BeerData {
  id: number;
  name: string;
  tagline: string;
  description: string;
  image_url: string;
}

export const getBeerList = (): Promise<BeerData[]> =>
  fetch(`${BASE_URL}/beers`)
    .then((response) => response.json())
    .then((data) => data);

export const getBeerById = (beerId: number): Promise<BeerData> =>
  fetch(`${BASE_URL}/beers/${beerId}`)
    .then((response) => response.json())
    .then((data) => data[0]);
```

**In the pages folder**, let's create two files: `BeerList.tsx` and `BeerDetail.tsx` and provide the following code:

```typescript
// src/pages/BeerList.jsx
import React from 'react';
import { getBeerList } from '../api';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: null,
      error: null,
      status: 'pending',
    }
  );

  const getAllBeers = async () => {
    setState({ status: 'pending' });
    return getBeerList()
      .then((response) => setState({ beers: response, status: 'success' }))
      .catch((error) => setState({ error: error.message, status: 'error' }));
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers, error, status } = state;

  if (status === 'pending') {
    return <p>Loading...</p>;
  }

  if (status === 'error') {
    return <p>Opps, there was some error..{error}</p>;
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
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.name}
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
                Detail
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};

export default BeerList;
```

```typescript
// src/pages/BeerDetail.jsx
import React from 'react';
import { getBeerById } from '../api';

export const BeerDetail = () => {
  const id = 1; // hardcoded for now

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
      error: null,
      status: 'pending',
    }
  );

  React.useEffect(() => {
    setState({ status: 'pending' });

    getBeerById(id)
      .then((response) => setState({ beer: response, status: 'success' }))
      .catch((error) => setState({ error: error.message, status: 'error' }));
  }, [id]);

  const { beer, error, status } = state;

  if (status === 'pending') {
    return <p>Loading...</p>;
  }

  if (status === 'error') {
    return <p>Opps, there was some error..{error}</p>;
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
                onClick={() => {}}
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

export default BeerDetail;
```

Last thing is the section is to verify whether everything is up and running. Modify the content of the `App.tsx` by removing the placeholder code and replacing it by one of our newly created components

```diff
--- a/App.tsx
+++ b/App.tsx
// App.tsx
import React from "react";
import ReactDOM from "react-dom";
+import BeerList from './pages/BeerList';

import "./index.scss";

-const App = () => (
- <div className="mt-10 text-3xl mx-auto max-w-6xl">
-   <div>Name: react-location-intro</div>
-   <div>Framework: react</div>
-   <div>Language: TypeScript</div>
-   <div>CSS: Tailwind</div>
- </div>
-);
+const App = () => <BeerList />;
ReactDOM.render(<App />, document.getElementById("app"));
```

The latest version of the `App.tsx` should looks as the following sample:

```typescript
import React from 'react';
import ReactDOM from 'react-dom';
import BeerList from './pages/BeerList';

import './index.scss';

const App = () => <BeerList />;
ReactDOM.render(<App />, document.getElementById('app'));
```

Now, we should be able to run the project. If you type `npm run start` and pass `http://localhost:8080` into your browser, you should be able to see the a list of beers as the result.

## Routing initialization

In this section, we are going to cover the core principles of the React Location. The first step is going to be initialization. Let's add a new file called `router.tsx` into the `src` folder with the following content:

```typescript
// src/router.tsx
import React from 'react';
import { Route, ReactLocation } from 'react-location';
import BeerList from './pages/BeerList';
import BeerDetail from './pages/BeerDetail';

export const routes: Route[] = [
  {
    path: '/',
    element: <BeerList />,
  },
  {
    path: 'beer/:id',
    element: <BeerDetail />,
  },
];

export const location = new ReactLocation();
```

> Note: by default, `BrowserHistory` is passed as default value. If you need to use a different type of history object (e.g `HashHistory`), you can import it (`const history = createHashHistory()`) and pass it to the constructor (new ReactLocation({ history })). More information is available here at this link: [https://react-location.tanstack.com/guides/history-types-and-location](https://react-location.tanstack.com/guides/history-types-and-location)

Once our routes are defined, we can incorporate them in the root of our app (`App.tsx`) as the following:

```diff
--- a/App.tsx
+++ b/App.tsx
import React from 'react';
import ReactDOM from 'react-dom';
-import BeerList from './pages/BeerList';
+import { Router, Outlet } from 'react-location';
+import { routes, location } from './router';

import './index.scss';

-const App = () => <BeerList />;
+const App = () => (
+ <Router routes={routes} location={location}>
+   <Outlet />
+ </Router>
+);
ReactDOM.render(<App />, document.getElementById('app'));
```

> Note: The `<Outlet />` is what renders the content of routes.

As we have the routing in place, let's back-fill the links within the app and make sure the detail page is possible visiting without any problem.

Adding links is very straightforward, there is a component called `<Link />` we can just easily add into the code.

Let's do that for `BeerList.tsx` first as the following:

```diff
--- a/BeerList.tsx
+++ b/BeerList.tsx
// src/pages/BeerList.jsx
import React from 'react';
+import { Link } from 'react-location';
import { getBeerList } from '../api';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: null,
      error: null,
      status: 'pending',
    }
  );

  const getAllBeers = async () => {
    setState({ status: 'pending' });
    return getBeerList()
      .then((response) => setState({ beers: response, status: 'success' }))
      .catch((error) => setState({ error: error.message, status: 'error' }));
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers, error, status } = state;

  if (status === 'pending') {
    return <p>Loading...</p>;
  }

  if (status === 'error') {
    return <p>Opps, there was some error..{error}</p>;
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
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.name}
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
-               Detail
+               <Link to={`beer/${beer.id}`}>Detail</Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};

export default BeerList;
```

After you save the file, we should be able to visit the detail page.

> Note: At this point, the ID for data fetching in the detail page is hardcoded. We are going to change it very soon.

On the detail page, let's see first how we can change the navigation programmatically (instead of using `<Link />` component). To be able to do that, we need to apply patterns from the [Navigate API](https://react-location.tanstack.com/docs/api#navigate). Let's do that now in the `BeerDetail.tsx` as the following:

```diff
--- a/BeerDetail.tsx
+++ b/BeerDetail.tsx
// src/pages/BeerDetail.jsx
import React from 'react';
+import { useNavigate} from 'react-location';
import { getBeerById } from '../api';

export const BeerDetail = () => {
+ const navigate = useNavigate();
  const id = 1; // hardcoded for now

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
      error: null,
      status: 'pending',
    }
  );

  React.useEffect(() => {
    setState({ status: 'pending' });

    getBeerById(id)
      .then((response) => setState({ beer: response, status: 'success' }))
      .catch((error) => setState({ error: error.message, status: 'error' }));
  }, [id]);

  const { beer, error, status } = state;

  if (status === 'pending') {
    return <p>Loading...</p>;
  }

  if (status === 'error') {
    return <p>Opps, there was some error..{error}</p>;
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
-               onClick={() => {}}
+               onClick={() => navigate({ to: '/', replace: true })}
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

export default BeerDetail;
```

Adding an ability for fetching data dynamically is also very straightforward. In the same file (`BeerDetail.tsx`) let's add the following code:

```diff
--- a/BeerDetail.tsx
+++ b/BeerDetail.tsx
// src/pages/BeerDetail.jsx
import React from 'react';
-import { useNavigate } from 'react-location';
+import { useNavigate, useMatch } from 'react-location';
import { getBeerById } from '../api';

export const BeerDetail = () => {
  const navigate = useNavigate();
- const id = 1; // hardcoded for now
+ const {
+   params: { id },
+ } = useMatch();

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
      error: null,
      status: 'pending',
    }
  );

  React.useEffect(() => {
    setState({ status: 'pending' });

-   getBeerById(id)
+   getBeerById(parseInt(id))
      .then((response) => setState({ beer: response, status: 'success' }))
      .catch((error) => setState({ error: error.message, status: 'error' }));
  }, [id]);

  const { beer, error, status } = state;

  if (status === 'pending') {
    return <p>Loading...</p>;
  }

  if (status === 'error') {
    return <p>Opps, there was some error..{error}</p>;
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
                onClick={() => navigate({ to: '/', replace: true })}
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

export default BeerDetail;
```

The last thing in this section is a little refactoring of our `router.tsx` file. We can adjust the dynamic route as the following without impacting the functionality.

```diff
--- a/router.tsx
+++ b/router.tsx
// src/router.tsx
import React from 'react';
import { Route, ReactLocation } from 'react-location';
import BeerList from './pages/BeerList';
import BeerDetail from './pages/BeerDetail';

export const routes: Route[] = [
  {
    path: '/',
    element: <BeerList />,
  },
  {
-   path: 'beer/:id',
-   element: <BeerDetail />,
+   path: 'beer',
+   children: [
+     {
+       path: ':id',
+       element: <BeerDetail />,
+     },
+   ],
  },
];

export const location = new ReactLocation();
```

The latest snapshots of the files look as the following:

```typescript
// src/router.tsx
import React from 'react';
import { Route, ReactLocation } from 'react-location';
import BeerList from './pages/BeerList';
import BeerDetail from './pages/BeerDetail';

export const routes: Route[] = [
  {
    path: '/',
    element: <BeerList />,
  },
  {
    path: 'beer',
    children: [
      {
        path: ':id',
        element: <BeerDetail />,
      },
    ],
  },
];

export const location = new ReactLocation();
```

```typescript
// src/pages/BeerList.jsx
import React from 'react';
import { Link } from 'react-location';
import { getBeerList } from '../api';

export const BeerList = () => {
  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beers: null,
      error: null,
      status: 'pending',
    }
  );

  const getAllBeers = async () => {
    setState({ status: 'pending' });
    return getBeerList()
      .then((response) => setState({ beers: response, status: 'success' }))
      .catch((error) => setState({ error: error.message, status: 'error' }));
  };

  React.useEffect(() => {
    getAllBeers();
  }, []);

  const { beers, error, status } = state;

  if (status === 'pending') {
    return <p>Loading...</p>;
  }

  if (status === 'error') {
    return <p>Opps, there was some error..{error}</p>;
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
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.name}
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
                <Link to={`beer/${beer.id}`}>Detail</Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};

export default BeerList;
```

```typescript
// src/pages/BeerDetail.jsx
import React from 'react';
import { useNavigate, useMatch } from 'react-location';
import { getBeerById } from '../api';

export const BeerDetail = () => {
  const navigate = useNavigate();
  const {
    params: { id },
  } = useMatch();

  const [state, setState] = React.useReducer(
    (state, action) => ({ ...state, ...action }),
    {
      beer: null,
      error: null,
      status: 'pending',
    }
  );

  React.useEffect(() => {
    setState({ status: 'pending' });

    getBeerById(parseInt(id))
      .then((response) => setState({ beer: response, status: 'success' }))
      .catch((error) => setState({ error: error.message, status: 'error' }));
  }, [id]);

  const { beer, error, status } = state;

  if (status === 'pending') {
    return <p>Loading...</p>;
  }

  if (status === 'error') {
    return <p>Opps, there was some error..{error}</p>;
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
                onClick={() => navigate({ to: '/', replace: true })}
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

export default BeerDetail;
```

```typescript
// src/App.tsx
import React from 'react';
import ReactDOM from 'react-dom';
import { Router, Outlet } from 'react-location';
import { routes, location } from './router';

import './index.scss';

const App = () => (
  <Router routes={routes} location={location}>
    <Outlet />
  </Router>
);
ReactDOM.render(<App />, document.getElementById('app'));
```

## Async routes

Async route is initialized when it's actually requested. And adding such a route is very straightforward. Let's see an example with both `default import` and `named import`.

In the `router.tsx`. let's make the following changes:

```diff
--- a/router.tsx
+++ b/router.tsx
// src/router.tsx
import React from 'react';
import { Route, ReactLocation } from 'react-location';
-import BeerList from './pages/BeerList';
-import BeerDetail from './pages/BeerDetail';

export const routes: Route[] = [
  {
    path: '/',
-   element: <BeerList />,
+   element: () =>
+     import('./pages/BeerList').then((module) => <module.default />),
  },
  {
    path: 'beer',
    children: [
      {
        path: ':id',
-       element: <BeerDetail />,
+       element: () =>
+         import('./pages/BeerDetail').then((module) => <module.BeerDetail />),
      },
    ],
  },
];

export const location = new ReactLocation();
```

The latest snapshot of the `router.tsx` looks as the following:

```typescript
// src/router.tsx
import React from 'react';
import { Route, ReactLocation } from 'react-location';

export const routes: Route[] = [
  {
    path: '/',
    element: () =>
      import('./pages/BeerList').then((module) => <module.default />),
  },
  {
    path: 'beer',
    children: [
      {
        path: ':id',
        element: () =>
          import('./pages/BeerDetail').then((module) => <module.BeerDetail />),
      },
    ],
  },
];

export const location = new ReactLocation();
```

## Loaders

Loaders are very useful, if we need to make some async operations like fetching the data. We can do that directly in the component as we do now, or using special option within the React Location. Let's see this functionality in action.

As the first step, we need to update `router.tsx` and add **loader** option. We are going to define a type with `MakeGenerics` type that help us working with data much easily.

```diff
--- a/router.tsx
+++ b/router.tsx
// src/router.tsx
import React from 'react';
-import { Route, ReactLocation } from 'react-location';
+import { Route, ReactLocation, MakeGenerics } from 'react-location';
+import { getBeerById, getBeerList } from './api';
+import type { BeerData } from './api';

export const routes: Route[] = [
  {
    path: '/',
+   loader: async () => {
+     return {
+       beers: await getBeerList(),
+     };
+   },
    element: () =>
      import('./pages/BeerList').then((module) => <module.default />),
  },
  {
    path: 'beer',
    children: [
      {
        path: ':id',
+       loader: async ({ params }) => {
+         return {
+           beer: await getBeerById(parseInt(params.id)),
+         };
+       },
        element: () =>
          import('./pages/BeerDetail').then((module) => <module.BeerDetail />),
      },
    ],
  },
];

+export type LocationGenerics = MakeGenerics<{
+ LoaderData: {
+   beers: BeerData[];
+   beer: BeerData;
+ };
+}>;

export const location = new ReactLocation();
```

Once our `router.tsx` file is update, we can adjust our React components as the following:

```diff
--- a/BeerList.tsx
+++ b/BeerList.tsx
// src/pages/BeerList.jsx
import React from 'react';
-import { Link } from 'react-location';
+import { Link, useMatch } from 'react-location';
-import { getBeerList } from '../api';
+import type { LocationGenerics } from '../router';

export const BeerList = () => {
+ const {
+   data: { beers },
+ } = useMatch<LocationGenerics>();

- const [state, setState] = React.useReducer(
-   (state, action) => ({ ...state, ...action }),
-   {
-     beers: null,
-     error: null,
-     status: 'pending',
-   }
- );

- const getAllBeers = async () => {
-   setState({ status: 'pending' });
-   return getBeerList()
-     .then((response) => setState({ beers: response, status: 'success' }))
-     .catch((error) => setState({ error: error.message, status: 'error' }));
- };

- React.useEffect(() => {
-   getAllBeers();
- }, []);

- const { beers, error, status } = state;

- if (status === 'pending') {
-   return <p>Loading...</p>;
- }

- if (status === 'error') {
-   return <p>Opps, there was some error..{error}</p>;
- }

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
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.name}
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
                <Link to={`beer/${beer.id}`}>Detail</Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};

export default BeerList;
```

```diff
--- a/BeerDetail.tsx
+++ b/BeerDetail.tsx
// src/pages/BeerDetail.jsx
import React from 'react';
import { useNavigate, useMatch } from 'react-location';
-import { getBeerById } from '../api';
+import type { LocationGenerics } from '../router';

export const BeerDetail = () => {
  const navigate = useNavigate();
  const {
-   params: { id },
+   data: { beer },
- } = useMatch();
+ } = useMatch<LocationGenerics>();

- const [state, setState] = React.useReducer(
-   (state, action) => ({ ...state, ...action }),
-   {
-     beer: null,
-     error: null,
-     status: 'pending',
-   }
- );

- React.useEffect(() => {
-   setState({ status: 'pending' });

-   getBeerById(parseInt(id))
-     .then((response) => setState({ beer: response, status: 'success' }))
-     .catch((error) => setState({ error: error.message, status: 'error' }));
- }, [id]);

- const { beer, error, status } = state;

- if (status === 'pending') {
-   return <p>Loading...</p>;
- }

- if (status === 'error') {
-   return <p>Opps, there was some error..{error}</p>;
- }

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
                onClick={() => navigate({ to: '/', replace: true })}
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

export default BeerDetail;
```

The latest snapshots of each component look as the following:

```typescript
// src/router.tsx
import React from 'react';
import { Route, ReactLocation, MakeGenerics } from 'react-location';
import { getBeerById, getBeerList } from './api';
import type { BeerData } from './api';

export const routes: Route[] = [
  {
    path: '/',
    loader: async () => {
      return {
        beers: await getBeerList(),
      };
    },
    element: () =>
      import('./pages/BeerList').then((module) => <module.default />),
  },
  {
    path: 'beer',
    children: [
      {
        path: ':id',
        loader: async ({ params }) => {
          return {
            beer: await getBeerById(parseInt(params.id)),
          };
        },
        element: () =>
          import('./pages/BeerDetail').then((module) => <module.BeerDetail />),
      },
    ],
  },
];

export type LocationGenerics = MakeGenerics<{
  LoaderData: {
    beers: BeerData[];
    beer: BeerData;
  };
}>;

export const location = new ReactLocation();
```

```typescript
// src/pages/BeerList.jsx
import React from 'react';
import { Link, useMatch } from 'react-location';
import type { LocationGenerics } from '../router';

export const BeerList = () => {
  const {
    data: { beers },
  } = useMatch<LocationGenerics>();

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
        </div>
      </div>
      <div className="flex flex-wrap mt-12 justify-center">
        <div className="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
          {beers.map((beer) => (
            <React.Fragment key={beer.id}>
              <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                <img
                  alt={beer.name}
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
                <Link to={`beer/${beer.id}`}>Detail</Link>
              </div>
            </React.Fragment>
          ))}
        </div>
      </div>
    </div>
  );
};

export default BeerList;
```

```typescript
// src/pages/BeerDetail.jsx
import React from 'react';
import { useNavigate, useMatch } from 'react-location';
import type { LocationGenerics } from '../router';

export const BeerDetail = () => {
  const navigate = useNavigate();
  const {
    data: { beer },
  } = useMatch<LocationGenerics>();

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
                onClick={() => navigate({ to: '/', replace: true })}
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

export default BeerDetail;
```

## Modules

The last thing is to discuss are modules. We can extract our routes parameters into separated files/modules and import them as options easily. This would potentially simplify the maintenance. Let's see that in action.

First of all, let's create two new files `beerListModule.tsx` and `beerDetailModule.tsx` in the `src` and add the following content:

```typescript
// src/beerListModule.tsx
import React from 'react';
import { getBeerList } from './api';

export default {
  loader: async () => {
    return {
      beers: await getBeerList(),
    };
  },
  element: () =>
    import('./pages/BeerList').then((module) => <module.default />),
};
```

```typescript
// src/beerDetailModule.tsx
import React from 'react';
import { getBeerById } from './api';

export default {
  loader: async ({ params }) => {
    return {
      beer: await getBeerById(parseInt(params.id)),
    };
  },
  element: () =>
    import('./pages/BeerDetail').then((module) => <module.BeerDetail />),
};
```

Once we have these files create, another step is to to adjust these changes in the `router.tsx` file.

```diff
--- a/router.tsx
+++ b/router.tsx
// src/router.tsx
-import React from 'react';
import { Route, ReactLocation, MakeGenerics } from 'react-location';
-import { getBeerById, getBeerList } from './api';
import type { BeerData } from './api';

export const routes: Route[] = [
  {
    path: '/',
-   loader: async () => {
-     return {
-       beers: await getBeerList(),
-     };
-   },
-   element: () =>
-     import('./pages/BeerList').then((module) => <module.default />),
+   import: () => import('./beerListModule').then((module) => module.default),
  },
  {
    path: 'beer',
    children: [
      {
        path: ':id',
-       loader: async ({ params }) => {
-         return {
-           beer: await getBeerById(parseInt(params.id)),
-         };
-       },
-       element: () =>
-         import('./pages/BeerDetail').then((module) => <module.BeerDetail />),
+       import: () => import('./beerDetailModule').then((module) => module.default),
      },
    ],
  },
];

export type LocationGenerics = MakeGenerics<{
  LoaderData: {
    beers: BeerData[];
    beer: BeerData;
  };
}>;

export const location = new ReactLocation();
```

The latest snapshot of the `router.tsx` should look as the following:

```typescript
// src/router.tsx
import { Route, ReactLocation, MakeGenerics } from 'react-location';
import type { BeerData } from './api';

export const routes: Route[] = [
  {
    path: '/',
    import: () => import('./beerListModule').then((module) => module.default),
  },
  {
    path: 'beer',
    children: [
      {
        path: ':id',
        import: () =>
          import('./beerDetailModule').then((module) => module.default),
      },
    ],
  },
];

export type LocationGenerics = MakeGenerics<{
  LoaderData: {
    beers: BeerData[];
    beer: BeerData;
  };
}>;

export const location = new ReactLocation();
```

## Conclusion

Thanks for following this workshop material. Any feedback appreciated (radek.tomasek@gmail.com).
