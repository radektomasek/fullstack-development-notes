# Introduction to Immer 

In this session we are going to implement a simple React app which fetches data from a weather service and displays it on a screen.

The code is written in [Typescript](https://www.typescriptlang.org/) with usage of [Immer.js](https://immerjs.github.io/immer/docs/introduction) as a state management solution. [Bulma](https://bulma.io/) is used for simple styling.

Inspired by Michel Weststrate's amazing egghead.io tutorial.

## Prerequisites

To be able to practically follow this workshop, you need to make sure the following criteria are met:

- Register an account at [https://openweathermap.org/api](https://openweathermap.org/api) and **obtain an API key**.
- Clone this repository `git clone git@github.com:ovotech/cop_frontend.git` / get the latest version `git pull --rebase`
- Checkout the initial commit of this project: `git checkout 974e570a900fb95d0083518f57c4b8b5d75e814e`
- In the root directory of this project `e.g. ./cop_frontend/immer_intro_workshop/`, create a `.env` file.
- Append `API_KEY='<API_KEY>'` string (api key from the weather service) into this file.
- Install all packages by running `npm install`
- Run `npm run start` to start the project
- Run `npm run test:watch` in second terminal to start tests in watch mode.

## Getting Started

Once the environment is setup completely, it's time to start coding.

### 1. Basic structure with initial set of unit tests and state not currently handled by Immer.js

```typescript
// state.ts

import { format } from 'date-fns';
import { SearchHistory } from '../types';

interface SearchHistory {
  cityName: string;
  temperature: string;
  initialTimestamp: string;
  updateTimestamp: string;
  iconUrl: string;
}

export const generateTimestamp = (date: Date = new Date()) =>
  format(date, 'yyyy-MM-dd HH:mm:ss.SSS');

export const findHistoryElementByCityId = (
  state: SearchHistory[],
  key: string
): SearchHistory | undefined => state.find(element => element.cityName === key);

export const addNewElement = (
  state: SearchHistory[],
  key: string,
  temperature: string,
  timestamp: string,
  iconUrl: string
) => [
  ...state,
  {
    iconUrl,
    temperature,
    cityName: key,
    initialTimestamp: timestamp,
    updateTimestamp: timestamp
  }
];

export const updateExistingElement = (
  state: SearchHistory[],
  key: string,
  temperature: string,
  timestamp: string,
  iconUrl: string
) =>
  state.reduce((accumulator, element) => {
    const updatedElement =
      element.cityName === key
        ? Object.assign({}, element, {
            iconUrl,
            temperature,
            updateTimestamp: timestamp
          })
        : Object.assign({}, element);

    return [...accumulator, updatedElement];
  }, [] as SearchHistory[]);

export const removeExistingElement = (state: SearchHistory[], key: string) =>
  state.filter(element => element.cityName !== key);

export const addOrUpdateDataInSearchHistory = (
  state: SearchHistory[],
  key: string,
  temperature: string,
  iconUrl: string
): SearchHistory[] => {
  const timestamp = generateTimestamp();
  const hasElementBeenAlreadyFetched = !!findHistoryElementByCityId(state, key);

  if (hasElementBeenAlreadyFetched) {
    return updateExistingElement(state, key, temperature, timestamp, iconUrl);
  }

  return addNewElement(state, key, temperature, timestamp, iconUrl);
};
```

```typescript
// state.test.ts

import { format } from 'date-fns';
import {
  generateTimestamp,
  findHistoryElementByCityId,
  addNewElement,
  updateExistingElement,
  addOrUpdateDataInSearchHistory,
  removeExistingElement
} from './state';
import { SearchHistory } from '../types';

const timestamp = generateTimestamp();
const initialState: SearchHistory[] = [
  {
    cityName: 'london',
    temperature: 'Sunny (15°C)',
    initialTimestamp: timestamp,
    updateTimestamp: timestamp,
    iconUrl: 'http://openweathermap.org/img/w/03d.png'
  },
  {
    cityName: 'san francisco',
    temperature: 'Rain (24°C)',
    initialTimestamp: timestamp,
    updateTimestamp: timestamp,
    iconUrl: 'http://openweathermap.org/img/w/04d.png'
  }
];

describe('#generateTimestamp', () => {
  test('generates a date in dateTime format', () => {
    const testDate = new Date();
    const year = format(testDate, 'yyyy');
    const month = format(testDate, 'MM');
    const day = format(testDate, 'dd');

    const hour = format(testDate, 'HH');
    const minute = format(testDate, 'mm');
    const second = format(testDate, 'ss');

    const miliseconds = format(testDate, 'SSS');

    expect(generateTimestamp(testDate)).toEqual(
      `${year}-${month}-${day} ${hour}:${minute}:${second}.${miliseconds}`
    );
  });
});

describe('#findHistoryElementByCityId', () => {
  test('returns an element from history array when key matches', () => {
    expect(findHistoryElementByCityId(initialState, 'london')).toBe(
      initialState[0]
    );
    expect(findHistoryElementByCityId(initialState, 'san francisco')).toBe(
      initialState[1]
    );
  });

  test(`returns an undefined if a key doesn't match what is in history array`, () => {
    expect(findHistoryElementByCityId(initialState, 'prague')).toEqual(
      undefined
    );
    expect(findHistoryElementByCityId(initialState, 'los angeles')).toEqual(
      undefined
    );
  });
});

