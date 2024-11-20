# Intro to Zustand

This project shows a basic usage of [Zustand](https://github.com/pmndrs/zustand) - another state management that works well with (but not exclusive to) React. It tries to resolve some of the complexity of most of the state management (e.g. there is no provider) whilst providing a simple and easy to use interface.

## Project Initialization

We are going to initialize this project with `Create React App` and its `Tailwind/TypeScript Template`. I chose TypeScript as **Zustand has a very good TS support**.

For demonstration purposes, we are going to utilize our [Notes API written in Deno](https://github.com/radektomasek/frontend-community-workshops-archive/tree/main/deno-v1-intro) in one of our previous workshops.

> Note: To run the service, just type `deno run --allow-net mod.ts` in your terminal within the backend directory.

Let's type the following to initialize the project with its key dependencies:

```bash
# Initializing the project (a React App with Tailwind and TypeScript).
npx create-react-app zustand-intro --template tailwindcss-typescript
```

Switch the context to the project directory:

```bash
# Switch the context to the project directory
cd zustand-intro
```

As an additional step, we need to install **Zustand** package to be able to play with it. Let's do that now as well in the project directory.

```bash
# Installing external Zustand dependency.
npm install zustand --save
```

Let's create three new directories: `api`, `store` and `components` in the **src** folder by typing the following:

```bash
# Create three empty folders
mkdir src/{api,store,components}
```

We can also add a prettier config with a simple quote rule.

```bash
# Create a .prettierrc config
touch .prettierrc
```

```json
{
  "singleQuote": true
}
```

Before we move into the exploration of **Zustand** in more details, let's add some extra code now which would save us some time later.

### Adding HTTP functions & mocks

Let's add some functions that help us to call the actual API. Create `index.ts` file in the `src/api` folder containing the following code:

```typescript
// src/api/index.ts
const BASE_URL = 'http://localhost:8000';

export type NoteColor = '' | 'red' | 'green' | 'yellow';

export interface Note {
  id: string;
  title: string;
  description: string;
  color: NoteColor;
}

export const getNotes = (): Promise<Note[]> =>
  fetch(`${BASE_URL}/notes`, {
    method: 'GET',
    mode: 'cors',
    headers: {
      'Content-Type': 'application/json',
    },
  })
    .then((response) => response.json())
    .then((data) => data);

export const getNoteById = (noteId: number): Promise<Note> =>
  fetch(`${BASE_URL}/notes/${noteId}`, {
    method: 'GET',
    mode: 'cors',
    headers: {
      'Content-Type': 'application/json',
    },
  })
    .then((response) => response.json())
    .then((data) => data[0]);

export const addNote = (note: Note): Promise<Note> =>
  fetch(`${BASE_URL}/notes`, {
    method: 'POST',
    mode: 'cors',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(note),
  }).then((response) => response.json());

export const updateNote = (note: Note): Promise<Note> =>
  fetch(`${BASE_URL}/notes/${note.id}`, {
    method: 'PUT',
    mode: 'cors',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(note),
  }).then((response) => response.json());

export const removeNote = (id: string): Promise<void> =>
  fetch(`${BASE_URL}/notes/${id}`, {
    method: 'DELETE',
    mode: 'cors',
    headers: {
      'Content-Type': 'application/json',
    },
  }).then((response) => response.json());

export const mockNotes: Note[] = [
  {
    id: 'fbaxd',
    title: 'My first note',
    description: 'This is a description of the first note',
    color: 'green',
  },
  {
    id: 'ho2qe9',
    title: 'Second note',
    description: 'This is a description of the second note',
    color: 'red',
  },
  {
    id: 'gnlls',
    title: 'Third note',
    description: 'This is a description of the third note',
    color: 'yellow',
  },
];
```

### Adding basic store functions

We are going to fully utilize the `Zustand` functionality later on, for now, let's just add some basic functions that handling data manipulation. In the `store` folder, let's create `index.ts` with the following content:

```typescript
// src/store/index.ts
import type { Note } from '../api';

const addNote = (
  notes: Note[],
  payload: Note | null,
  id: string = Math.random().toString(36).substring(7)
): Note[] =>
  payload !== null
    ? [
        ...notes,
        {
          ...payload,
          id,
        },
      ]
    : notes;

const updateNote = (notes: Note[], payload: Note | null): Note[] =>
  payload !== null
    ? notes.map((note) => (note.id === payload.id ? payload : note))
    : notes;

const removeNote = (notes: Note[], id: string): Note[] =>
  notes.filter((note) => note.id !== id);

const setActiveNote = (
  activeNote: Note | null,
  payload: Partial<Note>
): Note => ({
  id: activeNote?.id ?? '',
  title: payload.title ?? activeNote?.title ?? '',
  color: payload.color ?? activeNote?.color ?? '',
  description: payload.description ?? activeNote?.description ?? '',
});

const findNoteById = (notes: Note[], id: string): Note | null =>
  notes.find((note) => note.id === id) || null;
```

### Wrap-up the functionality by adding simple UI

Last piece of functionality in this section is to write some components containing the UI. Let's create three files (**AddEditForm.tsx**, **NoteList.tsx**, and **NoteTable.tsx**) in the **components** folders with the following content:

```typescript
// src/components/AddEditForm.tsx
import React from 'react';

export const AddEditForm = () => {
  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
  };

  return (
    <div className="flex items-center bg-gray-50 dark:bg-gray-900">
      <div className="container mx-auto">
        <div className="max-w-md mx-auto my-10 bg-white p-5 rounded-md shadow-sm">
          <div className="text-center">
            <h1 className="my-3 text-3xl font-semibold text-gray-700 dark:text-gray-200">
              My Notes
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
                  onChange={(event) => {}}
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
                  onChange={(event) => {}}
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
                  rows={5}
                  name="message"
                  id="message"
                  placeholder="Your Message"
                  className="w-full px-3 py-2 placeholder-gray-300 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-indigo-100 focus:border-indigo-300 dark:bg-gray-700 dark:text-white dark:placeholder-gray-500 dark:border-gray-600 dark:focus:ring-gray-900 dark:focus:border-gray-500"
                  onChange={(event) => {}}
                ></textarea>
              </div>
              <div className="mb-6">
                <button
                  type="submit"
                  className="w-full px-3 py-4 text-white bg-indigo-500 rounded-md focus:bg-indigo-600 focus:outline-none"
                >
                  Submit
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

```typescript
// src/components/NoteList.tsx
import { AddEditForm } from './AddEditForm';
import { NoteTable } from './NoteTable';

export const NoteList = () => {
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

      <AddEditForm />

      <div className="bg-white shadow-md rounded my-6">
        <NoteTable />
      </div>
    </div>
  );
};
```

```typescript
// src/components/NoteTable.tsx
import React from 'react';
import { mockNotes } from '../api';
import type { Note } from '../api';

export const NoteTable = () => {
  const [notes] = React.useState<Note[]>(mockNotes);

  return (
    <table className="text-left w-full border-collapse">
      <thead>
        <tr>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            #id
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Title
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Color
          </th>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Actions
          </th>
        </tr>
      </thead>
      <tbody>
        {notes.map((note) => (
          <tr className="hover:bg-grey-lighter" key={note.id}>
            <td className="py-4 px-2 border-b border-grey-light">{note.id}</td>
            <td className="py-4 border-b border-grey-light">{note.title}</td>
            <td className="py-4 border-b border-grey-light">
              <span
                className={`rounded text-xs py-2 px-10 text-white font-bold bg-${note.color}-500`}
              ></span>
            </td>
            <td className="border-b border-grey-light">
              <button className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs hover:bg-green-dark">
                Edit
              </button>
              <button className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs text-white bg-red-500 rounded-md">
                Delete
              </button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
};
```

Last thing we need to do is to update the main `App.tsx` file. Let's replace the initial component with the following code:

```typescript
// src/App.tsx
import React from 'react';
import { NoteList } from './components/NoteList';

export const App = () => {
  return <NoteList />;
};

export default App;
```

And when we type `npm run start` and open the browser. We should see a basic page with static data and simple form.

## Adding basic store functionality

In this section, let's connect the existing functionality with the Zustand store. In the `store/index.ts`, let's add the following code:

```diff
--- a/index.ts
+++ b/index.ts
// src/store/index.ts
import type { Note } from '../api';
+import create from 'zustand';

const addNote = (
  notes: Note[],
  payload: Note | null,
  id: string = Math.random().toString(36).substring(7)
): Note[] =>
  payload !== null
    ? [
        ...notes,
        {
          ...payload,
          id,
        },
      ]
    : notes;

const updateNote = (notes: Note[], payload: Note | null): Note[] =>
  payload !== null
    ? notes.map((note) => (note.id === payload.id ? payload : note))
    : notes;

const removeNote = (notes: Note[], id: string): Note[] =>
  notes.filter((note) => note.id !== id);

const setActiveNote = (
  activeNote: Note | null,
  payload: Partial<Note>
): Note => ({
  id: activeNote?.id ?? '',
  title: payload.title ?? activeNote?.title ?? '',
  color: payload.color ?? activeNote?.color ?? '',
  description: payload.description ?? activeNote?.description ?? '',
});

const findNoteById = (notes: Note[], id: string): Note | null =>
  notes.find((note) => note.id === id) || null;

+type Store = {
+ notes: Note[]; // This array contains  a list of stored notes.
+ activeNote: Note | null; // This object contains either a work-in-progress note (no id set) or existing one (editing existing one).
+ isEditing: () => boolean; // This function verifies whether there is an edit or not (a simple usage of get()).
+ adjustActiveNote: (payload: Partial<Note>) => void; // This function actively adjusts active note, either creating a new one or editing existing one (selected by getExistingNote function).
+ getExistingNote: (id: string) => void; // This one sets existing note into activeNote object.
+ submitNewNote: () => void; // This one merges new note with existing ones.
+ updateExistingNote: () => void; // This one updates stored notes by merging activeNote.
+ deleteExistingNote: (id: string) => void; // This one removes a specific note from the stored notes array.
+ fetchNotes: (notes: Note[]) => void; // This one is used for setting multiple elements at the same time.
+};

+export const useStore = create<Store>(
+ (set, get): Store => ({
+   notes: [],
+   activeNote: null,
+   isEditing: () => get().activeNote?.id !== '',
+   adjustActiveNote: (payload: Partial<Note>) =>
+     set((state) => ({
+       ...state, // Or we can remove this, zustand handles the update automatically
+       activeNote: setActiveNote(state.activeNote, payload),
+     })),
+   getExistingNote: (id: string) =>
+     set((state) => ({
+       activeNote: findNoteById(state.notes, id),
+     })),
+   submitNewNote: () =>
+     set((state) => ({
+       notes: addNote(state.notes, state.activeNote),
+       activeNote: null,
+     })),
+   updateExistingNote: () =>
+     set((state) => ({
+       notes: updateNote(state.notes, state.activeNote),
+       activeNote: null,
+     })),
+   deleteExistingNote: (id: string) =>
+     set((state) => ({
+       notes: removeNote(state.notes, id),
+     })),
+   fetchNotes: (notes: Note[]) =>
+     set((state) => ({
+       notes,
+     })),
+ })
+);
```

Once we have our store defined, we can easily apply it in our components. Let's make the following changes in `NoteTable.tsx`, and `AddEditForm.tsx`.

```diff
--- a/NoteTable.tsx
+++ b/NoteTable.tsx
// src/components/NoteTable.tsx
import React from 'react';
import { mockNotes } from '../api';
+import { useStore } from '../store';
-import type { Note } from '../api';

export const NoteTable = () => {
  const [notes] = React.useState<Note[]>(mockNotes);

+ React.useEffect(() => {
+   store.fetchNotes(mockNotes);
+ }, []);

  return (
    <table className="text-left w-full border-collapse">
      <thead>
        <tr>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            #id
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Title
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Color
          </th>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Actions
          </th>
        </tr>
      </thead>
      <tbody>
-       {notes.map((note) => (
+       {store.notes.map((note) => (
          <tr className="hover:bg-grey-lighter" key={note.id}>
            <td className="py-4 px-2 border-b border-grey-light">{note.id}</td>
            <td className="py-4 border-b border-grey-light">{note.title}</td>
            <td className="py-4 border-b border-grey-light">
              <span
                className={`rounded text-xs py-2 px-10 text-white font-bold bg-${note.color}-500`}
              ></span>
            </td>
            <td className="border-b border-grey-light">
-             <button className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs hover:bg-green-dark">
+             <button
+               onClick={() => store.getExistingNote(note.id)}
+               className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs hover:bg-green-dark"
+             >
                Edit
              </button>
-             <button className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs text-white bg-red-500 rounded-md">
+             <button
+               onClick={() => store.deleteExistingNote(note.id)}
+               className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs text-white bg-red-500 rounded-md"
+             >
                Delete
              </button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
};
```

```diff
--- a/AddEditForm.tsx
+++ b/AddEditForm.tsx
// src/components/AddEditForm.tsx
import React from 'react';
+import { useStore } from '../store';
+import type { NoteColor } from '../api';

export const AddEditForm = () => {
+ const store = useStore((state) => state);

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();

+   if (!store.isEditing()) {
+     store.submitNewNote();
+   } else {
+     store.updateExistingNote();
+   }
  };

  return (
    <div className="flex items-center bg-gray-50 dark:bg-gray-900">
      <div className="container mx-auto">
        <div className="max-w-md mx-auto my-10 bg-white p-5 rounded-md shadow-sm">
          <div className="text-center">
            <h1 className="my-3 text-3xl font-semibold text-gray-700 dark:text-gray-200">
              My Notes
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
+                 value={store.activeNote?.title || ''}
-                 onChange={(event) => {}}
+                 onChange={(event) => {
+                   store.adjustActiveNote({ title: event.target.value });
+                 }}
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
+                 value={store.activeNote?.color || ''}
-                 onChange={(event) => {}}
+                 onChange={(event) => {
+                   store.adjustActiveNote({
+                     color: event.target.value as NoteColor,
+                   });
+                 }}
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
                  rows={5}
                  name="message"
                  id="message"
                  placeholder="Your Message"
                  className="w-full px-3 py-2 placeholder-gray-300 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-indigo-100 focus:border-indigo-300 dark:bg-gray-700 dark:text-white dark:placeholder-gray-500 dark:border-gray-600 dark:focus:ring-gray-900 dark:focus:border-gray-500"
+                 value={store.activeNote?.description || ''}
-                 onChange={(event) => {}}
+                  onChange={(event) => {
+                   store.adjustActiveNote({
+                     description: event.target.value,
+                   });
                  }}
                ></textarea>
              </div>
              <div className="mb-6">
                <button
                  type="submit"
                  className="w-full px-3 py-4 text-white bg-indigo-500 rounded-md focus:bg-indigo-600 focus:outline-none"
                >
                  Submit
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

The latest state of our files look as the following:

```typescript
// src/store/index.ts
import type { Note } from '../api';
import create from 'zustand';

const addNote = (notes: Note[], payload: Note | null): Note[] =>
  payload !== null
    ? [
        ...notes,
        {
          ...payload,
          id: Math.random().toString(36).substring(7),
        },
      ]
    : notes;

const updateNote = (notes: Note[], payload: Note | null): Note[] =>
  payload !== null
    ? notes.map((note) => (note.id === payload.id ? payload : note))
    : notes;

const removeNote = (notes: Note[], id: string): Note[] =>
  notes.filter((note) => note.id !== id);

const setActiveNote = (
  activeNote: Note | null,
  payload: Partial<Note>
): Note => ({
  id: activeNote?.id ?? '',
  title: payload.title ?? activeNote?.title ?? '',
  color: payload.color ?? activeNote?.color ?? '',
  description: payload.description ?? activeNote?.description ?? '',
});

const findNoteById = (notes: Note[], id: string): Note | null =>
  notes.find((note) => note.id === id) || null;

type Store = {
  notes: Note[]; // This array contains  a list of stored notes.
  activeNote: Note | null; // This object contains either a work-in-progress note (no id set) or existing one (editing existing one).
  isEditing: () => boolean; // This function verifies whether there is an edit or not (a simple usage of get()).
  adjustActiveNote: (payload: Partial<Note>) => void; // This function actively adjusts active note, either creating a new one or editing existing one (selected by getExistingNote function).
  getExistingNote: (id: string) => void; // This one sets existing note into activeNote object.
  submitNewNote: () => void; // This one merges new note with existing ones.
  updateExistingNote: () => void; // This one updates stored notes by merging activeNote.
  deleteExistingNote: (id: string) => void; // This one removes a specific note from the stored notes array.
  fetchNotes: (notes: Note[]) => void; // This one is used for setting multiple elements at the same time.
};

export const useStore = create<Store>(
  (set, get): Store => ({
    notes: [],
    activeNote: null,
    isEditing: () => get().activeNote?.id !== '',
    adjustActiveNote: (payload: Partial<Note>) =>
      set((state) => ({
        ...state, // Or we can remove this, zustand handles the update automatically
        activeNote: setActiveNote(state.activeNote, payload),
      })),
    getExistingNote: (id: string) =>
      set((state) => ({
        activeNote: findNoteById(state.notes, id),
      })),
    submitNewNote: () =>
      set((state) => ({
        notes: addNote(state.notes, state.activeNote),
        activeNote: null,
      })),
    updateExistingNote: () =>
      set((state) => ({
        notes: updateNote(state.notes, state.activeNote),
        activeNote: null,
      })),
    deleteExistingNote: (id: string) =>
      set((state) => ({
        notes: removeNote(state.notes, id),
      })),
    fetchNotes: (notes: Note[]) =>
      set((state) => ({
        notes,
      })),
  })
);
```

```typescript
// src/components/NoteTable.tsx
import React from 'react';
import { mockNotes } from '../api';
import { useStore } from '../store';

export const NoteTable = () => {
  const store = useStore((state) => state);

  React.useEffect(() => {
    store.fetchNotes(mockNotes);
  }, []);

  return (
    <table className="text-left w-full border-collapse">
      <thead>
        <tr>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            #id
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Title
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Color
          </th>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Actions
          </th>
        </tr>
      </thead>
      <tbody>
        {store.notes.map((note) => (
          <tr className="hover:bg-grey-lighter" key={note.id}>
            <td className="py-4 px-2 border-b border-grey-light">{note.id}</td>
            <td className="py-4 border-b border-grey-light">{note.title}</td>
            <td className="py-4 border-b border-grey-light">
              <span
                className={`rounded text-xs py-2 px-10 text-white font-bold bg-${note.color}-500`}
              ></span>
            </td>
            <td className="border-b border-grey-light">
              <button
                onClick={() => store.getExistingNote(note.id)}
                className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs hover:bg-green-dark"
              >
                Edit
              </button>
              <button
                onClick={() => store.deleteExistingNote(note.id)}
                className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs text-white bg-red-500 rounded-md"
              >
                Delete
              </button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
};
```

```typescript
// src/components/AddEditForm.tsx
import React from 'react';
import { useStore } from '../store';
import type { NoteColor } from '../api';

export const AddEditForm = () => {
  const store = useStore((state) => state);

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();

    if (!store.isEditing()) {
      store.submitNewNote();
    } else {
      store.updateExistingNote();
    }
  };

  return (
    <div className="flex items-center bg-gray-50 dark:bg-gray-900">
      <div className="container mx-auto">
        <div className="max-w-md mx-auto my-10 bg-white p-5 rounded-md shadow-sm">
          <div className="text-center">
            <h1 className="my-3 text-3xl font-semibold text-gray-700 dark:text-gray-200">
              My Notes
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
                  value={store.activeNote?.title || ''}
                  onChange={(event) => {
                    store.adjustActiveNote({ title: event.target.value });
                  }}
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
                  value={store.activeNote?.color || ''}
                  onChange={(event) => {
                    store.adjustActiveNote({
                      color: event.target.value as NoteColor,
                    });
                  }}
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
                  rows={5}
                  name="message"
                  id="message"
                  placeholder="Your Message"
                  className="w-full px-3 py-2 placeholder-gray-300 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-indigo-100 focus:border-indigo-300 dark:bg-gray-700 dark:text-white dark:placeholder-gray-500 dark:border-gray-600 dark:focus:ring-gray-900 dark:focus:border-gray-500"
                  value={store.activeNote?.description || ''}
                  onChange={(event) => {
                    store.adjustActiveNote({
                      description: event.target.value,
                    });
                  }}
                ></textarea>
              </div>
              <div className="mb-6">
                <button
                  type="submit"
                  className="w-full px-3 py-4 text-white bg-indigo-500 rounded-md focus:bg-indigo-600 focus:outline-none"
                >
                  Submit
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

## Selecting multiple state slices

What we had done so far, pretty much cover the core concepts of Zustand. But there is more what can be done. Instead of using the same store at once, we can only pick desired pieces.

Let's see that in the following example:

```diff
--- a/NoteTable.tsx
+++ b/NoteTable.tsx
// src/components/NoteTable.tsx
import React from 'react';
import { mockNotes } from '../api';
import { useStore } from '../store';

export const NoteTable = () => {
- const store = useStore((state) => state);
+ const notes = useStore((state) => state.notes);
+ const fetchNotes = useStore((state) => state.fetchNotes);
+ const getExistingNote = useStore((state) => state.getExistingNote);
+ const deleteExistingNote = useStore((state) => state.deleteExistingNote);

  React.useEffect(() => {
-   store.fetchNotes(mockNotes);
+   fetchNotes(mockNotes);
- }, []);
+ }, [fetchNotes]);

  return (
    <table className="text-left w-full border-collapse">
      <thead>
        <tr>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            #id
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Title
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Color
          </th>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Actions
          </th>
        </tr>
      </thead>
      <tbody>
-       {store.notes.map((note) => (
+       {notes.map((note) => (
          <tr className="hover:bg-grey-lighter" key={note.id}>
            <td className="py-4 px-2 border-b border-grey-light">{note.id}</td>
            <td className="py-4 border-b border-grey-light">{note.title}</td>
            <td className="py-4 border-b border-grey-light">
              <span
                className={`rounded text-xs py-2 px-10 text-white font-bold bg-${note.color}-500`}
              ></span>
            </td>
            <td className="border-b border-grey-light">
              <button
-               onClick={() => store.getExistingNote(note.id)}
+               onClick={() => getExistingNote(note.id)}
                className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs hover:bg-green-dark"
              >
                Edit
              </button>
              <button
-               onClick={() => store.deleteExistingNote(note.id)}
+               onClick={() => deleteExistingNote(note.id)}
                className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs text-white bg-red-500 rounded-md"
              >
                Delete
              </button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
};
```

```diff
--- a/AddEditForm.tsx
+++ b/AddEditForm.tsx
// src/components/AddEditForm.tsx
import React from 'react';
import { useStore } from '../store';
import type { NoteColor } from '../api';

export const AddEditForm = () => {
- const store = useStore((state) => state);
+ const activeNote = useStore((state) => state.activeNote);
+ const isEditing = useStore((state) => state.isEditing);
+ const submitNewNote = useStore((state) => state.submitNewNote);
+ const updateExistingNote = useStore((state) => state.updateExistingNote);
+ const adjustActiveNote = useStore((state) => state.adjustActiveNote);

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();

-   if (!store.isEditing()) {
+   if (!isEditing()) {
-     store.submitNewNote();
+     submitNewNote();
    } else {
-     store.updateExistingNote();
+     updateExistingNote();
    }
  };

  return (
    <div className="flex items-center bg-gray-50 dark:bg-gray-900">
      <div className="container mx-auto">
        <div className="max-w-md mx-auto my-10 bg-white p-5 rounded-md shadow-sm">
          <div className="text-center">
            <h1 className="my-3 text-3xl font-semibold text-gray-700 dark:text-gray-200">
              My Notes
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
-                 value={store.activeNote?.title || ''}
+                 value={activeNote?.title || ''}
                  onChange={(event) => {
-                   store.adjustActiveNote({ title: event.target.value });
+                   adjustActiveNote({ title: event.target.value });
                  }}
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
-                 value={store.activeNote?.color || ''}
+                 value={activeNote?.color || ''}
                  onChange={(event) => {
-                   store.adjustActiveNote({
+                   adjustActiveNote({
                      color: event.target.value as NoteColor,
                    });
                  }}
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
                  rows={5}
                  name="message"
                  id="message"
                  placeholder="Your Message"
                  className="w-full px-3 py-2 placeholder-gray-300 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-indigo-100 focus:border-indigo-300 dark:bg-gray-700 dark:text-white dark:placeholder-gray-500 dark:border-gray-600 dark:focus:ring-gray-900 dark:focus:border-gray-500"
-                 value={store.activeNote?.description || ''}
+                 value={activeNote?.description || ''}
                  onChange={(event) => {
-                   store.adjustActiveNote({
+                   adjustActiveNote({
                      description: event.target.value,
                    });
                  }}
                ></textarea>
              </div>
              <div className="mb-6">
                <button
                  type="submit"
                  className="w-full px-3 py-4 text-white bg-indigo-500 rounded-md focus:bg-indigo-600 focus:outline-none"
                >
                  Submit
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

The latest state of our files look as the following:

```typescript
// src/components/NoteTable.tsx
import React from 'react';
import { mockNotes } from '../api';
import { useStore } from '../store';

export const NoteTable = () => {
  const notes = useStore((state) => state.notes);
  const fetchNotes = useStore((state) => state.fetchNotes);
  const getExistingNote = useStore((state) => state.getExistingNote);
  const deleteExistingNote = useStore((state) => state.deleteExistingNote);

  React.useEffect(() => {
    fetchNotes(mockNotes);
  }, [fetchNotes]);

  return (
    <table className="text-left w-full border-collapse">
      <thead>
        <tr>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            #id
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Title
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Color
          </th>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Actions
          </th>
        </tr>
      </thead>
      <tbody>
        {notes.map((note) => (
          <tr className="hover:bg-grey-lighter" key={note.id}>
            <td className="py-4 px-2 border-b border-grey-light">{note.id}</td>
            <td className="py-4 border-b border-grey-light">{note.title}</td>
            <td className="py-4 border-b border-grey-light">
              <span
                className={`rounded text-xs py-2 px-10 text-white font-bold bg-${note.color}-500`}
              ></span>
            </td>
            <td className="border-b border-grey-light">
              <button
                onClick={() => getExistingNote(note.id)}
                className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs hover:bg-green-dark"
              >
                Edit
              </button>
              <button
                onClick={() => deleteExistingNote(note.id)}
                className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs text-white bg-red-500 rounded-md"
              >
                Delete
              </button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
};
```

```typescript
// src/components/AddEditForm.tsx
import React from 'react';
import { useStore } from '../store';
import type { NoteColor } from '../api';

export const AddEditForm = () => {
  const activeNote = useStore((state) => state.activeNote);
  const isEditing = useStore((state) => state.isEditing);
  const submitNewNote = useStore((state) => state.submitNewNote);
  const updateExistingNote = useStore((state) => state.updateExistingNote);
  const adjustActiveNote = useStore((state) => state.adjustActiveNote);

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();

    if (!isEditing()) {
      submitNewNote();
    } else {
      updateExistingNote();
    }
  };

  return (
    <div className="flex items-center bg-gray-50 dark:bg-gray-900">
      <div className="container mx-auto">
        <div className="max-w-md mx-auto my-10 bg-white p-5 rounded-md shadow-sm">
          <div className="text-center">
            <h1 className="my-3 text-3xl font-semibold text-gray-700 dark:text-gray-200">
              My Notes
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
                  value={activeNote?.title || ''}
                  onChange={(event) => {
                    adjustActiveNote({ title: event.target.value });
                  }}
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
                  value={activeNote?.color || ''}
                  onChange={(event) => {
                    adjustActiveNote({
                      color: event.target.value as NoteColor,
                    });
                  }}
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
                  rows={5}
                  name="message"
                  id="message"
                  placeholder="Your Message"
                  className="w-full px-3 py-2 placeholder-gray-300 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-indigo-100 focus:border-indigo-300 dark:bg-gray-700 dark:text-white dark:placeholder-gray-500 dark:border-gray-600 dark:focus:ring-gray-900 dark:focus:border-gray-500"
                  value={activeNote?.description || ''}
                  onChange={(event) => {
                    adjustActiveNote({
                      description: event.target.value,
                    });
                  }}
                ></textarea>
              </div>
              <div className="mb-6">
                <button
                  type="submit"
                  className="w-full px-3 py-4 text-white bg-indigo-500 rounded-md focus:bg-indigo-600 focus:outline-none"
                >
                  Submit
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

## Async actions

We would like to work with a remote service, therefore we need to make sure we can work with them easily. **Zustand** allows you to manage the async patterns in a standard way and then update the state based on `set` functionality synchronously.

> Note: For the example below, make sure the deno service is running. If not, just type `deno run --allow-net mod.ts` in the [Deno API](https://github.com/radektomasek/frontend-community-workshops-archive/tree/main/deno-v1-intro)

```diff
--- a/index.ts
+++ b/index.ts
// src/store/index.ts
+import * as api from '../api';
import type { Note } from '../api';
import create from 'zustand';

const addNote = (
  notes: Note[],
  payload: Note | null,
  id: string = Math.random().toString(36).substring(7)
): Note[] =>
  payload !== null
    ? [
        ...notes,
        {
          ...payload,
          id,
        },
      ]
    : notes;

const updateNote = (notes: Note[], payload: Note | null): Note[] =>
  payload !== null
    ? notes.map((note) => (note.id === payload.id ? payload : note))
    : notes;

const removeNote = (notes: Note[], id: string): Note[] =>
  notes.filter((note) => note.id !== id);

const setActiveNote = (
  activeNote: Note | null,
  payload: Partial<Note>
): Note => ({
  id: activeNote?.id ?? '',
  title: payload.title ?? activeNote?.title ?? '',
  color: payload.color ?? activeNote?.color ?? '',
  description: payload.description ?? activeNote?.description ?? '',
});

const findNoteById = (notes: Note[], id: string): Note | null =>
  notes.find((note) => note.id === id) || null;

type Store = {
  notes: Note[]; // This array contains  a list of stored notes.
  activeNote: Note | null; // This object contains either a work-in-progress note (no id set) or existing one (editing existing one).
  isEditing: () => boolean; // This function verifies whether there is an edit or not (a simple usage of get()).
  adjustActiveNote: (payload: Partial<Note>) => void; // This function actively adjusts active note, either creating a new one or editing existing one (selected by getExistingNote function).
  getExistingNote: (id: string) => void; // This one sets existing note into activeNote object.
  submitNewNote: () => void; // This one merges new note with existing ones.
  updateExistingNote: () => void; // This one updates stored notes by merging activeNote.
  deleteExistingNote: (id: string) => void; // This one removes a specific note from the stored notes array.
  fetchNotes: (notes: Note[]) => void; // This one is used for setting multiple elements at the same time.
};

export const useStore = create<Store>(
  (set, get): Store => ({
    notes: [],
    activeNote: null,
    isEditing: () => get().activeNote?.id !== '',
    adjustActiveNote: (payload: Partial<Note>) =>
      set((state) => ({
        ...state, // Or we can remove this, zustand handles the update automatically
        activeNote: setActiveNote(state.activeNote, payload),
      })),
    getExistingNote: (id: string) =>
      set((state) => ({
        activeNote: findNoteById(state.notes, id),
      })),
-   submitNewNote: () =>
-    set((state) => ({
-      notes: addNote(state.notes, state.activeNote),
-      activeNote: null,
-    })),
+   submitNewNote: async () => {
+     const activeNote = get().activeNote;
+
+     if (activeNote) {
+       const result = await api.addNote(activeNote);
+
+       set((state) => ({
+         notes: addNote(state.notes, activeNote, result.id),
+         activeNote: null,
+       }));
+     }
+   },
-   updateExistingNote: () =>
-     set((state) => ({
-       notes: updateNote(state.notes, state.activeNote),
-       activeNote: null,
-     })),
+   updateExistingNote: async () => {
+     const activeNote = get().activeNote;
+
+     if (activeNote) {
+       await api.updateNote(activeNote);
+
+       set((state) => ({
+         notes: updateNote(state.notes, activeNote),
+         activeNote: null,
+       }));
+     }
+   },
-   deleteExistingNote: (id: string) =>
-     set((state) => ({
-       notes: removeNote(state.notes, id),
-     })),
+   deleteExistingNote: async (id: string) => {
+     await api.removeNote(id);
+
+     set((state) => ({
+       notes: removeNote(state.notes, id),
+     }));
+   },
-  fetchNotes: (notes: Note[]) =>
-     set((state) => ({
-       notes,
-     })),
+   fetchNotes: async () => {
+     const notes = await api.getNotes();
+
+     set((state) => ({
+       notes,
+     }));
+    },
  })
);
```

```diff
--- a/NoteTable.tsx
+++ b/NoteTable.tsx
// src/components/NoteTable.tsx
import React from 'react';
-import { mockNotes } from '../api';
import { useStore } from '../store';

export const NoteTable = () => {
  const notes = useStore((state) => state.notes);
  const fetchNotes = useStore((state) => state.fetchNotes);
  const getExistingNote = useStore((state) => state.getExistingNote);
  const deleteExistingNote = useStore((state) => state.deleteExistingNote);

  React.useEffect(() => {
-   fetchNotes(mockNotes);
+   fetchNotes();
  }, [fetchNotes]);

  return (
    <table className="text-left w-full border-collapse">
      <thead>
        <tr>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            #id
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Title
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Color
          </th>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Actions
          </th>
        </tr>
      </thead>
      <tbody>
        {notes.map((note) => (
          <tr className="hover:bg-grey-lighter" key={note.id}>
            <td className="py-4 px-2 border-b border-grey-light">{note.id}</td>
            <td className="py-4 border-b border-grey-light">{note.title}</td>
            <td className="py-4 border-b border-grey-light">
              <span
                className={`rounded text-xs py-2 px-10 text-white font-bold bg-${note.color}-500`}
              ></span>
            </td>
            <td className="border-b border-grey-light">
              <button
                onClick={() => getExistingNote(note.id)}
                className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs hover:bg-green-dark"
              >
                Edit
              </button>
              <button
                onClick={() => deleteExistingNote(note.id)}
                className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs text-white bg-red-500 rounded-md"
              >
                Delete
              </button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
};
```

The latest snapshot looks as the following:

```typescript
// src/store/index.ts
import * as api from '../api';
import type { Note } from '../api';
import create from 'zustand';

const addNote = (
  notes: Note[],
  payload: Note | null,
  id: string = Math.random().toString(36).substring(7)
): Note[] =>
  payload !== null
    ? [
        ...notes,
        {
          ...payload,
          id,
        },
      ]
    : notes;

const updateNote = (notes: Note[], payload: Note | null): Note[] =>
  payload !== null
    ? notes.map((note) => (note.id === payload.id ? payload : note))
    : notes;

const removeNote = (notes: Note[], id: string): Note[] =>
  notes.filter((note) => note.id !== id);

const setActiveNote = (
  activeNote: Note | null,
  payload: Partial<Note>
): Note => ({
  id: activeNote?.id ?? '',
  title: payload.title ?? activeNote?.title ?? '',
  color: payload.color ?? activeNote?.color ?? '',
  description: payload.description ?? activeNote?.description ?? '',
});

const findNoteById = (notes: Note[], id: string): Note | null =>
  notes.find((note) => note.id === id) || null;

type Store = {
  notes: Note[]; // This array contains  a list of stored notes.
  activeNote: Note | null; // This object contains either a work-in-progress note (no id set) or existing one (editing existing one).
  isEditing: () => boolean; // This function verifies whether there is an edit or not (a simple usage of get()).
  adjustActiveNote: (payload: Partial<Note>) => void; // This function actively adjusts active note, either creating a new one or editing existing one (selected by getExistingNote function).
  getExistingNote: (id: string) => void; // This one sets existing note into activeNote object.
  submitNewNote: () => void; // This one merges new note with existing ones.
  updateExistingNote: () => void; // This one updates stored notes by merging activeNote.
  deleteExistingNote: (id: string) => void; // This one removes a specific note from the stored notes array.
  fetchNotes: () => void; // This one is used for setting multiple elements at the same time.
};

export const useStore = create<Store>(
  (set, get): Store => ({
    notes: [],
    activeNote: null,
    isEditing: () => get().activeNote?.id !== '',
    adjustActiveNote: (payload: Partial<Note>) =>
      set((state) => ({
        ...state, // Or we can remove this, zustand handles the update automatically
        activeNote: setActiveNote(state.activeNote, payload),
      })),
    getExistingNote: (id: string) =>
      set((state) => ({
        activeNote: findNoteById(state.notes, id),
      })),
    submitNewNote: async () => {
      const activeNote = get().activeNote;

      if (activeNote) {
        const result = await api.addNote(activeNote);

        set((state) => ({
          notes: addNote(state.notes, activeNote, result.id),
          activeNote: null,
        }));
      }
    },
    updateExistingNote: async () => {
      const activeNote = get().activeNote;

      if (activeNote) {
        await api.updateNote(activeNote);

        set((state) => ({
          notes: updateNote(state.notes, activeNote),
          activeNote: null,
        }));
      }
    },
    deleteExistingNote: async (id: string) => {
      await api.removeNote(id);

      set((state) => ({
        notes: removeNote(state.notes, id),
      }));
    },
    fetchNotes: async () => {
      const notes = await api.getNotes();

      set((state) => ({
        notes,
      }));
    },
  })
);
```

```typescript
// src/components/NoteTable.tsx
import React from 'react';
import { useStore } from '../store';

export const NoteTable = () => {
  const notes = useStore((state) => state.notes);
  const fetchNotes = useStore((state) => state.fetchNotes);
  const getExistingNote = useStore((state) => state.getExistingNote);
  const deleteExistingNote = useStore((state) => state.deleteExistingNote);

  React.useEffect(() => {
    fetchNotes();
  }, [fetchNotes]);

  return (
    <table className="text-left w-full border-collapse">
      <thead>
        <tr>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            #id
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Title
          </th>
          <th className="py-4 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Color
          </th>
          <th className="py-4 px-2 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">
            Actions
          </th>
        </tr>
      </thead>
      <tbody>
        {notes.map((note) => (
          <tr className="hover:bg-grey-lighter" key={note.id}>
            <td className="py-4 px-2 border-b border-grey-light">{note.id}</td>
            <td className="py-4 border-b border-grey-light">{note.title}</td>
            <td className="py-4 border-b border-grey-light">
              <span
                className={`rounded text-xs py-2 px-10 text-white font-bold bg-${note.color}-500`}
              ></span>
            </td>
            <td className="border-b border-grey-light">
              <button
                onClick={() => getExistingNote(note.id)}
                className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs hover:bg-green-dark"
              >
                Edit
              </button>
              <button
                onClick={() => deleteExistingNote(note.id)}
                className="text-grey-lighter font-bold py-1 px-2 rounded underline text-xs text-white bg-red-500 rounded-md"
              >
                Delete
              </button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
};
```

## Using DEV tools

Another great advantage of `Zustand` is an easy integration with **Redux** ecosystem. For now, let's just discuss the most essential part - integration with Redux DevTools.

```diff
--- a/index.ts
+++ b/index.ts
// src/store/index.ts
import * as api from '../api';
import type { Note } from '../api';
import create from 'zustand';
+import { devtools } from 'zustand/middleware';

const addNote = (
  notes: Note[],
  payload: Note | null,
  id: string = Math.random().toString(36).substring(7)
): Note[] =>
  payload !== null
    ? [
        ...notes,
        {
          ...payload,
          id,
        },
      ]
    : notes;

const updateNote = (notes: Note[], payload: Note | null): Note[] =>
  payload !== null
    ? notes.map((note) => (note.id === payload.id ? payload : note))
    : notes;

const removeNote = (notes: Note[], id: string): Note[] =>
  notes.filter((note) => note.id !== id);

const setActiveNote = (
  activeNote: Note | null,
  payload: Partial<Note>
): Note => ({
  id: activeNote?.id ?? '',
  title: payload.title ?? activeNote?.title ?? '',
  color: payload.color ?? activeNote?.color ?? '',
  description: payload.description ?? activeNote?.description ?? '',
});

const findNoteById = (notes: Note[], id: string): Note | null =>
  notes.find((note) => note.id === id) || null;

type Store = {
  notes: Note[]; // This array contains  a list of stored notes.
  activeNote: Note | null; // This object contains either a work-in-progress note (no id set) or existing one (editing existing one).
  isEditing: () => boolean; // This function verifies whether there is an edit or not (a simple usage of get()).
  adjustActiveNote: (payload: Partial<Note>) => void; // This function actively adjusts active note, either creating a new one or editing existing one (selected by getExistingNote function).
  getExistingNote: (id: string) => void; // This one sets existing note into activeNote object.
  submitNewNote: () => void; // This one merges new note with existing ones.
  updateExistingNote: () => void; // This one updates stored notes by merging activeNote.
  deleteExistingNote: (id: string) => void; // This one removes a specific note from the stored notes array.
  fetchNotes: () => void; // This one is used for setting multiple elements at the same time.
};

export const useStore = create<Store>(
- (set, get): Store => ({
+ devtools((set, get): Store => ({
    notes: [],
    activeNote: null,
    isEditing: () => get().activeNote?.id !== '',
    adjustActiveNote: (payload: Partial<Note>) =>
      set((state) => ({
        ...state, // Or we can remove this, zustand handles the update automatically
        activeNote: setActiveNote(state.activeNote, payload),
      })),
    getExistingNote: (id: string) =>
      set((state) => ({
        activeNote: findNoteById(state.notes, id),
      })),
    submitNewNote: async () => {
      const activeNote = get().activeNote;

      if (activeNote) {
        const result = await api.addNote(activeNote);

        set((state) => ({
          notes: addNote(state.notes, activeNote, result.id),
          activeNote: null,
        }));
      }
    },
    updateExistingNote: async () => {
      const activeNote = get().activeNote;

      if (activeNote) {
        await api.updateNote(activeNote);

        set((state) => ({
          notes: updateNote(state.notes, activeNote),
          activeNote: null,
        }));
      }
    },
    deleteExistingNote: async (id: string) => {
      await api.removeNote(id);

      set((state) => ({
        notes: removeNote(state.notes, id),
      }));
    },
    fetchNotes: async () => {
      const notes = await api.getNotes();

      set((state) => ({
        notes,
      }));
    },
  })
-);
+));
```

And if we look at details in the Redux tools, we can see nicely adjusted updates.

The latest snapshot of the `store/index.ts` looks as the following:

```typescript
// src/store/index.ts
import * as api from '../api';
import type { Note } from '../api';
import create from 'zustand';
import { devtools } from 'zustand/middleware';

const addNote = (
  notes: Note[],
  payload: Note | null,
  id: string = Math.random().toString(36).substring(7)
): Note[] =>
  payload !== null
    ? [
        ...notes,
        {
          ...payload,
          id,
        },
      ]
    : notes;

const updateNote = (notes: Note[], payload: Note | null): Note[] =>
  payload !== null
    ? notes.map((note) => (note.id === payload.id ? payload : note))
    : notes;

const removeNote = (notes: Note[], id: string): Note[] =>
  notes.filter((note) => note.id !== id);

const setActiveNote = (
  activeNote: Note | null,
  payload: Partial<Note>
): Note => ({
  id: activeNote?.id ?? '',
  title: payload.title ?? activeNote?.title ?? '',
  color: payload.color ?? activeNote?.color ?? '',
  description: payload.description ?? activeNote?.description ?? '',
});

const findNoteById = (notes: Note[], id: string): Note | null =>
  notes.find((note) => note.id === id) || null;

type Store = {
  notes: Note[]; // This array contains  a list of stored notes.
  activeNote: Note | null; // This object contains either a work-in-progress note (no id set) or existing one (editing existing one).
  isEditing: () => boolean; // This function verifies whether there is an edit or not (a simple usage of get()).
  adjustActiveNote: (payload: Partial<Note>) => void; // This function actively adjusts active note, either creating a new one or editing existing one (selected by getExistingNote function).
  getExistingNote: (id: string) => void; // This one sets existing note into activeNote object.
  submitNewNote: () => void; // This one merges new note with existing ones.
  updateExistingNote: () => void; // This one updates stored notes by merging activeNote.
  deleteExistingNote: (id: string) => void; // This one removes a specific note from the stored notes array.
  fetchNotes: () => void; // This one is used for setting multiple elements at the same time.
};

export const useStore = create<Store>(
  devtools(
    (set, get): Store => ({
      notes: [],
      activeNote: null,
      isEditing: () => get().activeNote?.id !== '',
      adjustActiveNote: (payload: Partial<Note>) =>
        set((state) => ({
          ...state, // Or we can remove this, zustand handles the update automatically
          activeNote: setActiveNote(state.activeNote, payload),
        })),
      getExistingNote: (id: string) =>
        set((state) => ({
          activeNote: findNoteById(state.notes, id),
        })),
      submitNewNote: async () => {
        const activeNote = get().activeNote;

        if (activeNote) {
          const result = await api.addNote(activeNote);

          set((state) => ({
            notes: addNote(state.notes, activeNote, result.id),
            activeNote: null,
          }));
        }
      },
      updateExistingNote: async () => {
        const activeNote = get().activeNote;

        if (activeNote) {
          await api.updateNote(activeNote);

          set((state) => ({
            notes: updateNote(state.notes, activeNote),
            activeNote: null,
          }));
        }
      },
      deleteExistingNote: async (id: string) => {
        await api.removeNote(id);

        set((state) => ({
          notes: removeNote(state.notes, id),
        }));
      },
      fetchNotes: async () => {
        const notes = await api.getNotes();

        set((state) => ({
          notes,
        }));
      },
    })
  )
);
```

## Conclusion

Thanks for following this workshop. Any feedback appreciated (radek.tomasek@gmail.com).
