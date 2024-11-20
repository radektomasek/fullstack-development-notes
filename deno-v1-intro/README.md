# Introduction to Deno (v1) - Building a Notes App

This workshops covers basics of server-side development with [Deno v1](https://deno.land/). It shows how to build a simple REST API with [Oak Framework](https://oakserver.github.io/oak) and how to nicely incorporate things like logging and serving static assets.

> Note: Recently [Deno announced the version 2.0](https://deno.com/blog/v2.0) that improves the DX and the runtime significantly. This material covers the older version, but I will try to create a separate material for the newer one.

## Deno and http frameworks

Deno is relatively new technology created by Ryan Dahl who wanted to address the main problems of his former creation - node.js. A native support of TypeScript and Promises with a standard library for common things are one of the main benefits.

In terms of web framework, there are (as you can expect in JavaScript world) many alternatives already and you can choose from:

- [Servest](https://servestjs.org/) - second most popular option, nice support for web stuff, React.
- [Pogo Framework](https://github.com/sholladay/pogo) - based on hapi, a support for JSX.
- [Deno Express](https://github.com/NMathar/deno-express) - based on Node.js express.
- [Drash Framework](https://github.com/drashland/deno-drash) - inspired by Python Ecosystem.
- [Oak Framework](https://oakserver.github.io/oak) - the number one framework, based on Koa patterns. Will be covered in this workshop.

## Prerequisites

This workshop was built with little (to none) prerequisites in mind. A massive benefit for deno is a standard library with baked-in tooling for testing, documentation and other common tasks. Deno itself is packed as a single executable in the first place.

To be able to follow and code along, make sure you have these two things installed on your computer.

- [Deno](https://deno.land/) - follow the [installation section](https://deno.land/#installation) on the main page and you should be able to get deno to your computer.
- [Denon](https://deno.land/x/denon@2.4.4) - to be able to improve the development experience (DX), I recommend to install this tool to your computer as well. Another single executable file will be stored alongside your deno and help you to skip the necessity to restart your server manually every time you would like to incorporate your change. It is a direct replacement for [Nodemon](https://www.npmjs.com/package/nodemon).
- (Optional) [Deno Visual Studio Code Extension](https://marketplace.visualstudio.com/items?itemName=denoland.vscode-deno) - this might come handy you would like to have a support for Deno in your Visual Studio Code. It is a good extension, but it isn't fully reliable yet. I recommend to install it, however, also please make sure you won't enable it globally to avoid a collision with your standard JS/node tooling.

After you install all of these items mentioned above, you shouldn't have any problem to follow and code along.

## Getting started

Before we start putting any initial implementation in place, let's see what our initial structure of the project contains.

The good things is that when it comes to initialization, not many things are required. The main thing available in this commit is the **public folder** that contains a simple frontend app written in plain JS which will be served directly from the backend. In practical world, you would probably use web components or React (or any other framework).

Another file worth mentioning is `scripts.js`. This file is for dev only purposes and it is connected to `denon`, a tool that reloads your server automatically on every change. It is not required to have this file created (you can parametrize it in the terminal), but I prefer to have this file over manual parameters. When you run `deno`, the file is ignored completely.

## Adding dependencies and bootstrapping the server

In Deno, there is a convention to put every dependency into a dedicated file which is called `deps.ts` (or `.js`) for standard dependencies and `test_deps.ts` (or `.js`) for testing specific tools. For now, let's create a `deps.ts` for the main app dependencies.

```typescript
// deps.ts
// standard library dependencies
export { v4 } from 'https://deno.land/std@0.74.0/uuid/mod.ts';
export * as log from 'https://deno.land/std@0.74.0/log/mod.ts';

// third party dependencies
export * as oak from 'https://deno.land/x/oak@v6.3.1/mod.ts';
```

Before the actual implementation, let's also create a simple model. I initially considered using `MongoDB`, but I decided to go with a simple in-memory model for two reasons which are the following:

- The [Deno MongoDB driver](https://github.com/manyuanrong/deno_mongo) is currently unstable
- I wanted to create a server which is easily deployable (either to an EC2 instance or lambda environment) with minimum dependencies considering our time constrains.

For that reason, let's create a simple model with `Map` data structure.

```typescript
// models/index.ts
import { v4 } from '../deps.ts';

export type NoteColor = 'red' | 'green' | 'yellow';

export interface User {
  id: string;
  name: string;
}

export const userId = v4.generate();
export const users = new Map<string, User>();
users.set(userId, { id: userId, name: 'Jack Kirby' });

export interface Note {
  id: string;
  title: string;
  description: string;
  color: NoteColor;
  userId: string;
}

export const generateSampleNoteData = (
  records: Map<string, Note>,
  id: string,
  title: string,
  description: string,
  color: NoteColor,
  userId: string
) => {
  records.set(id, { id, title, description, color, userId });
};

export const notes = new Map<string, Note>();
generateSampleNoteData(
  notes,
  v4.generate(),
  'My first note',
  'This is a description of the first note',
  'green',
  userId
);

generateSampleNoteData(
  notes,
  v4.generate(),
  'Second note',
  'This is a description of the second note',
  'red',
  userId
);

generateSampleNoteData(
  notes,
  v4.generate(),
  'Third note',
  'This is a description of the third note',
  'yellow',
  userId
);
```

We will also spend a lot of time creating routes for our server. For now, let's bootstrap a basic structure with a single route in place.

```typescript
// routes/index.ts
import { oak } from '../deps.ts';
import { notes } from '../models/index.ts';

export const router = new oak.Router();

router.get('/notes', (context: oak.RouterContext) => {
  context.response.body = Array.from(notes.values());
});
```

The main file of our project will be **mod.ts**. You can obviously choose any name, but it is one of the convention, recommended in the [Deno style guide](https://deno.land/manual@v1.5.0/contributing/style_guide), in particular, when you write a module and/or share your code with others.

```typescript
// mod.ts
import { log, oak } from './deps.ts';
import { router } from './routes/index.ts';

const app = new oak.Application();
const port = 8000;

app.use(router.routes());
app.use(router.allowedMethods());

await app.listen({ port });
```

Run the script with `denon start` or with plain deno by typing `deno run --allow-net mod.ts`.

If you use `denon`, everytime you make a change, the server will reload automatically.

Let's add a simple logging and see what is going to happen.

```typescript
// mod.ts
import { log, oak } from './deps.ts';
import { router } from './routes/index.ts';

const app = new oak.Application();
const port = 8000;

app.use(router.routes());
app.use(router.allowedMethods());

log.info(`Server is listening on port: ${port}`);
await app.listen({ port });
```

You can also make sure the module is **run as the main module** (it can be imported to other modules which `would make the condition falsy`)

```typescript
import { log, oak } from './deps.ts';
import { router } from './routes/index.ts';

const app = new oak.Application();
const port = 8000;

app.use(router.routes());
app.use(router.allowedMethods());

if (import.meta.main) {
  log.info(`Server is listening on port: ${port}`);
  await app.listen({ port });
}
```

## Setting up the routes

Let's start implementing all the routes we are going to use. The actual code will be refactored later, this version just shows the various options how to implement things.

```typescript
// routes/index.ts
import { oak } from '../deps.ts';
import { notes } from '../models/index.ts';

export const router = new oak.Router();

router.get('/notes', (context: oak.RouterContext) => {
  context.response.body = Array.from(notes.values());
});

router.get('/notes/:id', (context: oak.RouterContext) => {
  context.response.body = context.params.id;
});

router.post('/notes', (context: oak.RouterContext) => {
  context.response.status = 201;
});

router.put('/notes/:id', (context: oak.RouterContext) => {
  const { id } = oak.helpers.getQuery(context, { mergeParams: true });
  context.response.body = id;
});

router.delete('/notes/:id', (context: oak.RouterContext) => {
  context.throw(400, `Note with ID doesn't exist`);
});
```

We can also check, what happens when we try to get an unsupported method (e.g. PATCH) with commented/uncommented `.allowedMethods()`

```typescript
// mod.ts
import { log, oak } from './deps.ts';
import { router } from './routes/index.ts';

const app = new oak.Application();
const port = 8000;

app.use(router.routes());
// app.use(router.allowedMethods());

if (import.meta.main) {
  log.info(`Server is listening on port: ${port}`);
  await app.listen({ port });
}
```

Let's also incorporate two listeners, including a centralized error handling which can override the default behaviour.

```typescript
// mod.ts
import { log, oak } from './deps.ts';
import { router } from './routes/index.ts';

const app = new oak.Application();
const port = 8000;

app.addEventListener('listen', () => {
  log.info(`Server is listening on port: ${port}`);
});

app.addEventListener('error', (event) => {
  log.error(event.error);
});

app.use(async (context, next) => {
  try {
    await next();
  } catch (error) {
    context.response.status = 500;
    context.response.body = 'Internal server error';
    // throw error;
  }
});

app.use(router.routes());
app.use(router.allowedMethods());

if (import.meta.main) {
  await app.listen({ port });
}
```

To keep things simple, let's just pass the error without any override.

```typescript
// mod.ts
import { log, oak } from './deps.ts';
import { router } from './routes/index.ts';

const app = new oak.Application();
const port = 8000;

app.addEventListener('listen', () => {
  log.info(`Server is listening on port: ${port}`);
});

app.addEventListener('error', (event) => {
  log.error(event.error);
});

app.use(async (context, next) => {
  try {
    await next();
  } catch (error) {
    // context.response.status = 500;
    // context.response.body = 'Internal server error';
    throw error;
  }
});

app.use(router.routes());
app.use(router.allowedMethods());

if (import.meta.main) {
  await app.listen({ port });
}
```

## Finalizing the route implementation

Let's add the full implementation of the server routes. The first step would be about adding a custom middleware which pretends to work with the authorized user.

```typescript
// mod.ts
import { log, oak } from './deps.ts';
import { router } from './routes/index.ts';
import { users, userId } from './models/index.ts';

const app = new oak.Application();
const port = 8000;

app.addEventListener('listen', () => {
  log.info(`Server is listening on port: ${port}`);
});

app.addEventListener('error', (event) => {
  log.error(event.error);
});

app.use(async (context, next) => {
  context.state = { authUser: users.get(userId) };

  await next();
});

app.use(async (context, next) => {
  try {
    await next();
  } catch (error) {
    // context.response.status = 500;
    // context.response.body = 'Internal server error';
    throw error;
  }
});

app.use(router.routes());
app.use(router.allowedMethods());

if (import.meta.main) {
  await app.listen({ port });
}
```

The last main piece of the implementation is to update the routes and implement the actual logic for each of them.

```typescript
// routes/index.ts
import { v4 } from '../deps.ts';
import { oak } from '../deps.ts';
import { notes } from '../models/index.ts';

export const router = new oak.Router();

router.get('/notes', (context: oak.RouterContext) => {
  context.response.body = Array.from(notes.values());
});

router.get('/notes/:id', (context: oak.RouterContext) => {
  const { id } = oak.helpers.getQuery(context, { mergeParams: true });
  if (id && notes.has(id)) {
    context.response.body = notes.get(id);
  } else {
    context.throw(400, `Note with ID doesn't exist`);
  }
});

router.post('/notes', async (context: oak.RouterContext) => {
  const id = v4.generate();

  const { value } = context.request.body({ type: 'json' });
  const { title, description, color } = await value;

  notes.set(id, {
    id,
    title,
    description,
    color,
    userId: context.state.authUser.id,
  });

  context.response.status = 201;
  context.response.body = notes.get(id);
});

router.put('/notes/:id', async (context: oak.RouterContext) => {
  const { id } = oak.helpers.getQuery(context, { mergeParams: true });
  if (id && notes.has(id)) {
    const { value } = context.request.body({ type: 'json' });
    const { title, description, color } = await value;

    notes.set(id, {
      id,
      title,
      description,
      color,
      userId: context.state.authUser.id,
    });

    context.response.status = 200;
    context.response.body = notes.get(id);
  } else {
    context.throw(400, `Note with ID doesn't exist`);
  }
});

router.delete('/notes/:id', (context: oak.RouterContext) => {
  const { id } = oak.helpers.getQuery(context, { mergeParams: true });
  if (id && notes.has(id)) {
    const isDeleted = notes.delete(id);
    context.response.body = isDeleted;
  } else {
    context.throw(400, `Note with ID doesn't exist`);
  }
});
```

## Enabling the static content

This step is very simple, we need to add another middleware which would be responsible for serving the static content from our public folder. The important things to mention is that the static middleware has to be added at the end after the other routes.

```typescript
// mod.ts
import { log, oak } from './deps.ts';
import { router } from './routes/index.ts';
import { users, userId } from './models/index.ts';

const app = new oak.Application();
const port = 8000;

app.addEventListener('listen', () => {
  log.info(`Server is listening on port: ${port}`);
});

app.addEventListener('error', (event) => {
  log.error(event.error);
});

app.use(async (context, next) => {
  context.state = { authUser: users.get(userId) };

  await next();
});

app.use(async (context, next) => {
  try {
    await next();
  } catch (error) {
    // context.response.status = 500;
    // context.response.body = 'Internal server error';
    throw error;
  }
});

app.use(router.routes());
app.use(router.allowedMethods());

app.use(async (ctx) => {
  const filePath = ctx.request.url.pathname;
  const fileAllowList = [
    '/index.html',
    '/javascripts/script.js',
    '/stylesheets/style.css',
  ];

  if (fileAllowList.includes(filePath)) {
    await oak.send(ctx, filePath, {
      root: `${Deno.cwd()}/public`,
    });
  }
});

if (import.meta.main) {
  await app.listen({ port });
}
```

The project should be available on this address: `http://localhost:8000/index.html`

## Testing

Testing functionality is one of the many features baked in Deno. The full description is available in [the official documentation](https://deno.land/manual@v1.5.0/testing)

In a nutshell, there is an assertion module available in this link: [https://deno.land/std@0.74.0/testing/asserts.ts](https://deno.land/std@0.74.0/testing/asserts.ts).

Let's create a new `test_deps.ts` file that contains the following content.

```typescript
// test_deps.ts
export {
  assertEquals,
  assertNotEquals,
} from 'https://deno.land/std@0.74.0/testing/asserts.ts';
```

We can write some basics tests in the `demo.test.ts` file. In a nutshell, there are two forms how you can write your tests. A standard one (`test #1`) and modular one (`test #2`).

The main difference between the, is that the latter one allows us to put some extra parameters like setting the `ignore` flag or override the default safety measures (e.g. in case you want to ignore the number of opening files, for instance).

You can also type `deno` in your terminal and execute `Deno.metrics()`. This gives you a quick benchmark how the resources are utilized in your deno app.

```typescript
// demo.test.ts
import { assertEquals } from './test_deps.ts';

Deno.test('test #1', () => {
  const x = 1;
  assertEquals(x, 1);
});

Deno.test({
  name: 'test #2',
  ignore: false,
  sanitizeResources: false,
  sanitizeOps: false,
  fn: () => {
    const x = 1;
    Deno.open('./README.md');

    assertEquals(x, 1);
  },
});
```

You can just type `deno test --allow-read` and it works. There are some advanced use-cases (where you can filter tests by files, for instance). But for now, let's generate a test coverage by typing `deno test --coverage --unstable --allow-read`.

## Conclusion

Thanks for following this workshop. Any feedback appreciated (radek.tomasek@gmail.com).