describe('#addNewElement', () => {
  const nextState = addNewElement(
    initialState,
    'paris',
    'Rain (10°C)',
    timestamp,
    'http://openweathermap.org/img/w/05d.png'
  );

  test('add a new element at the end of the collection', () => {
    expect(nextState.length).toEqual(3);
    expect(nextState[2].cityName).toEqual('paris');
  });

  test(`didn't modify the original state`, () => {
    expect(initialState.length).toEqual(2);
  });
});

describe('#updateExistingElement', () => {
  const updatedTimestamp = generateTimestamp();
  const updatedTemperature = 'Wind (17°C)';
  const updatedImageUrl = 'http://openweathermap.org/img/w/05d.png';

  test('update temperature and timestamp if element (by key) exists', () => {
    const nextState = updateExistingElement(
      initialState,
      'london',
      updatedTemperature,
      updatedTimestamp,
      updatedImageUrl
    );

    const expectedObject: SearchHistory = {
      cityName: 'london',
      temperature: updatedTemperature,
      initialTimestamp: initialState[0].initialTimestamp,
      updateTimestamp: updatedTimestamp,
      iconUrl: updatedImageUrl
    };

    expect(nextState[0]).toEqual(expectedObject);
    expect(nextState[1]).toEqual(initialState[1]);
  });

  test(`keep elements without any touch if element (by key) doesn't exist`, () => {
    const nextState = updateExistingElement(
      initialState,
      'los angeles',
      updatedTemperature,
      updatedTimestamp,
      updatedImageUrl
    );

    expect(nextState[0]).toEqual(initialState[0]);
    expect(nextState[1]).toEqual(initialState[1]);
  });

  test(`didn't modify the original state at any case`, () => {
    expect(initialState[0].initialTimestamp).toEqual(
      initialState[0].updateTimestamp
    );
    expect(initialState[1].initialTimestamp).toEqual(
      initialState[1].updateTimestamp
    );
  });
});

describe('#removeExistingElement', () => {
  const nextState = removeExistingElement(initialState, 'london');

  test('remove the element from an array', () => {
    expect(nextState.length).toBe(1);
    expect(nextState[0].cityName).toBe(initialState[1].cityName);
  });

  test(`didn't modify the original state at any case`, () => {
    expect(initialState.length).toBe(2);
  });
});

