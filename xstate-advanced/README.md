# Advanced usage of XState with React and Typescript

State management is a complex topic and can be handled in different ways. In **React world**, probably the most common way is a usage of **Redux** library or utilizing **Hooks APIs**. However, there are a lot of challenging aspects whilst choosing either approach, especially when it comes to handle some complex state transitions like async states or dynamic forms.

[XState](https://xstate.js.org/) is a powerful library that helps to manage state and its transitions more consistently and with predictable results.

This workshop is a simple demonstration of the capabilities of how to write a simple React Application with a usage of Typescript and XState. The app is based on the one used during last workshop dedicated to Concurrent React.

There are also a few small asides inspired by examples from a great course on egghead.io - [Introduction to State Machines Using XState](https://egghead.io/courses/introduction-to-state-machines-using-xstate) by [Kyle Shevlin](https://egghead.io/instructors/kyle-shevlin) which I initially used as the main learning material. These examples are written in Javascript, just to be able to easily transfer them into [XState visualiser](https://xstate.js.org/viz/).

## Prerequisites

- Good working knowledge of React and Typescript

## Punk API

The REST API we are going to use is called PUNK API (v2). It is a fan project related to the products of Brewdog Brewery. The full documentation is available [here](https://punkapi.com/documentation/v2).

During the workshop, we are going to use the following endpoints:

```
GET /beers?beer_name=name - list of beers that match the name parameter
```

## Available commands

- `npm run start` - run the project.

## Getting Started

### 1. Basic structure with initial set of React components and data fetching stuff

There is an `api.ts` file with logic for data fetching

```typescript
// src/api.ts
import axios from 'axios';

const BASE_URL = 'https://api.punkapi.com/v2';

const instance = axios.create({
  baseURL: BASE_URL,
});

export interface BeerData {
  id: number;
  name: string;
  tagline: string;
  description: string;
  image_url: string;
}

export const getListOfBeersByName = (beerName: string = '') =>
  instance.get<BeerData[]>(
    `${BASE_URL}/beers?beer_name=${beerName.replace(' ', '_')}`
  );
```

`SearchBeer` component contains simple form which helps to enter a beer string and handle the final submit

```typescript
// SearchBeer.tsx
import * as React from 'react';

interface Props {
  onSubmit: (suggestedName: string) => void;
}

export const SearchBeer = ({ onSubmit }: Props) => {
  const [value, setValue] = React.useState('');

  return (
    <section className="mt-10 mb-5 w-3/5">
      <div className="flex flex-row">
        <input
          type="text"
          placeholder="Find beers by name"
          value={value}
          onChange={(event) => setValue(event.target.value)}
          className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        />
        <button
          type="button"
          className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
          onClick={() => onSubmit(value)}
        >
          Search
        </button>
      </div>
    </section>
  );
};
```

`BeerList` is a simple component that shows result based based on the data array which is returned from the API.

```typescript
// BeerList.tsx

import * as React from 'react';
import { BeerData } from './api';

interface Props {
  beers: BeerData[];
  errorMessage: string;
  isLoading: boolean;
}

export const BeerList = ({ beers, errorMessage, isLoading }: Props) => {
  if (isLoading) {
    return <p>Loading...</p>;
  }

  if (errorMessage) {
    return <p>There was an error: {errorMessage}</p>;
  }

  return (
    <>
      <ul>
        {beers.map((beer) => (
          <li
            key={beer.id}
            className="rounded overflow-hidden shadow-lg w-3/5 p-3"
          >
            <div className="mt-4 mb-6">
              <h3 className="text-lg font-semibold mb-1">{beer.name}</h3>
              <div className="uppercase tracking-wide text-sm text-gray-700 font-bold">
                {beer.tagline}
              </div>

              <p className="mt-2 text-gray-600">{beer.description}</p>
            </div>
          </li>
        ))}
      </ul>
    </>
  );
};
```

`App.tsx` is a component that glues things together. In this initial version, we rely on the simple React only state management.

```typescript
// App.tsx
import * as React from 'react';
import { getListOfBeersByName, BeerData } from './api';
import { SearchBeer } from './SearchBeer';
import { BeerList } from './BeerList';

export const App = () => {
  const [beerName, setBeerName] = React.useState('');

  const [isLoading, setIsLoading] = React.useState(false);
  const [beers, setBeers] = React.useState<BeerData[]>([]);
  const [errorMessage, setErrorMessage] = React.useState('');

  const handleSubmit = (value: string) => {
    setBeerName(value);
    setIsLoading(true);
    getListOfBeersByName(value)
      .then((response) => {
        setBeers(response.data);
        setIsLoading(false);
      })
      .catch((error) => {
        setBeers([]);
        setErrorMessage(error.data);
        setIsLoading(false);
      });
  };

  return (
    <div className="container mx-auto">
      <SearchBeer onSubmit={handleSubmit} />
      <BeerList
        beers={beers}
        isLoading={isLoading}
        errorMessage={errorMessage}
      />
    </div>
  );
};
```

### 2. Applying XState Mechanism for data fetching

We can replace some boilerplate by `creating a new XState machine`. Let's create a new folder called `machines` and put a new `fetch.ts` file inside. This will be our initial machine for data fetching.

```typescript
// machines/fetch.ts
import { Machine, assign } from 'xstate';
import { BeerData } from '../api';

interface FetchContext {
  beers: BeerData[];
  errorMessage: string;
}

interface FetchStates {
  states: {
    idle: {};
    pending: {};
    success: {};
    failure: {};
  };
}

type FetchEvent = { type: 'FETCH'; beerName: string };
type ResolveEvent = { type: 'RESOLVE'; beers: BeerData[] };
type RejectEvent = { type: 'REJECT'; errorMessage: string };

type FetchMachineEvents = FetchEvent | ResolveEvent | RejectEvent;

export const fetchMachine = Machine<
  FetchContext,
  FetchStates,
  FetchMachineEvents
>(
  {
    id: 'fetch',
    initial: 'idle',
    context: {
      beers: [],
      errorMessage: '',
    },
    states: {
      idle: {
        on: {
          FETCH: 'pending',
        },
      },
      pending: {
        entry: ['fetchData'],
        on: {
          RESOLVE: { target: 'success', actions: ['setBeers'] },
          REJECT: { target: 'failure', actions: ['setErrorMessage'] },
        },
      },
      success: {
        on: {
          FETCH: 'pending',
        },
      },
      failure: {
        on: {
          FETCH: 'pending',
        },
      },
    },
  },
  {
    actions: {
      setBeers: assign((context, event) => ({
        beers: event.type === 'RESOLVE' ? event.beers : [],
      })),
      setErrorMessage: assign((context, event) => ({
        errorMessage: event.type === 'REJECT' ? event.errorMessage : '',
      })),
    },
  }
);
```

After we create a new machine, we can easily integrate it with the React Component by using `useMachine` hook imported from `@xstate/react`.

```typescript
// App.tsx
import * as React from 'react';
import { getListOfBeersByName, BeerData } from './api';
import { SearchBeer } from './SearchBeer';
import { BeerList } from './BeerList';
import { fetchMachine } from './machines/fetch';
import { useMachine } from '@xstate/react';

export const App = () => {
  const [beerName, setBeerName] = React.useState('');
  const [fetchState, sendToFetchMachine] = useMachine(fetchMachine, {
    actions: {
      fetchData: (context, event) => {
        const beerName = event.type === 'FETCH' ? event.beerName : '';
        getListOfBeersByName(beerName)
          .then((response) => {
            sendToFetchMachine({ type: 'RESOLVE', beers: response.data });
          })
          .catch((error) => {
            sendToFetchMachine({
              type: 'REJECT',
              errorMessage: 'A problem with fetching data',
            });
          });
      },
    },
  });

  const handleSubmit = (value: string) => {
    setBeerName(value);
    sendToFetchMachine({ type: 'FETCH', beerName: value });
  };

  return (
    <div className="container mx-auto">
      <SearchBeer onSubmit={handleSubmit} />
      <BeerList
        beers={fetchState.context.beers}
        isLoading={fetchState.matches('pending')}
        errorMessage={fetchState.context.errorMessage}
      />
    </div>
  );
};
```

### 3. Aside #1 - Using XState's invoke mechanism

The previous example used data fetching in a really simple way. That led into adding more boilerplate into our React component. `XState has some mechanism how to simplify the logic and reduce that boilerplate`. However, before we update the code in our React App, let's discuss the invoke functionality separately.

Let's create a new folder `examples` and new file called `invokePromise.js`. The first iteration might look like the following.

```javascript
// examples/invokePromise.js
const { Machine, interpret, assign } = require('xstate');

const extractMetadata = (responseData = []) =>
  responseData.map(({ id, name, tagline, description }) => ({
    id,
    name,
    tagline,
    description,
  }));

const fetchBeers = () => {
  return fetch(`https://api.punkapi.com/v2/beers`)
    .then((response) => response.json())
    .then(extractMetadata);
};

const beerInvokeMachine = Machine({
  id: 'beerInvokeMachine',
  initial: 'idle',
  context: {
    beers: [],
    error: null,
  },
  states: {
    idle: {},
    pending: {},
    success: {},
    failure: {},
  },
});
```

We can extend a few state objects easily, by adding a transition from `idle` state to `pending` and set `success` state to `final` and put a `retry` transition to `failure` state. The code might look like this:

```javascript
// examples/invokePromise.js
const { Machine, interpret, assign } = require('xstate');

const extractMetadata = (responseData = []) =>
  responseData.map(({ id, name, tagline, description }) => ({
    id,
    name,
    tagline,
    description,
  }));

const fetchBeers = () => {
  return fetch(`https://api.punkapi.com/v2/beers`)
    .then((response) => response.json())
    .then(extractMetadata);
};

const beerInvokeMachine = Machine({
  id: 'beerInvokeMachine',
  initial: 'idle',
  context: {
    beers: [],
    error: null,
  },
  states: {
    idle: {
      on: { FETCH: 'pending' },
    },
    pending: {},
    success: {
      type: 'final',
    },
    failure: {
      on: { RETRY: 'pending' },
    },
  },
});
```

Another step would be adding `invoke` mechanism to the `pending` state.

```javascript
// examples/invokePromise.js
const { Machine, interpret, assign } = require('xstate');

const extractMetadata = (responseData = []) =>
  responseData.map(({ id, name, tagline, description }) => ({
    id,
    name,
    tagline,
    description,
  }));

const fetchBeers = () => {
  return fetch(`https://api.punkapi.com/v2/beers`)
    .then((response) => response.json())
    .then(extractMetadata);
};

const beerInvokeMachine = Machine({
  id: 'beerInvokeMachine',
  initial: 'idle',
  context: {
    beers: [],
    error: null,
  },
  states: {
    idle: {
      on: { FETCH: 'pending' },
    },
    pending: {
      invoke: {
        id: '',
        src: '',
        onDone: {},
        onError: {},
      },
    },
    success: {
      type: 'final',
    },
    failure: {
      on: { RETRY: 'pending' },
    },
  },
});
```

We can complete the implementation by providing the functionality for the invoke object.

```javascript
// examples/invokePromise.js
const { Machine, interpret, assign } = require('xstate');

const extractMetadata = (responseData = []) =>
  responseData.map(({ id, name, tagline, description }) => ({
    id,
    name,
    tagline,
    description,
  }));

const fetchBeers = () => {
  return fetch(`https://api.punkapi.com/v2/beers`)
    .then((response) => response.json())
    .then(extractMetadata);
};

const beerInvokeMachine = Machine({
  id: 'beerInvokeMachine',
  initial: 'idle',
  context: {
    beers: [],
    error: null,
  },
  states: {
    idle: {
      on: { FETCH: 'pending' },
    },
    pending: {
      invoke: {
        id: 'fetchBeers',
        src: fetchBeers,
        onDone: {
          target: 'success',
          actions: assign({
            beers: (context, event) => event.data,
          }),
        },
        onError: {
          target: 'failure',
          actions: assign({
            error: (context, event) => 'a problem with data fetching',
          }),
        },
      },
    },
    success: {
      type: 'final',
    },
    failure: {
      on: { RETRY: 'pending' },
    },
  },
});
```

### 4. Applying XState's invoke mechanism in the React App

We can apply the previous exercise into our React App very easily. Let's use the `invoke` mechanism in our `fetchMachine`.

```typescript
// machines/fetch.ts
import { Machine, assign } from 'xstate';
import { BeerData } from '../api';

interface FetchContext {
  beers: BeerData[];
  errorMessage: string;
}

interface FetchStates {
  states: {
    idle: {};
    pending: {};
    success: {};
    failure: {};
  };
}

type FetchEvent = { type: 'FETCH'; beerName: string };

type FetchMachineEvents = FetchEvent;

export const fetchMachine = Machine<
  FetchContext,
  FetchStates,
  FetchMachineEvents
>(
  {
    id: 'fetch',
    initial: 'idle',
    context: {
      beers: [],
      errorMessage: '',
    },
    states: {
      idle: {
        on: {
          FETCH: 'pending',
        },
      },
      pending: {
        invoke: {
          id: 'fetchData',
          src: 'fetchData',
          onDone: { target: 'success', actions: ['setBeers'] },
          onError: { target: 'failure', actions: ['setErrorMessage'] },
        },
      },
      success: {
        on: {
          FETCH: 'pending',
        },
      },
      failure: {
        on: {
          FETCH: 'pending',
        },
      },
    },
  },
  {
    actions: {
      setBeers: assign((context, event: any) => ({
        beers: event.data,
      })),
      setErrorMessage: assign((context, event) => ({
        errorMessage: 'A problem with fetching data',
      })),
    },
  }
);
```

After the machine is updated, we need to change our React implementation a little bit. The main difference is that we replace `actions` by `services`.

```typescript
// App.tsx
import * as React from 'react';
import { getListOfBeersByName, BeerData } from './api';
import { SearchBeer } from './SearchBeer';
import { BeerList } from './BeerList';
import { fetchMachine } from './machines/fetch';
import { useMachine } from '@xstate/react';

export const App = () => {
  const [beerName, setBeerName] = React.useState('');
  const [fetchState, sendToFetchMachine] = useMachine(fetchMachine, {
    services: {
      fetchData: (context, event) =>
        getListOfBeersByName(event.beerName).then((response) => response.data),
    },
  });

  const handleSubmit = (value: string) => {
    setBeerName(value);
    sendToFetchMachine({ type: 'FETCH', beerName: value });
  };

  return (
    <div className="container mx-auto">
      <SearchBeer onSubmit={handleSubmit} />
      <BeerList
        beers={fetchState.context.beers}
        isLoading={fetchState.matches('pending')}
        errorMessage={fetchState.context.errorMessage}
      />
    </div>
  );
};
```

### 5. Adding a guard to check whether there are some data in the results or not

We can extend `success` state in `fetch machine` by adding a guard. This helps to set condition and based on the result, make a decision in which state it should transit into. Let's modify the machine.

```typescript
// machines/fetch.ts

import { Machine, assign } from 'xstate';
import { BeerData } from '../api';

interface FetchContext {
  beers: BeerData[];
  errorMessage: string;
}

interface FetchStates {
  states: {
    idle: {};
    pending: {};
    success: {
      states: {
        check: {};
        withData: {};
        withoutData: {};
      };
    };
    failure: {};
  };
}

type FetchEvent = { type: 'FETCH'; beerName: string };

type FetchMachineEvents = FetchEvent;

export const fetchMachine = Machine<
  FetchContext,
  FetchStates,
  FetchMachineEvents
>(
  {
    id: 'fetch',
    initial: 'idle',
    context: {
      beers: [],
      errorMessage: '',
    },
    states: {
      idle: {
        on: {
          FETCH: 'pending',
        },
      },
      pending: {
        invoke: {
          id: 'fetchData',
          src: 'fetchData',
          onDone: { target: 'success', actions: ['setBeers'] },
          onError: { target: 'failure', actions: ['setErrorMessage'] },
        },
      },
      success: {
        initial: 'check',
        on: {
          FETCH: 'pending',
        },
        states: {
          check: {
            on: {
              '': [
                { target: 'withData', cond: 'hasData' },
                { target: 'withoutData' },
              ],
            },
          },
          withData: {},
          withoutData: {},
        },
      },
      failure: {
        on: {
          FETCH: 'pending',
        },
      },
    },
  },
  {
    actions: {
      setBeers: assign((context, event: any) => ({
        beers: event.data,
      })),
      setErrorMessage: assign((context, event) => ({
        errorMessage: 'A problem with fetching data',
      })),
    },
    guards: {
      hasData: (context, event) => {
        return context.beers.length > 0;
      },
    },
  }
);
```

We also need to make some modifications in the `BeerList` component and remove some code.

```typescript
// BeerList.tsx
import * as React from 'react';
import { BeerData } from './api';

interface Props {
  beers: BeerData[];
}

export const BeerList = ({ beers }: Props) => (
  <ul>
    {beers.map((beer) => (
      <li key={beer.id} className="rounded overflow-hidden shadow-lg w-3/5 p-3">
        <div className="mt-4 mb-6">
          <h3 className="text-lg font-semibold mb-1">{beer.name}</h3>
          <div className="uppercase tracking-wide text-sm text-gray-700 font-bold">
            {beer.tagline}
          </div>

          <p className="mt-2 text-gray-600">{beer.description}</p>
        </div>
      </li>
    ))}
  </ul>
);
```

The final change will happen in the `App.tsx` where we utilize the condition.

```typescript
// App.tsx
import * as React from 'react';
import { getListOfBeersByName, BeerData } from './api';
import { SearchBeer } from './SearchBeer';
import { BeerList } from './BeerList';
import { fetchMachine } from './machines/fetch';
import { useMachine } from '@xstate/react';

export const App = () => {
  const [beerName, setBeerName] = React.useState('');
  const [fetchState, sendToFetchMachine] = useMachine(fetchMachine, {
    services: {
      fetchData: (context, event) =>
        getListOfBeersByName(event.beerName).then((response) => response.data),
    },
  });

  const handleSubmit = (value: string) => {
    setBeerName(value);
    sendToFetchMachine({ type: 'FETCH', beerName: value });
  };

  return (
    <div className="container mx-auto">
      <SearchBeer onSubmit={handleSubmit} />
      {fetchState.matches('pending') && <p>Loading...</p>}
      {fetchState.matches('success.withData') && (
        <BeerList beers={fetchState.context.beers} />
      )}
      {fetchState.matches('success.withoutData') && <p>No beer was found</p>}
      {fetchState.context.errorMessage && (
        <p>There was an error: {fetchState.context.errorMessage}</p>
      )}
    </div>
  );
};
```

### 6. Aside #2 - Parent and Child machines

Another interesting use-case for XState is having two machines - `parent` and `child` ones and make some interaction between them. Let's create a new file in `examples` folder called `parentChild.js`

```javascript
// examples/parentChild.js
const { Machine, interpret, send } = require('xstate');

const parentMachine = Machine({
  id: 'parentMachine',
  initial: 'idle',
  states: {
    idle: {
      on: {
        ACTIVATE: 'active',
      },
    },
    active: {},
    done: {},
  },
});
```

As a second step, let's create a child state machine.

```javascript
// examples/parentChild.js
const { Machine, interpret, send } = require('xstate');

const childMachine = Machine({
  id: 'childMachine',
  initial: 'step1',
  states: {
    step1: {},
    step2: {},
    step3: {},
  },
});

const parentMachine = Machine({
  id: 'parentMachine',
  initial: 'idle',
  states: {
    idle: {
      on: {
        ACTIVATE: 'active',
      },
    },
    active: {},
    done: {},
  },
});
```

The idea is to have a mechanism, which sends data from `parentMachine` to `childMachine` and invokes the step. And as you might guess, we can utilize the invoke mechanism again.

```javascript
// examples/parentChild.js
const { Machine, interpret, send } = require('xstate');

const childMachine = Machine({
  id: 'childMachine',
  initial: 'step1',
  states: {
    step1: {},
    step2: {},
    step3: {},
  },
});

const parentMachine = Machine({
  id: 'parentMachine',
  initial: 'idle',
  states: {
    idle: {
      on: {
        ACTIVATE: 'active',
      },
    },
    active: {
      invoke: {
        id: 'child',
        src: childMachine,
        onDone: 'done',
      },
    },
    done: {
      type: 'final',
    },
  },
});
```

However, we also need a mechanism how to active events from parent machine. We will create a `STEP` event in the `parentMachine` and send another `STEP` event to the child machine which will utilize it.

```javascript
// examples/parentChild.js
const { Machine, interpret, send } = require('xstate');

const childMachine = Machine({
  id: 'childMachine',
  initial: 'step1',
  states: {
    step1: {
      on: {
        STEP: 'step2',
      },
    },
    step2: {
      on: {
        STEP: 'step3',
      },
    },
    step3: {
      type: 'final',
    },
  },
});

const parentMachine = Machine({
  id: 'parentMachine',
  initial: 'idle',
  states: {
    idle: {
      on: {
        ACTIVATE: 'active',
      },
    },
    active: {
      invoke: {
        id: 'child',
        src: childMachine,
        onDone: 'done',
      },
      on: {
        STEP: {
          actions: send('STEP', { to: 'child' }),
        },
      },
    },
    done: {
      type: 'final',
    },
  },
});
```

### 7. Aside #3 - Delayed Events and transitions

Creating a standard XState Machine with manual transition is quite straightforward. Let's create a new file in the examples folder called `` followed by an implementation of a simple traffic light state machine.

```javascript
// examples/delayedEvents.js
const trafficLightMachine = Machine({
  id: 'trafficLightMachine',
  initial: 'red',
  states: {
    red: {
      on: {
        TIMER: 'yellow',
      },
    },
    yellow: {
      on: {
        TIMER: 'green',
      },
    },
    green: {
      on: {
        TIMER: 'red',
      },
    },
  },
});
```

We can also automated our transitions by replacing `on` fields by `after` and the `names of the events` by `numbers`.

```javascript
// examples/delayedEvents.js
const trafficLightMachine = Machine({
  id: 'trafficLightMachine',
  initial: 'red',
  states: {
    red: {
      after: {
        1000: 'yellow',
      },
    },
    yellow: {
      after: {
        1000: 'green',
      },
    },
    green: {
      after: {
        1000: 'red',
      },
    },
  },
});
```

You can also put these delay values into the option object and replaced the hardcoded values in states object by the names of these delay elements.

```javascript
// examples/delayedEvents.js
const trafficLightMachine = Machine(
  {
    id: 'trafficLightMachine',
    initial: 'red',
    states: {
      red: {
        after: {
          RED_TIMER: 'yellow',
        },
      },
      yellow: {
        after: {
          YELLOW_TIMER: 'green',
        },
      },
      green: {
        after: {
          GREEN_TIMER: 'red',
        },
      },
    },
  },
  {
    delays: {
      RED_TIMER: 1000,
      YELLOW_TIMER: 2000,
      GREEN_TIMER: 3000,
    },
  }
);
```

You can also parametrized these values and make them fully dynamic.

```javascript
// examples/delayedEvents.js
const trafficLightMachine = Machine(
  {
    id: 'trafficLightMachine',
    initial: 'red',
    context: {
      multiplier: 1,
    },
    on: {
      INCREASE_MULTIPLIER: {
        actions: 'increaseMultipler',
      },
    },
    states: {
      red: {
        after: {
          RED_TIMER: 'yellow',
        },
      },
      yellow: {
        after: {
          YELLOW_TIMER: 'green',
        },
      },
      green: {
        after: {
          GREEN_TIMER: 'red',
        },
      },
    },
  },
  {
    actions: {
      increaseMultipler: assign({
        multiplier: (context, event) => context.multiplier + 1,
      }),
    },
    delays: {
      RED_TIMER: (context) => context.multiplier * 1000,
      YELLOW_TIMER: (context) => context.multiplier * 2000,
      GREEN_TIMER: (context) => context.multiplier * 3000,
    },
  }
);
```

## Conclusion

Thanks for following this workshop material. Any feedback appreciated (radek.tomasek@gmail.com)
