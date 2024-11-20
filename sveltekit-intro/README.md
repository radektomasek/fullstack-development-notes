# Intro to SvelteKit

In this workshop, we are going to work with **SvelteKit** and see a great way how to use [Svelte.js](https://svelte.dev) for building large scale applications.

In the past, we covered **basics of Svelte.js** during our Frontend Group sessions.

SvelteKit is a framework built on top of Svelte.js that simplifies the development of large apps by providing a great development experience and supporting all major features. It's a successor of [Sapper](https://svelte.dev/blog/whats-the-deal-with-sveltekit)

> If you are familiar with Next.js for React and Nuxt for Vue, you might find a lot of similarities.

## Project Initialization

In this section, we are going to initialize the project.

> Note: Under the hood, there is [Vite 2](https://vitejs.dev/) which a ESM bundler similar to Snowpack 3. When I started following SvelteKit, Snowpack was used as the main bundler [but it was ditched by Vite 2](https://www.reddit.com/r/sveltejs/comments/m0ftf2/waitwhat_sveltekit_drops_snowpack_for_vite_2/).

To initialize the project, just type the following in your terminal:

```bash
# project initialization
npm init svelte@next sveltekit-intro
```

It initializes a simple command-line interface where you can choose various options (linting, prettier by default, TS support, just to name a few).

Free free to tweak the options, but in our case, there are just some basics selections (skeleton project, JavaScript, prettier and eslint).

> Note: Make sure you have node.js version at least 12.0.0 to be able to install the dependencies correctly.

Once the project is initialized, you need to install npm modules, by typing:

```bash
# change the directory
cd sveltekit-intro

# install the packages
npm install
```

Another step now is to install `Tailwind`. With `SvelteKit` and `Tailwind JIT` the process is very straightforward.

Let's start by installing Tailwind and PostCSS:

```bash
npm install --save-dev tailwindcss postcss autoprefixer postcss-cli @tailwindcss/jit
```

Following that, let's add a few more files by typing:

```bash
# create an empty postcss.config.cjs file
touch postcss.config.cjs
# create an empty styles directory and add tailwind.css file
mkdir src/styles && touch src/styles/tailwind.css
# add command initializing the tailwind config
npx tailwindcss init tailwind.config.cjs
```

In `src/styles/tailwind.css`, let's add the following code:

```css
/* src/styles/tailwind.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

In the `postcss.config.cjs`, use this code:

```javascript
// postcss.config.cjs
module.exports = {
	plugins: {
		autoprefixer: {},
		tailwindcss: {}
	}
};
```

And in the `tailwind.config.cjs`, we need to make the following changes:

```diff
--- a/tailwind.config.cjs
+++ b/tailwind.config.jcs
// tailwind.config.cjs
module.exports = {
+ mode: 'jit',
- purge: [],
+ purge: ['./src/**/*.svelte'],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
}
```

So, the final version of the file looks like as the following:

```javascript
// tailwind.config.cjs
module.exports = {
	mode: 'jit',
	purge: ['./src/**/*.svelte'],
	darkMode: false, // or 'media' or 'class'
	theme: {
		extend: {}
	},
	variants: {
		extend: {}
	},
	plugins: []
};
```

Now, as things have been setup, we can just make a quick test and update the `src/routes/index.svelte`:

```diff
--- a/index.svelte
+++ b/index.svelte
<!-- src/routes/index.svelte -->
+<script>
+	import '../styles/tailwind.css';
+</script>

<h1 class="text-4xl text-center my-8 uppercase">Welcome to SvelteKit</h1>
-<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>
```

The latest snapshot of the `index.svelte` looks as the following:

```html
<!-- src/routes/index.svelte -->
<script>
	import '../styles/tailwind.css';
</script>

<h1 class="text-4xl text-center my-8 uppercase">Welcome to SvelteKit</h1>
```

If we type `npm run dev -- --open` in the terminal, we should be able to see a simple upper-cased title. Which is a great pre-requisite for next steps.

## Add helpers and basics components

In this section, we are going to create basic template structure and add additional files which help us to easily copy and paste them later in the workshop and help us to focus on the core principles of Svelte/SvelteKit whilst skipping the boilerplate part.

### snippets/about.html

```html
<!-- src/snippets/about.html -->
<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				alt="About Page"
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
				src="https://images.punkapi.com/v2/4.png"
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">About</h1>
				<p class="leading-relaxed">
					Lorem ipsum dolor sit amet consectetur adipisicing elit. Quos obcaecati
					reiciendis ad corrupti deleniti ipsam ab quidem veritatis assumenda? Cum
					hic ullam nobis illo. Explicabo repellat quas voluptates harum velit!
				</p>
			</div>
		</div>
	</div>
</section>
```

### snippets/detail.html

```html
<!-- src/snippets/detail.html -->
<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
				<h2 class="text-sm title-font text-gray-500 tracking-widest">Tagline</h2>
				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">Name</h1>
				<p class="leading-relaxed">Description</p>
				<div class="flex mt-4">
					<button
						class="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
					>
						Back
					</button>
				</div>
			</div>
		</div>
	</div>
</section>
```

### snippets/navbar.html

```html
<!-- src/snippets/navbar.html -->
<div class="flex">
	<div
		class="flex flex-wrap justify-between w-screen h-20 text-white bg-black md:flex-nowrap"
	>
		<div class="z-30 flex items-center h-full pl-3 space-x-3 bg-black">
			<p class="text-2xl text-">SvelteKit Demo</p>
		</div>
		<!-- Menu -->
		<div
			class="flex flex-col items-stretch w-screen text-xl text-center transform bg-black md:flex-row md:translate-y-0 md:space-x-5 md:items-center md:justify-end md:pr-3"
		>
			<a href="/" class="h-10 leading-10 border-b-2 border-dotted md:border-none"
				>Home</a
			>
			<a
				href="/about"
				class="h-10 leading-10 border-b-2 border-dotted md:border-none"
				>About</a
			>
		</div>
	</div>
</div>
```

### snippets/index.html

```html
<!-- src/snippets/index.html -->
<div id="menu" class="container mx-auto px-4 lg:pt-24 lg:pb-64">
	<div class="flex flex-wrap text-center justify-center">
		<div class="w-full lg:w-6/12 px-4">
			<h2 class="text-4xl font-semibold text-black">Brewdog's Beer Selection</h2>
			<p class="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
				Let's enjoy Brewdog's selection of beer whilst developing a simple page with
				SvelteKit.
			</p>
			<div class="flex justify-center">
				<button
					class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
				>
					All Beers
				</button>
				<button
					class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
				>
					Pilsner
				</button>
				<button
					class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
				>
					Lager
				</button>
				<button
					class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
				>
					Ale
				</button>
			</div>
		</div>
	</div>
	<div class="flex flex-wrap mt-12 justify-center">
		<div
			class="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4"
		>
			<div class="col-span-2 sm:col-span-1 xl:col-span-1">
				<img alt="" src="" class="h-20 w-8 rounded  mx-auto" />
			</div>
			<div class="col-span-2 sm:col-span-4 xl:col-span-4">
				<h3 class="font-semibold text-black">beer name</h3>
				<p>Beer description</p>
			</div>
			<div class="col-span-2 sm:col-span-1 xl:col-span-1">
				<a href="#" class="underline">Detail</a>
			</div>
		</div>
	</div>
</div>
```

### utils/http.js

```javascript
// src/utils/http.js
const BASE_URL = 'https://api.punkapi.com/v2';

export const getBeerList = (page, perPage = 25) =>
	fetch(
		page ? `${BASE_URL}/beers?page=${page}&per_page=${perPage}` : `${BASE_URL}/beers`
	)
		.then((response) => response.json())
		.then((data) => data);

export const getBeerById = (beerId) =>
	fetch(`${BASE_URL}/beers/${beerId}`)
		.then((response) => response.json())
		.then((data) => data[0]);

export const getListOfBeersByName = (beerName = '') =>
	fetch(`${BASE_URL}/beers?beer_name=${beerName.replace(' ', '_')}`)
		.then((response) => response.json())
		.then((data) => data);
```

## First steps with SvelteKit

In this section, we are going to **add a new route** that demonstrates how easy it's to do such a thing (spoiler alert, it's the same as in Next/Nuxt) and we will also check how to manage the structure in case you would like to have a piece of functionality on multiple pages.

### Adding a new route

Adding a new route is very straightforward. You just `need to create a new file` within `routes` directory and the file is going to represent the name of the route.

Let's copy the content of the `snippets/about.html` into `routes/about.svelte`.

```html
<!-- src/routes/about.svelte -->
<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				alt="About Page"
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
				src="https://images.punkapi.com/v2/4.png"
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">About</h1>
				<p class="leading-relaxed">
					Lorem ipsum dolor sit amet consectetur adipisicing elit. Quos obcaecati
					reiciendis ad corrupti deleniti ipsam ab quidem veritatis assumenda? Cum
					hic ullam nobis illo. Explicabo repellat quas voluptates harum velit!
				</p>
			</div>
		</div>
	</div>
</section>
```

Now, we can open (in browser) `http://localhost:3000/about` and we will see a new "about page".

But there is a problem, the styles are not applied. We can easily fix that but importing the same `tailwind` file as in the case of `index.svelte`. Let's add the following code:

```diff
+++ a/about.svelte
<!-- src/routes/about.svelte -->
+<script>
+ import '../styles/tailwind.css';
+</script>
+
<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				alt="About Page"
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
				src="https://images.punkapi.com/v2/4.png"
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">About</h1>
				<p class="leading-relaxed">
					Lorem ipsum dolor sit amet consectetur adipisicing elit. Quos obcaecati reiciendis ad
					corrupti deleniti ipsam ab quidem veritatis assumenda? Cum hic ullam nobis illo. Explicabo
					repellat quas voluptates harum velit!
				</p>
			</div>
		</div>
	</div>
</section>
```

Another step could be adding of Navigation bar, which enable the navigation between two pages. Let's copy the content of the `snippets/navbar.html` into `routes/about.svelte`.

> Note: When you are naming the routes, make sure you follow the all lower-case convention.

Before we start making more changes, let's create a folder called `components`, add `NavBar.svelte` and copy the content of `snippets/navbar.html` into it. The full code looks as the following:

```html
<!-- src/components/NavBar.svelte -->
<div class="flex min-h-screen">
	<div
		class="flex flex-wrap justify-between w-screen h-20 text-white bg-black md:flex-nowrap"
	>
		<div class="z-30 flex items-center h-full pl-3 space-x-3 bg-black">
			<p class="text-2xl text-">SvelteKit Demo</p>
		</div>
		<!-- Menu -->
		<div
			class="flex flex-col items-stretch w-screen text-xl text-center transform bg-black md:flex-row md:translate-y-0 md:space-x-5 md:items-center md:justify-end md:pr-3"
		>
			<a href="/" class="h-10 leading-10 border-b-2 border-dotted md:border-none"
				>Home</a
			>
			<a
				href="/about"
				class="h-10 leading-10 border-b-2 border-dotted md:border-none"
				>About</a
			>
		</div>
	</div>
</div>
```

Now, let's add the `components/NavBar.svelte` into `routes/index.svelte` as the following:

```diff
--- a/index.svelte
+++ b/index.svelte
<!-- src/routes/index.svelte -->
<script>
	import '../styles/tailwind.css';
+ import NavBar from '../components/NavBar.svelte';
</script>

+<NavBar />
<h1 class="text-4xl text-center my-8 uppercase">Welcome to SvelteKit</h1>
```

The `NavBar` component will appear on the main, but the problem occurs when you click on the `/about` page and as you probably expected, unless we re-import the component again, the other page won't contain that NavBar.

### Group layout into a shared file

The solution above works well, but there must be a better solution for avoiding such a repetition. Good news is that the solution exists and is called `__layout.svelte`.

Let's create that file with `routes` directory first. When you have a look at any page url from the project, everything is going to blank. The reason for that is simple. There is no object which allows us to render the content. Let's add a `<slot />` object first.

```diff
+++ a/__layout.svelte
<!-- src/routes/__layout.svelte -->
+<slot />
```

By adding the `<slot />`, you make the placeholder available for rendering the content of the routes. So everything we had implemented before within the routes works now as expected.

However, we can also add more content into the `__layout.svelte` and it appears onto every page. Let's make the following modifications:

```diff
--- a/__layout.svelte
+++ b/__layout.svelte
<!-- src/routes/__layout.svelte -->
+<script>
+ import '../styles/tailwind.css';
+	import NavBar from '../components/NavBar.svelte';
+</script>
-<slot />
+<div>
+	<NavBar />
+	<slot />
+</div>
```

When you save the file, you can see the `duplicated NavBar` on the page. This is a sign, the new code was applied and we can easily remove the code from `index.svelte` as the following:

```diff
--- a/index.svelte
<!-- src/routes/index.svelte -->
-<script>
-	import '../styles/tailwind.css';
-	import NavBar from '../components/NavBar.svelte';
-</script>
-
-<NavBar />
<h1 class="text-4xl text-center my-8 uppercase">Welcome to SvelteKit</h1>
```

### Changing the title of the page and using fragments

Svelte contains special elements that allow us to modify some extra information, like `title` of a page.

```diff
+++ a/index.svelte
+<svelte:head>
+ <title>Svelte Kit - Intro Page</title>
+</svelte:head>
<h1 class="text-4xl text-center my-8 uppercase">Welcome to SvelteKit</h1>
```

```diff
+++ a/about.svelte
<!-- src/routes/about.svelte -->
-<script>
-	import '../styles/tailwind.css';
-</script>
-
+<svelte:head>
+	<title>Svelte Kit - About Page</title>
+</svelte:head>

<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				alt="About Page"
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
				src="https://images.punkapi.com/v2/4.png"
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">About</h1>
				<p class="leading-relaxed">
					Lorem ipsum dolor sit amet consectetur adipisicing elit. Quos obcaecati reiciendis ad
					corrupti deleniti ipsam ab quidem veritatis assumenda? Cum hic ullam nobis illo. Explicabo
					repellat quas voluptates harum velit!
				</p>
			</div>
		</div>
	</div>
</section>
```

> Note: For the full lists of the special elements, please visit: [https://svelte.dev](https://svelte.dev/).

### Latest snapshots

The latest snapshot of the files looks like the code below:

```html
<!-- src/components/NavBar.svelte -->
<div class="flex">
	<div
		class="flex flex-wrap justify-between w-screen h-20 text-white bg-black md:flex-nowrap"
	>
		<div class="z-30 flex items-center h-full pl-3 space-x-3 bg-black">
			<p class="text-2xl text-">SvelteKit Demo</p>
		</div>
		<!-- Menu -->
		<div
			class="flex flex-col items-stretch w-screen text-xl text-center transform bg-black md:flex-row md:translate-y-0 md:space-x-5 md:items-center md:justify-end md:pr-3"
		>
			<a href="/" class="h-10 leading-10 border-b-2 border-dotted md:border-none"
				>Home</a
			>
			<a
				href="/about"
				class="h-10 leading-10 border-b-2 border-dotted md:border-none"
				>About</a
			>
		</div>
	</div>
</div>
```

```html
<!-- src/routes/__layout.svelte -->
<script>
	import '../styles/tailwind.css';
	import NavBar from '../components/NavBar.svelte';
</script>

<div>
	<NavBar />
	<slot />
</div>
```

```html
<!-- src/routes/about.svelte -->
<svelte:head>
	<title>Svelte Kit - About Page</title>
</svelte:head>

<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				alt="About Page"
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
				src="https://images.punkapi.com/v2/4.png"
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">About</h1>
				<p class="leading-relaxed">
					Lorem ipsum dolor sit amet consectetur adipisicing elit. Quos obcaecati
					reiciendis ad corrupti deleniti ipsam ab quidem veritatis assumenda? Cum
					hic ullam nobis illo. Explicabo repellat quas voluptates harum velit!
				</p>
			</div>
		</div>
	</div>
</section>
```

```html
<!-- src/routes/index.svelte -->
<svelte:head>
	<title>Svelte Kit - Intro Page</title>
</svelte:head>
<h1 class="text-4xl text-center my-8 uppercase">Welcome to SvelteKit</h1>
```

## Detail Page and basic interactivity

In this section, we are going to `add the detail page` and `update the layout of the main one` (index.svelte) to be able to make a basic interaction between the pages.

### Adding layout of the detail page

Creating a detail page is very straightforward. **The main complexity is making sure to cover the dynamic url**. To do that, let's `create a new folder` called `beer` within `routes` directory and add a new file named as `[id].svelte`.

As a next step, let's copy the content of the `detail.html` snippet into it.

```html
<!-- src/routes/beer/[id].svelte -->
<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
				<h2 class="text-sm title-font text-gray-500 tracking-widest">Tagline</h2>
				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">Name</h1>
				<p class="leading-relaxed">Description</p>
				<div class="flex mt-4">
					<button
						class="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
					>
						Back
					</button>
				</div>
			</div>
		</div>
	</div>
</section>
```

If you put any route prefixed as `beer` and containing some id (e.g. `/beer/1`), you will see the detail page, but without any meaningful content.

### Updating layout of the main one (index.svelte)

Before adding some more interactivity between index and detail pages, let's actually update the content of the index one.

Let's remove the existing `h1` placeholder and replace it by the content of the `index.html` from snippets directory as the following:

```diff
--- a/index.svelte
+++ b/index.svelte
<!-- src/routes/index.svelte -->
<svelte:head>
	<title>Svelte Kit - Intro Page</title>
</svelte:head>
-<h1 class="text-4xl text-center my-8 uppercase">Welcome to SvelteKit</h1>
+<div id="menu" class="container mx-auto px-4 lg:pt-24 lg:pb-64">
+	<div class="flex flex-wrap text-center justify-center">
+	  <div class="w-full lg:w-6/12 px-4">
+			<h2 class="text-4xl font-semibold text-black">Brewdog's Beer Selection</h2>
+			<p class="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
+				Let's enjoy Brewdog's selection of beer whilst developing a simple page with SvelteKit.
+			</p>
+			<div class="flex justify-center">
+				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
+					All Beers
+				</button>
+				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
+					Pilsner
+				</button>
+				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
+					Lager
+				</button>
+				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
+					Ale
+				</button>
+			</div>
+		</div>
+	</div>
+	<div class="flex flex-wrap mt-12 justify-center">
+		<div class="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
+			<div class="col-span-2 sm:col-span-1 xl:col-span-1">
+				<img alt="" src="" class="h-20 w-8 rounded  mx-auto" />
+			</div>
+			<div class="col-span-2 sm:col-span-4 xl:col-span-4">
+				<h3 class="font-semibold text-black">beer name</h3>
+				<p>Beer description</p>
+			</div>
+			<div class="col-span-2 sm:col-span-1 xl:col-span-1">
+				<a href="#" class="underline">Detail</a>
+			</div>
+		</div>
+	</div>
+</div>
```

### Making HTTP calls and finishing the basics of the interactivity

Now, as there is layout in place for `index.svelte` and `[id].svelte`, let's make some interactivity. Let's utilise the `http utils functions` and `svelte onMount lifecycle method` in both files.

```diff
--- a/index.svelte
+++ b/index.svelte
<!-- src/routes/index.svelte -->
+<script>
+	import { onMount } from 'svelte';
+	import { getBeerList } from '../utils/http';
+
+	let beers = [];
+
+	onMount(async () => {
+		beers = await getBeerList();
+	});
</script>
<svelte:head>
	<title>Svelte Kit - Intro Page</title>
</svelte:head>
<div id="menu" class="container mx-auto px-4 lg:pt-24 lg:pb-64">
	<div class="flex flex-wrap text-center justify-center">
		<div class="w-full lg:w-6/12 px-4">
			<h2 class="text-4xl font-semibold text-black">Brewdog's Beer Selection</h2>
			<p class="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
				Let's enjoy Brewdog's selection of beer whilst developing a simple page with SvelteKit.
			</p>
			<div class="flex justify-center">
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					All Beers
				</button>
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					Pilsner
				</button>
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					Lager
				</button>
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					Ale
				</button>
			</div>
		</div>
	</div>
	<div class="flex flex-wrap mt-12 justify-center">
		<div class="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
+     {#each beers as beer (beer.id)}
        <div class="col-span-2 sm:col-span-1 xl:col-span-1">
-         <img alt="" src="" class="h-20 w-8 rounded  mx-auto" />
+         <img alt={beer.title} src={beer.image_url} class="h-20 w-8 rounded  mx-auto" />
        </div>
        <div class="col-span-2 sm:col-span-4 xl:col-span-4">
-         <h3 class="font-semibold text-black">beer name</h3>
+         <h3 class="font-semibold text-black">{beer.name}</h3>
-         <p>Beer description</p>
+         <p>{beer.description.slice(0, beer.description.indexOf('.'))}.</p>
        </div>
        <div class="col-span-2 sm:col-span-1 xl:col-span-1">
-         <a href="#" class="underline">Detail</a>
+         <a href={`/beer/${beer.id}`} class="underline">Detail</a>
        </div>
+     {/each}
    </div>
	</div>
</div>
```

```diff
--- a/[id].svelte
+++ b/[id].svelte
<!-- src/routes/beer/[id].svelte -->
+<script>
+	import { page } from '$app/stores';
+	import { goto } from '$app/navigation';
+	import { onMount } from 'svelte';
+	import { getBeerById } from '../../utils/http';
+
+	let beer = {};
+
+	onMount(async () => {
+		beer = await getBeerById($page.params.id);
+	});
+
+	const handleClick = () => {
+		goto('/');
+	};
+</script>
+
<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
-			<img class="w-40 h-80 object-cover object-center rounded border border-gray-200" />
+     <img
+				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
+				alt={beer?.name}
+				src={beer?.image_url}
+			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
-				<h2 class="text-sm title-font text-gray-500 tracking-widest">Tagline</h2>
+       <h2 class="text-sm title-font text-gray-500 tracking-widest">{beer?.tagline}</h2>
-				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">Name</h1>
+       <h1 class="text-gray-900 text-3xl title-font font-medium mb-1">{beer?.name}</h1>
-				<p class="leading-relaxed">Description</p>
+       <p class="leading-relaxed">{beer?.description}</p>
				<div class="flex mt-4">
					<button
+           on:click={handleClick}
						class="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
					>
						Back
					</button>
				</div>
			</div>
		</div>
	</div>
</section>
```

> Note: For getting the route id, we needed to use a functionality from stores, including the subscription. In the next section, we are going to have a closer look on this functionality.

### Latest snapshots

The latest state of the files look as the following:

```html
<!-- src/routes/beer/[id].svelte -->
<script>
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
	import { getBeerById } from '../../utils/http';

	let beer = {};

	onMount(async () => {
		beer = await getBeerById($page.params.id);
	});

	const handleClick = () => {
		goto('/');
	};
</script>

<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
				alt="{beer?.name}"
				src="{beer?.image_url}"
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
				<h2 class="text-sm title-font text-gray-500 tracking-widest">
					{beer?.tagline}
				</h2>
				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">
					{beer?.name}
				</h1>
				<p class="leading-relaxed">{beer?.description}</p>
				<div class="flex mt-4">
					<button
						on:click="{handleClick}"
						class="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
					>
						Back
					</button>
				</div>
			</div>
		</div>
	</div>
</section>
```

```html
<!-- src/routes/index.svelte -->
<script>
	import { onMount } from 'svelte';
	import { getBeerList } from '../utils/http';

	let beers = [];

	onMount(async () => {
		beers = await getBeerList();
	});
</script>

<svelte:head>
	<title>Svelte Kit - Intro Page</title>
</svelte:head>
<div id="menu" class="container mx-auto px-4 lg:pt-24 lg:pb-64">
	<div class="flex flex-wrap text-center justify-center">
		<div class="w-full lg:w-6/12 px-4">
			<h2 class="text-4xl font-semibold text-black">Brewdog's Beer Selection</h2>
			<p class="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
				Let's enjoy Brewdog's selection of beer whilst developing a simple page with SvelteKit.
			</p>
			<div class="flex justify-center">
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					All Beers
				</button>
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					Pilsner
				</button>
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					Lager
				</button>
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					Ale
				</button>
			</div>
		</div>
	</div>
	<div class="flex flex-wrap mt-12 justify-center">
		<div class="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
			{#each beers as beer (beer.id)}
				<div class="col-span-2 sm:col-span-1 xl:col-span-1">
					<img alt={beer.title} src={beer.image_url} class="h-20 w-8 rounded  mx-auto" />
				</div>
				<div class="col-span-2 sm:col-span-4 xl:col-span-4">
					<h3 class="font-semibold text-black">{beer.name}</h3>
					<p>{beer.description.slice(0, beer.description.indexOf('.'))}.</p>
				</div>
				<div class="col-span-2 sm:col-span-1 xl:col-span-1">
					<a href={`/beer/${beer.id}`} class="underline">Detail</a>
				</div>
			{/each}
		</div>
	</div>
</div>
```

## Adding stores

We can slightly tweak the previous implementation by adding stores.

> A store is simply an object with a subscribe method that allows interested parties to be notified whenever the store value changes.

As the first step, let's create `stores` folder within `src` directory and add `beerstore.js` into it with the following content.

```javascript
// src/stores/beerstore.js
import { writable } from 'svelte/store';
import { getBeerList, getBeerById, getListOfBeersByName } from '../utils/http';

export const beers = writable([]);
export const beer = writable({});

export const fetchBeers = async (page, perPage = 25) => {
	beers.set(await getBeerList(page, perPage));
};

export const fetchListOfBeersByName = async (beerName = '') => {
	beers.set(await getListOfBeersByName(beerName));
};

export const fetchBeerById = async (id) => {
	beer.set(await getBeerById(id));
};
```

Now, we can update the `index.svelte` with the following changes:

```diff
--- a/index.svelte
+++ b/index.svelte
<!-- src/routes/index.svelte -->
<script>
	import { onMount } from 'svelte';
-	import { getBeerList } from '../utils/http';
+ import { fetchBeers, beers } from '../stores/beerstore';

-	let beers = [];

	onMount(async () => {
-		beers = await getBeerList();
+   await fetchBeers();
	});
</script>

<svelte:head>
	<title>Svelte Kit - Intro Page</title>
</svelte:head>
<div id="menu" class="container mx-auto px-4 lg:pt-24 lg:pb-64">
	<div class="flex flex-wrap text-center justify-center">
		<div class="w-full lg:w-6/12 px-4">
			<h2 class="text-4xl font-semibold text-black">Brewdog's Beer Selection</h2>
			<p class="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
				Let's enjoy Brewdog's selection of beer whilst developing a simple page with SvelteKit.
			</p>
			<div class="flex justify-center">
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					All Beers
				</button>
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					Pilsner
				</button>
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					Lager
				</button>
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					Ale
				</button>
			</div>
		</div>
	</div>
	<div class="flex flex-wrap mt-12 justify-center">
		<div class="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
-			{#each beers as beer (beer.id)}
+     {#each $beers as beer (beer.id)}
				<div class="col-span-2 sm:col-span-1 xl:col-span-1">
					<img alt={beer.title} src={beer.image_url} class="h-20 w-8 rounded  mx-auto" />
				</div>
				<div class="col-span-2 sm:col-span-4 xl:col-span-4">
					<h3 class="font-semibold text-black">{beer.name}</h3>
					<p>{beer.description.slice(0, beer.description.indexOf('.'))}.</p>
				</div>
				<div class="col-span-2 sm:col-span-1 xl:col-span-1">
					<a href={`/beer/${beer.id}`} class="underline">Detail</a>
				</div>
			{/each}
		</div>
	</div>
</div>
```

And we can do a similar thing in the `[id].svelte` as the following:

```diff
--- a/[id].svelte
+++ b/[id].svelte
<!-- src/routes/beer/[id].svelte -->
<script>
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
-	import { getBeerById } from '../../utils/http';
+ import { fetchBeerById, beer } from '../../stores/beerstore';

-	let beer = {};

	onMount(async () => {
-		beer = await getBeerById($page.params.id);
+   await fetchBeerById($page.params.id);
	});

	const handleClick = () => {
		goto('/');
	};
</script>

<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
-				alt={beer?.name}
+       alt={$beer?.name}
-				src={beer?.image_url}
+       src={$beer?.image_url}
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
-				<h2 class="text-sm title-font text-gray-500 tracking-widest">{beer?.tagline}</h2>
+       <h2 class="text-sm title-font text-gray-500 tracking-widest">{$beer?.tagline}</h2>
-				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">{beer?.name}</h1>
+       <h1 class="text-gray-900 text-3xl title-font font-medium mb-1">{$beer?.name}</h1>
-				<p class="leading-relaxed">{beer?.description}</p>
+       <p class="leading-relaxed">{$beer?.description}</p>
				<div class="flex mt-4">
					<button
						on:click={handleClick}
						class="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
					>
						Back
					</button>
				</div>
			</div>
		</div>
	</div>
</section>
```

### Latest snapshots

The latest state of the files look as the following:

```html
<!-- src/routes/beer/[id].svelte -->
<script>
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
	import { fetchBeerById, beer } from '../../stores/beerstore';

	onMount(async () => {
		await fetchBeerById($page.params.id);
	});

	const handleClick = () => {
		goto('/');
	};
</script>

<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
				alt="{$beer?.name}"
				src="{$beer?.image_url}"
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
				<h2 class="text-sm title-font text-gray-500 tracking-widest">
					{$beer?.tagline}
				</h2>
				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">
					{$beer?.name}
				</h1>
				<p class="leading-relaxed">{$beer?.description}</p>
				<div class="flex mt-4">
					<button
						on:click="{handleClick}"
						class="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
					>
						Back
					</button>
				</div>
			</div>
		</div>
	</div>
</section>
```

```html
<!-- src/routes/index.svelte -->
<script>
	import { onMount } from 'svelte';
	import { fetchBeers, beers } from '../stores/beerstore';

	onMount(async () => {
		await fetchBeers();
	});
</script>

<svelte:head>
	<title>Svelte Kit - Intro Page</title>
</svelte:head>
<div id="menu" class="container mx-auto px-4 lg:pt-24 lg:pb-64">
	<div class="flex flex-wrap text-center justify-center">
		<div class="w-full lg:w-6/12 px-4">
			<h2 class="text-4xl font-semibold text-black">Brewdog's Beer Selection</h2>
			<p class="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
				Let's enjoy Brewdog's selection of beer whilst developing a simple page with SvelteKit.
			</p>
			<div class="flex justify-center">
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					All Beers
				</button>
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					Pilsner
				</button>
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					Lager
				</button>
				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
					Ale
				</button>
			</div>
		</div>
	</div>
	<div class="flex flex-wrap mt-12 justify-center">
		<div class="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
			{#each $beers as beer (beer.id)}
				<div class="col-span-2 sm:col-span-1 xl:col-span-1">
					<img alt={beer.title} src={beer.image_url} class="h-20 w-8 rounded  mx-auto" />
				</div>
				<div class="col-span-2 sm:col-span-4 xl:col-span-4">
					<h3 class="font-semibold text-black">{beer.name}</h3>
					<p>{beer.description.slice(0, beer.description.indexOf('.'))}.</p>
				</div>
				<div class="col-span-2 sm:col-span-1 xl:col-span-1">
					<a href={`/beer/${beer.id}`} class="underline">Detail</a>
				</div>
			{/each}
		</div>
	</div>
</div>
```

## Reactivity

Svelte has a special construction that helps managing updates. There is the following construction `$:` that help observing changes. Let's see that in action in two occasions.

### Adding reactivity to `[id].svelte`

Let's remove the duplicity in the store subscription by providing reactivity construction.

```diff
--- a/[id].svelte
+++ b/[id].svelte
<!-- src/routes/beer/[id].svelte -->
<script>
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
-	import { fetchBeerById, beer } from '../../stores/beerstore';
+ import { fetchBeerById, beer as selectedBeer } from '../../stores/beerstore';

	onMount(async () => {
		await fetchBeerById($page.params.id);
	});

	const handleClick = () => {
		goto('/');
	};

+ let beer = {};
+
+ beer = $selectedBeer;
+
+ // let's run it commented first
+	// $: {
+	//	beer = $selectedBeer;
+	// }
</script>

<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
-				alt="{$beer?.name}"
+				alt="{beer?.name}"
-				src="{$beer?.image_url}"
+       src="{beer?.image_url}"
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
-				<h2 class="text-sm title-font text-gray-500 tracking-widest">{$beer?.tagline}</h2>
+       <h2 class="text-sm title-font text-gray-500 tracking-widest">{beer?.tagline}</h2>
-				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">{$beer?.name}</h1>
+       <h1 class="text-gray-900 text-3xl title-font font-medium mb-1">{beer?.name}</h1>
-				<p class="leading-relaxed">{$beer?.description}</p>
+       <p class="leading-relaxed">{beer?.description}</p>
				<div class="flex mt-4">
					<button
						on:click="{handleClick}"
						class="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
					>
						Back
					</button>
				</div>
			</div>
		</div>
	</div>
</section>
```

### Adding reactivity to `index.svelte`

Let's add a more complex example into the `index.svelte`.

```diff
--- a/index.svelte
+++ b/index.svelte
<!-- src/routes/index.svelte -->
<script>
	import { onMount } from 'svelte';
-	import { fetchBeers, beers } from '../stores/beerstore';
+ import { fetchBeers, fetchListOfBeersByName, beers } from '../stores/beerstore';

	onMount(async () => {
		await fetchBeers();
	});

+ let beerName = '';
+
+	const handleChange = (name = '') => {
+		beerName = name;
+	};
+
+	$: {
+		if (beerName.length > 0) {
+			fetchListOfBeersByName(beerName);
+		} else {
+			fetchBeers();
+		}
+	}
</script>

<svelte:head>
	<title>Svelte Kit - Intro Page</title>
</svelte:head>
<div id="menu" class="container mx-auto px-4 lg:pt-24 lg:pb-64">
	<div class="flex flex-wrap text-center justify-center">
		<div class="w-full lg:w-6/12 px-4">
			<h2 class="text-4xl font-semibold text-black">Brewdog's Beer Selection</h2>
			<p class="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
				Let's enjoy Brewdog's selection of beer whilst developing a simple page with SvelteKit.
			</p>
			<div class="flex justify-center">
-				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
+				<button
+					on:click={() => handleChange('')}
+					class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
+				>
					All Beers
				</button>
-				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
+				<button
+					on:click={() => handleChange('pilsner')}
+					class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
+				>
					Pilsner
				</button>
-				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
+				<button
+					on:click={() => handleChange('lager')}
+					class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
+				>
					Lager
				</button>
-				<button class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l">
+				<button
+					on:click={() => handleChange('ale')}
+					class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
+				>
					Ale
				</button>
			</div>
		</div>
	</div>
	<div class="flex flex-wrap mt-12 justify-center">
		<div class="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
			{#each $beers as beer (beer.id)}
				<div class="col-span-2 sm:col-span-1 xl:col-span-1">
					<img alt={beer.title} src={beer.image_url} class="h-20 w-8 rounded  mx-auto" />
				</div>
				<div class="col-span-2 sm:col-span-4 xl:col-span-4">
					<h3 class="font-semibold text-black">{beer.name}</h3>
					<p>{beer.description.slice(0, beer.description.indexOf('.'))}.</p>
				</div>
				<div class="col-span-2 sm:col-span-1 xl:col-span-1">
					<a href={`/beer/${beer.id}`} class="underline">Detail</a>
				</div>
			{/each}
		</div>
	</div>
</div>
```

### Latest snapshots

The latest state of the files look as the following:

```html
<!-- src/routes/beer/[id].svelte -->
<script>
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
	import { fetchBeerById, beer as selectedBeer } from '../../stores/beerstore';

	onMount(async () => {
		await fetchBeerById($page.params.id);
	});

	const handleClick = () => {
		goto('/');
	};

	let beer = {};

	beer = $selectedBeer;

	$: {
		beer = $selectedBeer;
	}
</script>

<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
				alt="{beer?.name}"
				src="{beer?.image_url}"
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
				<h2 class="text-sm title-font text-gray-500 tracking-widest">
					{beer?.tagline}
				</h2>
				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">
					{beer?.name}
				</h1>
				<p class="leading-relaxed">{beer?.description}</p>
				<div class="flex mt-4">
					<button
						on:click="{handleClick}"
						class="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
					>
						Back
					</button>
				</div>
			</div>
		</div>
	</div>
</section>
```

```html
<!-- src/routes/index.svelte -->
<script>
	import { onMount } from 'svelte';
	import { fetchBeers, fetchListOfBeersByName, beers } from '../stores/beerstore';

	onMount(async () => {
		await fetchBeers();
	});

	let beerName = '';

	const handleChange = (name = '') => {
		beerName = name;
	};

	$: {
		if (beerName.length > 0) {
			fetchListOfBeersByName(beerName);
		} else {
			fetchBeers();
		}
	}
</script>

<svelte:head>
	<title>Svelte Kit - Intro Page</title>
</svelte:head>
<div id="menu" class="container mx-auto px-4 lg:pt-24 lg:pb-64">
	<div class="flex flex-wrap text-center justify-center">
		<div class="w-full lg:w-6/12 px-4">
			<h2 class="text-4xl font-semibold text-black">Brewdog's Beer Selection</h2>
			<p class="text-lg leading-relaxed mt-4 mb-4 text-gray-500">
				Let's enjoy Brewdog's selection of beer whilst developing a simple page with SvelteKit.
			</p>
			<div class="flex justify-center">
				<button
					on:click={() => handleChange('')}
					class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
				>
					All Beers
				</button>
				<button
					on:click={() => handleChange('pilsner')}
					class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
				>
					Pilsner
				</button>
				<button
					on:click={() => handleChange('lager')}
					class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
				>
					Lager
				</button>
				<button
					on:click={() => handleChange('ale')}
					class="bg-blue-100 hover:underline text-gray-800 py-2 px-4 mr-2 rounded-l"
				>
					Ale
				</button>
			</div>
		</div>
	</div>
	<div class="flex flex-wrap mt-12 justify-center">
		<div class="grid grid-cols-1 sm:grid-cols-6 md:grid-cols-6 lg:grid-cols-6 xl:grid-cols-6 gap-4">
			{#each $beers as beer (beer.id)}
				<div class="col-span-2 sm:col-span-1 xl:col-span-1">
					<img alt={beer.title} src={beer.image_url} class="h-20 w-8 rounded  mx-auto" />
				</div>
				<div class="col-span-2 sm:col-span-4 xl:col-span-4">
					<h3 class="font-semibold text-black">{beer.name}</h3>
					<p>{beer.description.slice(0, beer.description.indexOf('.'))}.</p>
				</div>
				<div class="col-span-2 sm:col-span-1 xl:col-span-1">
					<a href={`/beer/${beer.id}`} class="underline">Detail</a>
				</div>
			{/each}
		</div>
	</div>
</div>
```

## Server-side rendering

In this section, we are going refactor our data fetching and introduce the **Server-side rendering** (SSR).

Svelte contains a specific directive related to scripts that make sure the code is run just once. You can just add the attribute `context` with value `module` and it does the trick.

SvelteKit utilizes the technique to execute the code which is going to be run in advance before the content is rendered. Let's have a look at that on the detail page now.

```diff
--- a/[id].svelte
+++ b/[id].svelte
<!-- src/routes/beer/[id].svelte -->
+<script context="module">
+	import { getBeerById } from '../../utils/http';
+	export async function load({ page }) {
+		const id = page.params.id;
+		const beer = await getBeerById(id);
+
+		return { props: { beer } };
+	}
+</script>

<script>
-	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
-	import { onMount } from 'svelte';
-	import { fetchBeerById, beer as selectedBeer } from '../../stores/beerstore';
-
-	onMount(async () => {
-		await fetchBeerById($page.params.id);
-	});

+ export let beer;

	const handleClick = () => {
		goto('/');
	};

-	let beer = {};

-	beer = $selectedBeer;

-	$: {
-		beer = $selectedBeer;
-	}
</script>

<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
				alt={beer?.name}
				src={beer?.image_url}
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
				<h2 class="text-sm title-font text-gray-500 tracking-widest">{beer?.tagline}</h2>
				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">{beer?.name}</h1>
				<p class="leading-relaxed">{beer?.description}</p>
				<div class="flex mt-4">
					<button
						on:click={handleClick}
						class="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
					>
						Back
					</button>
				</div>
			</div>
		</div>
	</div>
</section>
```

### Latest snapshot

The final version of the file looks as the following:

```html
<!-- src/routes/beer/[id].svelte -->
<script context="module">
	import { getBeerById } from '../../utils/http';
	export async function load({ page }) {
		const id = page.params.id;
		const beer = await getBeerById(id);

		return { props: { beer } };
	}
</script>

<script>
	import { goto } from '$app/navigation';
	export let beer;

	const handleClick = () => {
		goto('/');
	};
</script>

<section class="text-gray-700 body-font overflow-hidden bg-white">
	<div class="container px-5 py-24 mx-auto">
		<div class="lg:w-4/5 mx-auto flex flex-wrap">
			<img
				class="w-40 h-80 object-cover object-center rounded border border-gray-200"
				alt="{beer?.name}"
				src="{beer?.image_url}"
			/>
			<div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
				<h2 class="text-sm title-font text-gray-500 tracking-widest">
					{beer?.tagline}
				</h2>
				<h1 class="text-gray-900 text-3xl title-font font-medium mb-1">
					{beer?.name}
				</h1>
				<p class="leading-relaxed">{beer?.description}</p>
				<div class="flex mt-4">
					<button
						on:click="{handleClick}"
						class="flex text-white bg-red-500 border-0 py-2 px-6 focus:outline-none hover:bg-red-600 rounded"
					>
						Back
					</button>
				</div>
			</div>
		</div>
	</div>
</section>
```

## API Endpoints

The last thing we are going to discuss are API endpoints. In routes directory (`src/routes`), let's create a new folder called `api` and then add `beer` one into it.

As the next step, we can add two files into the `src/routes/api/beer` directory named as `index.js` and `[id].js` with the following content:

```javascript
// src/routes/api/beer/[id].js
import { getBeerById } from '../../../utils/http';

export async function get({ params }) {
	const beer = await getBeerById(params.id);

	return {
		status: 200,
		body: beer
	};
}
```

```javascript
// src/routes/api/beer/index.js
import { getBeerList } from '../../../utils/http';

export async function get({ params }) {
	const beers = await getBeerList();

	return {
		status: 200,
		body: beers
	};
}
```

When you open these two urls in the browser:

- http://localhost:3000/api/beer
- http://localhost:3000/api/beer/1

You should see the appropriate responses.

## Conclusion

Thanks for following this workshop material. Any feedback appreciated (radek.tomasek@gmail.com).