describe('#addOrUpdateDataInSearchHistory', () => {
  const updatedTemperature = 'Wind (17°C)';
  const updatedImageUrl = 'http://openweathermap.org/img/w/05d.png';
  describe(`when key doesn't exist`, () => {
    test('add new element into the array without modifying the intial state', () => {
      const nextState = addOrUpdateDataInSearchHistory(
        initialState,
        'los angeles',
        updatedTemperature,
        updatedImageUrl
      );

      expect(initialState.length).toEqual(2);
      expect(nextState.length).toEqual(3);
    });
  });

  describe('when key does exist', () => {
    test('update element in the array by key without modifying the initial state', () => {
      const nextState = addOrUpdateDataInSearchHistory(
        initialState,
        'london',
        updatedTemperature,
        updatedImageUrl
      );

      expect(initialState.length).toEqual(2);
      expect(nextState.length).toEqual(2);
      expect(nextState[0].initialTimestamp).not.toEqual(
        nextState[0].updateTimestamp
      );
      expect(nextState[1].initialTimestamp).toEqual(
        nextState[1].updateTimestamp
      );
    });
  });
});
```

### 2. Basic concepts of Immer.js

```typescript
// introduction.ts

import produce, { Draft } from 'immer';

export interface Introduction {
  name: string;
  timestamp: Date | null;
}

export const addRecord = (
  state: Draft<Readonly<Introduction[]>>,
  name: string
) =>
  produce(state, draft => {
    draft.push({ name, timestamp: null });
  });
```

```typescript
// introduction.test.ts

import { setAutoFreeze } from 'immer';
import { Introduction, addRecord } from './introduction';

export const initialState: Introduction[] = [
  {
    name: 'Tim',
    timestamp: new Date()
  },
  {
    name: 'Kevin',
    timestamp: new Date()
  }
];

describe('#addRecord', () => {
  // setAutoFreeze(false);
  const newState = addRecord(initialState, 'Stephen');
  test('add new record without modifiyng the existing state', () => {
    expect(initialState.length).toEqual(2);
    expect(newState.length).toEqual(3);
  });

  test(`doesn't modify the untouched elements (keep the original references - structural changes)`, () => {
    expect(initialState[0]).toBe(newState[0]);
    expect(initialState[1]).toBe(newState[1]);
  });

  test('throw an error for newly produced (freezed) object', () => {
    expect(() => {
      newState[2].timestamp = new Date();
    }).toThrow();
  });
});
```

### 3. Changing core functions from immutable to mutable ones and start using produce function in state.ts

Let's first 'refactor' core functions to make them mutate the initial state. **This makes the test fail**.

```typescript
// state.ts  (selected fragments)

import produce from 'immer';

export const addNewElement = (
  state: SearchHistory[],
  key: string,
  temperature: string,
  timestamp: string,
  iconUrl: string
) => {
  state.push({
    iconUrl,
    temperature,
    cityName: key,
    initialTimestamp: timestamp,
    updateTimestamp: timestamp
  });

  return state;
};

export const updateExistingElement = (
  state: SearchHistory[],
  key: string,
  temperature: string,
  timestamp: string,
  iconUrl: string
) => {
  return state.map(element => {
    if (element.cityName !== key) {
      return element;
    }

    element.iconUrl = iconUrl;
    element.updateTimestamp = timestamp;
    element.temperature = temperature;

    return element;
  });
};

export const removeExistingElement = (state: SearchHistory[], key: string) => {
  const indexOf = state.findIndex(element => element.cityName === key);
  state.splice(indexOf, 1);
};
```

After initial 'refactoring' is completed, let's apply the `produce` function **to make results immutable**.

```typescript
// state.ts  (selected fragments)

export const addNewElement = (
  state: SearchHistory[],
  key: string,
  temperature: string,
  timestamp: string,
  iconUrl: string
) =>
  produce(state, draft => {
    draft.push({
      iconUrl,
      temperature,
      cityName: key,
      initialTimestamp: timestamp,
      updateTimestamp: timestamp
    });
  });

export const updateExistingElement = (
  state: SearchHistory[],
  key: string,
  temperature: string,
  timestamp: string,
  iconUrl: string
) => {
  return produce(state, draft => {
    draft.map(element => {
      if (element.cityName !== key) {
        return element;
      }

      element.iconUrl = iconUrl;
      element.updateTimestamp = timestamp;
      element.temperature = temperature;

      return element;
    });
  });
};

export const removeExistingElement = (state: SearchHistory[], key: string) => {
  return produce(state, draft => {
    const indexOf = draft.findIndex(element => element.cityName === key);
    draft.splice(indexOf, 1);
  });
};
```

### 4. Applying some currying

```typescript
// state.ts  (selected fragments)

// produce(state, recipe) => nextState
// produce(recipe) => (state) => nextState

export const addNewElement = produce(
  (
    draft: SearchHistory[],
    key: string,
    temperature: string,
    timestamp: string,
    iconUrl: string
  ) => {
    draft.push({
      iconUrl,
      temperature,
      cityName: key,
      initialTimestamp: timestamp,
      updateTimestamp: timestamp
    });
  }
);

export const updateExistingElement = produce(
  (
    draft: SearchHistory[],
    key: string,
    temperature: string,
    timestamp: string,
    iconUrl: string
  ) => {
    draft.map(element => {
      if (element.cityName !== key) {
        return element;
      }

      element.iconUrl = iconUrl;
      element.updateTimestamp = timestamp;
      element.temperature = temperature;

      return element;
    });
  }
);

export const removeExistingElement = produce(
  (draft: SearchHistory[], key: string) => {
    const indexOf = draft.findIndex(element => element.cityName === key);
    draft.splice(indexOf, 1);
  }
);
```

### 5. Implementation of returning a new state + using void keyword.

```typescript
// introduction.ts  (selected fragments)

export const addRecord = (
  state: Draft<Readonly<Introduction[]>>,
  name: string
) => produce(state, draft => void draft.push({ name, timestamp: null }));

export const clearRecords = (state: Introduction[]) =>
  produce(state, draft => []);
```

```typescript
// introduction.test.ts  (selected fragments)

describe('#clearRecords', () => {
  const newState = clearRecords(initialState);
  test(`returns an empty array and doesn't modify the original state`, () => {
    expect(newState).toEqual([]);
    expect(initialState.length).toEqual(2);
  });
});
```

### 6. Added a small component using use-immer (https://github.com/immerjs/use-immer)

Let's create a basic component

```typescript
// AppWithUseImmer.tsx

import * as React from 'react';
import produce from 'immer';
import { useImmer } from 'use-immer';

interface CityList {
  cityName: string;
  timestamp: Date;
}

export const AppWithUseImmer = () => {
  const [value, setValue] = React.useState<string>('');
  const [cityObjects, addCityObject] = React.useState<CityList[]>([]);

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    setValue(event.target.value);
  };

  const handleSubmit = (name: string) => {
    addCityObject(state =>
      produce(state, draft => {
        draft.push({ cityName: name, timestamp: new Date() });
      })
    );
  };

  return (
    <div>
      <input type="text" value={value} onChange={handleChange} />
      <button onClick={() => handleSubmit(value)}>Add</button>
      <ul>
        {cityObjects.map(cityObject => (
          <li key={cityObject.timestamp.toString()}>{cityObject.cityName}</li>
        ))}
      </ul>
    </div>
  );
};
```

```typescript
// index.tsx

import * as React from 'react';
import * as ReactDOM from 'react-dom';
import { App } from './components/App';
import { AppWithUseImmer } from './components/AppWithUseImmer';

ReactDOM.render(
  // <App apiKey={process.env.API_KEY || ''} />,
  <AppWithUseImmer />,
  document.getElementById('root')
);
```

We can simplify `handleSubmit` method by applying currying.

```typescript
// AppWithUseImmer.tsx  (selected fragments)

const handleSubmit = (name: string) => {
  addCityObject(
    produce(draft => {
      draft.push({ cityName: name, timestamp: new Date() });
    })
  );
};
```

If we use `useImmer` hook, we can also get rid of the `produce` function.

```typescript
// AppWithUseImmer.tsx (selected fragments)

const [cityObjects, addCityObject] = useImmer<CityList[]>(() => []);

const handleSubmit = (name: string) => {
  addCityObject(draft => {
    draft.push({ cityName: name, timestamp: new Date() });
  });
};
```

### 7. Immer.js with useReducer hook

```typescript
// index.tsx

import * as React from 'react';
import * as ReactDOM from 'react-dom';
import { App } from './components/App';
import { AppWithUseImmer } from './components/AppWithUseImmer';

ReactDOM.render(
  <App apiKey={process.env.API_KEY || ''} />,
  // <AppWithUseImmer />,
  document.getElementById('root')
);
```

```typescript
// state.ts (selected fragments)

export const weatherHistoryReducer = produce(
  (draft: SearchHistory[], action) => {
    const timestamp = generateTimestamp();
    switch (action.type) {
      case 'ADD_ELEMENT':
        draft.push({
          iconUrl: action.iconUrl,
          temperature: action.temperature,
          cityName: action.key,
          initialTimestamp: timestamp,
          updateTimestamp: timestamp
        });
        break;
      case 'UPDATE_ELEMENT':
        draft.map(element => {
          if (element.cityName !== action.key) {
            return element;
          }

          element.iconUrl = action.iconUrl;
          element.updateTimestamp = timestamp;
          element.temperature = action.temperature;

          return element;
        });
        break;
      case 'REMOVE_ELEMENT':
        const indexOf = draft.findIndex(
          element => element.cityName === action.key
        );
        draft.splice(indexOf, 1);
        break;
    }
  }
);
```

```typescript
// App.tsx (selected fragments)

import {
  weatherHistoryReducer,
  findHistoryElementByCityId
} from '../utils/state';

const [searchHistory, dispatch] = React.useReducer(weatherHistoryReducer, []);

const handleRemove = (key: string): void => {
  dispatch({ type: 'REMOVE_ELEMENT', key });
};

const handleSubmit = async (value: string) => {
  try {
    const { data } = await fetchWeatherDataByCityName(value, apiKey);
    if (!findHistoryElementByCityId(searchHistory, value)) {
      dispatch({
        type: 'ADD_ELEMENT',
        iconUrl: generateOpenWeatherIconUrl(data),
        temperature: generateTemperatureDescription(data),
        key: data.name
      });
    } else {
      dispatch({
        type: 'UPDATE_ELEMENT',
        iconUrl: generateOpenWeatherIconUrl(data),
        temperature: generateTemperatureDescription(data),
        key: data.name
      });
    }
  } catch (error) {
    console.log(error);
  }
};
```

### 8. Additional content

You can use some helper functions from immer like `original` or `isDraft` which returns original object within proxy lifecycle or check (within the same lifecycle) whether the object is still a draft (being modified by immer produce, but not finalized) respectively.

```typescript
// AppWithUseImmer.ts (selected fragments)
const handleSubmit = (name: string) => {
  const result = addCityObject(draft => {
    draft.push({ cityName: name, timestamp: new Date() });
    console.log(original(draft));
    console.log('isDraft withint addCityObject call', isDraft(draft));
  });
  console.log('isDraft outside addCityObject call', isDraft(result));
};
```

You can also use `createDraft` and `finishDraft` as a low-level approach to manage `produce` functionality.

```typescript
// introduction.ts
export const addRecordWithCreateDraft = (
  state: Introduction[],
  name: string
) => {
  const draft = createDraft(state);
  draft.push({ name, timestamp: null });
  return finishDraft(draft);
};
```

Patches are another powerful part of immer library which helps to manage the state updates in small increments/batches (and potentially re-creates the state).

```typescript
// introduction.ts
export const addRecoedWithProduceWithPatches = (
  state: Introduction[],
  name: string
) =>
  produceWithPatches(state, draft => {
    draft.push({ name, timestamp: null });
  });
```

```typescript
// introduction.test.ts
describe('#addRecordWithProduceWithPatches', () => {
  const [nextState, patches, inversePatches] = addRecoedWithProduceWithPatches(
    initialState,
    'Tim'
  );
  expect(initialState.length).toEqual(2);
  expect(nextState.length).toEqual(3);
  expect(patches).toEqual([
    { op: 'add', path: [2], value: { name: 'Tim', timestamp: null } }
  ]);
  expect(inversePatches).toEqual([{ op: 'remove', path: [2] }]);
});
```

## Other topics for discussion

- leverage structural sharing for rendering optimization (`memo` + `useCallback`).

## Conclusion

Thanks for following this workshop material. Any feedback appreciated (radek.tomasek@gmail.com)
