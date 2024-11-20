# Intro to React Table

> Note: This workshop handles the legacy React Query. In the foreseeable future, I would like to migrate it to [Tanstack Table for React](https://tanstack.com/table/latest).

During our Frontend Group sessions, we had covered two libraries created by Tanner Linsley ([TanStack](https://tanstack.com/)) - **React Query** and **React Location**.

Today, we are going to have a look at [React Table](https://react-table.tanstack.com) which is a lightweight and extensible way how to handle data tables for React.

And without a further ado, let's get started.

## Project Initialization

We are going to initialize a very simple JS application with a JSON datafile that helps us to render the source of data for our table. We will start small, but increase the complexity over time.

The boilerplate of the app is going to be setup with [Create-MF-app](https://www.npmjs.com/package/create-mf-app) which is a very cool tool allowing us creating a simple prototype based on almost any JS framework, whilst choosing between JavaScript/TypeScript and CSS/Tailwind for building simple web apps, API servers or libraries.

As a first step let's initialize the project by typing the following:

```bash
npx create-mf-app
```

There are going to be a set of prompt questions. Let's set them as the following:

- **react-table-intro** (Name of the app)
- **Application** (Project type)
- **8080** (Port Number)
- **React** (Framework)
- **JavaScript** (Language)
- **Tailwind** (CSS)

Once the project is initialized, we need to install the modules. You can choose either `yarn` or `npm` , based on your preference. In this example, I stick to the latter.

Let's install the dependencies:

```
# if you prefer yarn, feel free to use it instead
cd react-table-intro
npm install
```

As a next step, we need to install `react-table`, `date-fns` and `json-loader`. It's a pretty straightforward operation, just type the following:

```bash
npm install react-table date-fns json-loader --save
```

Let's also add a simple script into the `index.html`

```diff
+++ a/index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>react-table-demo</title>
  </head>

  <body>
    <div id="app"></div>
+   <script src="https://unpkg.com/regenerator-runtime@0.13.1/runtime.js"></script>
  </body>
</html>
```

The fully updated file looks as the following:

```html
<!-- src/index.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>react-table-demo</title>
  </head>

  <body>
    <div id="app"></div>
    <script src="https://unpkg.com/regenerator-runtime@0.13.1/runtime.js"></script>
  </body>
</html>
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

We need to use the `json-loader` within our webpack app to be able to import JSON files. Let's make the following changes in the `webpack.config.js`

```diff
+++ a/webpack.config.js
const HtmlWebPackPlugin = require('html-webpack-plugin');
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

const deps = require('./package.json').dependencies;
module.exports = {
  output: {
    publicPath: 'http://localhost:8080/',
  },

  resolve: {
    extensions: ['.tsx', '.ts', '.jsx', '.js', '.json'],
  },

  devServer: {
    port: 8080,
    historyApiFallback: true,
  },

  module: {
    rules: [
      {
        test: /\.m?js/,
        type: 'javascript/auto',
        resolve: {
          fullySpecified: false,
        },
      },
      {
        test: /\.(css|s[ac]ss)$/i,
        use: ['style-loader', 'css-loader', 'postcss-loader'],
      },
+     {
+       test: /\.json$/,
+       use: {
+         loader: 'json-loader',
+       },
+     },
      {
        test: /\.(ts|tsx|js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
        },
      },
    ],
  },

  plugins: [
    new ModuleFederationPlugin({
      name: 'react_table_demo',
      filename: 'remoteEntry.js',
      remotes: {},
      exposes: {},
      shared: {
        ...deps,
        react: {
          singleton: true,
          requiredVersion: deps.react,
        },
        'react-dom': {
          singleton: true,
          requiredVersion: deps['react-dom'],
        },
      },
    }),
    new HtmlWebPackPlugin({
      template: './src/index.html',
    }),
  ],
};
```

The full updated version of the `webpack.config.js` looks as the following:

```javascript
const HtmlWebPackPlugin = require('html-webpack-plugin');
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

const deps = require('./package.json').dependencies;
module.exports = {
  output: {
    publicPath: 'http://localhost:8080/',
  },

  resolve: {
    extensions: ['.tsx', '.ts', '.jsx', '.js', '.json'],
  },

  devServer: {
    port: 8080,
    historyApiFallback: true,
  },

  module: {
    rules: [
      {
        test: /\.m?js/,
        type: 'javascript/auto',
        resolve: {
          fullySpecified: false,
        },
      },
      {
        test: /\.(css|s[ac]ss)$/i,
        use: ['style-loader', 'css-loader', 'postcss-loader'],
      },
      {
        test: /\.json$/,
        use: {
          loader: 'json-loader',
        },
      },
      {
        test: /\.(ts|tsx|js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
        },
      },
    ],
  },

  plugins: [
    new ModuleFederationPlugin({
      name: 'react_table_demo',
      filename: 'remoteEntry.js',
      remotes: {},
      exposes: {},
      shared: {
        ...deps,
        react: {
          singleton: true,
          requiredVersion: deps.react,
        },
        'react-dom': {
          singleton: true,
          requiredVersion: deps['react-dom'],
        },
      },
    }),
    new HtmlWebPackPlugin({
      template: './src/index.html',
    }),
  ],
};
```

In terms of app structure, we are not going to follow any specific pattern. Let's just create a `components` directory within `src` folder by typing:

```bash
mkdir src/components
```

We need to download some source of data to populate our table. When I was learning React Table following a youtube tutorial, [Mockaroo](https://www.mockaroo.com) looked as a really good data generator and we are going to use it as data source in this example as well.

Our table data source (using Mockaroo) is going to look as the following:

| Field         | Type          | Options                                                            |
| ------------- | ------------- | ------------------------------------------------------------------ |
| id            | Row Number    | Blank 0%                                                           |
| first_name    | First Name    | Blank 0%                                                           |
| last_name     | Last Name     | Blank 0%                                                           |
| email         | Email Address | Blank 0%                                                           |
| date_of_birth | Datetime      | from: 01/01/1970, to: 01/01/2020, format: ISO 8601 (UTC), Blank 0% |
| phone         | Phone         | format: +# ### ### ####, Blank 0%                                  |

We can generate some larger amount data to demonstrate capabilities such as pagination. Let's set the output as the following:

- **#rows** 1000
- **Format** JSON

Download the JSON file and place it into `src/components` folder.

Let's add a simple `SearchBar.jsx` component

```javascript
// src/components/SearchBar.jsx
import React from 'react';

export const SearchBar = () => {
  return (
    <div className="flex relative mx-auto mb-2">
      <input
        className="border-2 border-primary bg-red transition h-12 px-5 pr-16 rounded-md focus:outline-none w-full text-black text-lg "
        type="search"
        name="search"
        placeholder="Search"
      />
      <button type="submit" className="absolute right-2 top-3 mr-4">
        <svg
          className="text-black h-6 w-6 fill-current"
          version="1.1"
          id="Capa_1"
          x="0px"
          y="0px"
          viewBox="0 0 56.966 56.966"
          width="512px"
          height="512px"
        >
          <path d="M55.146,51.887L41.588,37.786c3.486-4.144,5.396-9.358,5.396-14.786c0-12.682-10.318-23-23-23s-23,10.318-23,23  s10.318,23,23,23c4.761,0,9.298-1.436,13.177-4.162l13.661,14.208c0.571,0.593,1.339,0.92,2.162,0.92  c0.779,0,1.518-0.297,2.079-0.837C56.255,54.982,56.293,53.08,55.146,51.887z M23.984,6c9.374,0,17,7.626,17,17s-7.626,17-17,17  s-17-7.626-17-17S14.61,6,23.984,6z" />
        </svg>
      </button>
    </div>
  );
};
```

We will also need a simple input component for further search capabilities. Let's create a file called `LocalInput.jsx` with the following content:

```javascript
// src/components/LocalInput.jsx
import React from 'react';

export const LocalInput = () => {
  return (
    <div className="flex flex-col">
      <div>
        <label className="input-field inline-flex items-baseline border-none shadow-md bg-white p-2">
          <div className="flex-1 leading-none">
            <input
              id="handle"
              type="text"
              className="placeholder-blue w-full p-0 no-outline text-dusty-blue-darker"
              name="handle"
              placeholder="filter"
            />
          </div>
        </label>
      </div>
    </div>
  );
};
```

Another step is to create a simple table component. We can just add static hardcoded data for now, that help us establishing the layout. Let's add a new file called `TableComponent.jsx` with the code below.

```javascript
// src/components/TableComponent.jsx
import React from 'react';

export const TableComponent = () => {
  return (
    <table className="items-center bg-transparent w-full border-collapse">
      <thead>
        <tr>
          <th className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left">
            #id
          </th>
          <th className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left">
            First Name
          </th>
          <th className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left">
            Last Name
          </th>
          <th className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left">
            Email
          </th>
          <th className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left">
            Date of birth
          </th>
        </tr>
      </thead>

      <tbody>
        <tr>
          <td className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700 ">
            1
          </td>
          <td className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 ">
            Salem
          </td>
          <td className="border-t-0 px-6 align-center border-l-0 border-r-0 text-xs whitespace-nowrap p-4">
            Paffot
          </td>
          <td className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4">
            spaffot0@plala.or.jp
          </td>
          <td className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4">
            1975-06-09T07:59:05Z
          </td>
        </tr>
      </tbody>
    </table>
  );
};
```

And add the component into the `App.jsx` as the following:

```diff
--- a/App.jsx
+++ b/App.jsx
import React from 'react';
import ReactDOM from 'react-dom';
+import { TableComponent } from './components/TableComponent';

import './index.scss';

const App = () => (
  <div className="mt-10 text-3xl mx-auto max-w-6xl">
-   <div>Name: react-table-demo</div>
-   <div>Framework: react</div>
-   <div>Language: JavaScript</div>
-   <div>CSS: Tailwind</div>
+   <TableComponent />
  </div>
);
ReactDOM.render(<App />, document.getElementById('app'));
```

The latest version of the `App.jsx` should looks as the following sample:

```javascript
// src/App.jsx
import React from 'react';
import ReactDOM from 'react-dom';
import { TableComponent } from './components/TableComponent';

import './index.scss';

const App = () => (
  <div className="mt-10 text-3xl mx-auto max-w-6xl">
    <TableComponent />
  </div>
);
ReactDOM.render(<App />, document.getElementById('app'));
```

If you type `npm run start` in the terminal and pass `http://localhost:8080` into the browser, you should be able to see the a simple static table with one row as a result.

## Creating Basic Table

In this section, we are going to initialize the React Table. The process is relatively straightforward, let's see it in action now. In components folder, let's create a new file called `columns.js` with the following content:

```javascript
// src/components/columns.js
export const COLUMNS = [
  {
    Header: 'Id',
    accessor: 'id',
  },
  {
    Header: 'First Name',
    accessor: 'first_name',
  },
  {
    Header: 'Last Name',
    accessor: 'last_name',
  },
  {
    Header: 'Email',
    accessor: 'email',
  },
  {
    Header: 'Date of Birth',
    accessor: 'date_of_birth',
  },
];
```

> Note: We skipped **phone** field. In our case it was the last field, but we can technically skipped any field from the input data object.

In the `TableComponent.jsx` we need to add the boilerplate for rendering the table. Let's make the following changes:

```diff
--- a/App.jsx
+++ b/App.jsx
// src/components/TableComponent.jsx
import React from 'react';
+import { useTable } from 'react-table';
+import { COLUMNS } from './columns';
+import MOCK_DATA from './MOCK_DATA.json';

+ const { getTableProps, getTableBodyProps, headerGroups, rows, prepareRow } =
+   useTable({
+     columns,
+     data,
+   });

export const TableComponent = () => {
  return (
-   <table className="items-center bg-transparent w-full border-collapse">
+   <table className="items-center bg-transparent w-full border-collapse" {...getTableProps()}>
      <thead>
-       <tr>
-         <th className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left">
-           #id
-         </th>
-         <th className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left">
-           First Name
-         </th>
-         <th className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left">
-           Last Name
-         </th>
-         <th className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left">
-           Email
-         </th>
-         <th className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left">
-           Date of birth
-         </th>
-       </tr>
+       {headerGroups.map((headerGroup) => (
+         <tr {...headerGroup.getHeaderGroupProps()}>
+           {headerGroup.headers.map((column) => (
+             <th
+               {...column.getHeaderProps()}
+               className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
+             >
+               {column.render('Header')}
+             </th>
+           ))}
+         </tr>
+       ))}
      </thead>
-     <tbody>
+     <tbody {...getTableBodyProps()}>
-       <tr>
-         <td className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700 ">
-           1
-         </td>
-         <td className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 ">
-           Salem
-         </td>
-         <td className="border-t-0 px-6 align-center border-l-0 border-r-0 text-xs whitespace-nowrap p-4">
-           Paffot
-         </td>
-         <td className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4">
-           spaffot0@plala.or.jp
-         </td>
-         <td className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4">
-           1975-06-09T07:59:05Z
-         </td>
-       </tr>
+       {rows.map((row) => {
+         prepareRow(row);
+         return (
+           <tr {...row.getRowProps()}>
+             {row.cells.map((cell) => {
+               return (
+                 <td
+                   {...cell.getCellProps()}
+                   className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
+                 >
+                   {cell.render('Cell')}
+                 </td>
+               );
+             })}
+           </tr>
+         );
+       })}
      </tbody>
    </table>
  );
};
```

The full version of the updated file looks as the following:

```javascript
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import { useTable } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const { getTableProps, getTableBodyProps, headerGroups, rows, prepareRow } =
    useTable({
      columns,
      data,
    });

  return (
    <table
      className="items-center bg-transparent w-full border-collapse"
      {...getTableProps()}
    >
      <thead>
        {headerGroups.map((headerGroup) => (
          <tr {...headerGroup.getHeaderGroupProps()}>
            {headerGroup.headers.map((column) => (
              <th
                {...column.getHeaderProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Header')}
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody {...getTableBodyProps()}>
        {rows.map((row) => {
          prepareRow(row);
          return (
            <tr {...row.getRowProps()}>
              {row.cells.map((cell) => {
                return (
                  <td
                    {...cell.getCellProps()}
                    className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                  >
                    {cell.render('Cell')}
                  </td>
                );
              })}
            </tr>
          );
        })}
      </tbody>
    </table>
  );
};
```

> Note: The **memo** is used for prevention of the re-rendering the table constantly. Any data update would cause an un-necessary re-render.

The changes above help to render the full table with the desired columns. It was quite a work to add all of the 
boilerplate, however, most of other tasks are going to be pretty straightforward.

## Adding missing column

The next step we are going to do is adding/re-ordering a column. We can easily extend the `columns.js` file and make the changes as the following:

```diff
+++ a/columns.js
// src/components/columns.js

export const COLUMNS = [
  {
    Header: 'Id',
    accessor: 'id',
  },
  {
    Header: 'First Name',
    accessor: 'first_name',
  },
  {
    Header: 'Last Name',
    accessor: 'last_name',
  },
  {
    Header: 'Email',
    accessor: 'email',
  },
+ {
+   Header: 'Phone',
+   accessor: 'phone',
+ },
  {
    Header: 'Date of Birth',
    accessor: 'date_of_birth',
  },
];
```

The full output looks as the following:

```javascript
// src/components/columns.js

export const COLUMNS = [
  {
    Header: 'Id',
    accessor: 'id',
  },
  {
    Header: 'First Name',
    accessor: 'first_name',
  },
  {
    Header: 'Last Name',
    accessor: 'last_name',
  },
  {
    Header: 'Email',
    accessor: 'email',
  },
  {
    Header: 'Phone',
    accessor: 'phone',
  },
  {
    Header: 'Date of Birth',
    accessor: 'date_of_birth',
  },
];
```

It doesn't matter which order we choose. The table is rendered correctly with a very small change.

## Adding footer

Another very easy change to make is adding a footer. As the first step, we need to update our **COLUMNS** array in the `columns.js` by adding a new field.

```diff
--- a/columns.js
+++ b/columns.js
// src/components/columns.js

export const COLUMNS = [
  {
    Header: 'Id',
+   Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'First Name',
+   Footer: 'First Name',
    accessor: 'first_name',
  },
  {
    Header: 'Last Name',
+   Footer: 'Last Name',
    accessor: 'last_name',
  },
  {
    Header: 'Email',
+   Footer: 'Email',
    accessor: 'email',
  },
  {
    Header: 'Phone',
+   Footer: 'Phone',
    accessor: 'phone',
  },
  {
    Header: 'Date of Birth',
+   Footer: 'Date of Birth',
    accessor: 'date_of_birth',
  },
];
```

Once we extend the definition of the **COLUMNS** array, we need to update the **TableComponent** by adding the following:

```diff
--- a/TableComponent.jsx
+++ b/TableComponent.jsx
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import { useTable } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

- const { getTableProps, getTableBodyProps, headerGroups, rows, prepareRow } =
-   useTable({
-     columns,
-     data,
-   });
+ const {
+   getTableProps,
+   getTableBodyProps,
+   headerGroups,
+   footerGroups,
+   rows,
+   prepareRow,
+ } = useTable({
+   columns,
+   data,
+ });

  return (
    <table
      className="items-center bg-transparent w-full border-collapse"
      {...getTableProps()}
    >
      <thead>
        {headerGroups.map((headerGroup) => (
          <tr {...headerGroup.getHeaderGroupProps()}>
            {headerGroup.headers.map((column) => (
              <th
                {...column.getHeaderProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Header')}
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody {...getTableBodyProps()}>
        {rows.map((row) => {
          prepareRow(row);
          return (
            <tr {...row.getRowProps()}>
              {row.cells.map((cell) => {
                return (
                  <td
                    {...cell.getCellProps()}
                    className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                  >
                    {cell.render('Cell')}
                  </td>
                );
              })}
            </tr>
          );
        })}
      </tbody>
+     <tfoot>
+       {footerGroups.map((footerGroup) => (
+         <tr {...footerGroup.getFooterGroupProps()}>
+           {footerGroup.headers.map((column) => (
+             <td
+               {...column.getFooterProps()}
+               className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
+             >
+               {column.render('Footer')}
+             </td>
+           ))}
+         </tr>
+       ))}
+     </tfoot>
    </table>
  );
};
```

The latest snapshot of both files look as the following:

```javascript
// src/components/columns.js

export const COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'First Name',
    Footer: 'First Name',
    accessor: 'first_name',
  },
  {
    Header: 'Last Name',
    Footer: 'Last Name',
    accessor: 'last_name',
  },
  {
    Header: 'Email',
    Footer: 'Email',
    accessor: 'email',
  },
  {
    Header: 'Phone',
    Footer: 'Phone',
    accessor: 'phone',
  },
  {
    Header: 'Date of Birth',
    Footer: 'Date of Birth',
    accessor: 'date_of_birth',
  },
];
```

```javascript
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import { useTable } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
  } = useTable({
    columns,
    data,
  });

  return (
    <table
      className="items-center bg-transparent w-full border-collapse"
      {...getTableProps()}
    >
      <thead>
        {headerGroups.map((headerGroup) => (
          <tr {...headerGroup.getHeaderGroupProps()}>
            {headerGroup.headers.map((column) => (
              <th
                {...column.getHeaderProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Header')}
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody {...getTableBodyProps()}>
        {rows.map((row) => {
          prepareRow(row);
          return (
            <tr {...row.getRowProps()}>
              {row.cells.map((cell) => {
                return (
                  <td
                    {...cell.getCellProps()}
                    className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                  >
                    {cell.render('Cell')}
                  </td>
                );
              })}
            </tr>
          );
        })}
      </tbody>
      <tfoot>
        {footerGroups.map((footerGroup) => (
          <tr {...footerGroup.getFooterGroupProps()}>
            {footerGroup.headers.map((column) => (
              <td
                {...column.getFooterProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Footer')}
              </td>
            ))}
          </tr>
        ))}
      </tfoot>
    </table>
  );
};
```

Now, we can refresh the page, scroll down and see the footer.

## Column grouping

Another thing we can achieve easily is putting multiple columns into groups. Let's edit the `columns.js` and add the following content:

```diff
+++ a/columns.js
// src/components/columns.js

export const COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'First Name',
    Footer: 'First Name',
    accessor: 'first_name',
  },
  {
    Header: 'Last Name',
    Footer: 'Last Name',
    accessor: 'last_name',
  },
  {
    Header: 'Email',
    Footer: 'Email',
    accessor: 'email',
  },
  {
    Header: 'Phone',
    Footer: 'Phone',
    accessor: 'phone',
  },
  {
    Header: 'Date of Birth',
    Footer: 'Date of Birth',
    accessor: 'date_of_birth',
  },
];

+export const GROUPED_COLUMNS = [
+ {
+   Header: 'Id',
+   Footer: 'Id',
+   accessor: 'id',
+ },
+ {
+   Header: 'Name',
+   Footer: 'Name',
+   columns: [
+     {
+       Header: 'First Name',
+       Footer: 'First Name',
+       accessor: 'first_name',
+     },
+     {
+       Header: 'Last Name',
+       Footer: 'Last Name',
+       accessor: 'last_name',
+     },
+  ],
+ },
+ {
+   Header: 'Info',
+   Footer: 'Info',
+   columns: [
+     {
+       Header: 'Email',
+       Footer: 'Email',
+       accessor: 'email',
+     },
+     {
+       Header: 'Phone',
+       Footer: 'Phone',
+       accessor: 'phone',
+     },
+     {
+       Header: 'Date of Birth',
+       Footer: 'Date of Birth',
+       accessor: 'date_of_birth',
+     },
+   ],
+ },
];
```

In `TableComponent.jsx`, we need to replace the original data source by our new one as shown below.

```diff
--- a/TableComponent.jsx
+++ b/TableComponent.jsx
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import { useTable } from 'react-table';
-import { COLUMNS } from './columns';
+import { COLUMNS, GROUPED_COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';

export const TableComponent = () => {
- const columns = useMemo(() => COLUMNS, []);
+ const columns = useMemo(() => GROUPED_COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
  } = useTable({
    columns,
    data,
  });

  return (
    <table
      className="items-center bg-transparent w-full border-collapse"
      {...getTableProps()}
    >
      <thead>
        {headerGroups.map((headerGroup) => (
          <tr {...headerGroup.getHeaderGroupProps()}>
            {headerGroup.headers.map((column) => (
              <th
                {...column.getHeaderProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Header')}
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody {...getTableBodyProps()}>
        {rows.map((row) => {
          prepareRow(row);
          return (
            <tr {...row.getRowProps()}>
              {row.cells.map((cell) => {
                return (
                  <td
                    {...cell.getCellProps()}
                    className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                  >
                    {cell.render('Cell')}
                  </td>
                );
              })}
            </tr>
          );
        })}
      </tbody>
      <tfoot>
        {footerGroups.map((footerGroup) => (
          <tr {...footerGroup.getFooterGroupProps()}>
            {footerGroup.headers.map((column) => (
              <td
                {...column.getFooterProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Footer')}
              </td>
            ))}
          </tr>
        ))}
      </tfoot>
    </table>
  );
};
```

The latest snapshots look as the following:

```javascript
// src/components/columns.js

export const COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'First Name',
    Footer: 'First Name',
    accessor: 'first_name',
  },
  {
    Header: 'Last Name',
    Footer: 'Last Name',
    accessor: 'last_name',
  },
  {
    Header: 'Email',
    Footer: 'Email',
    accessor: 'email',
  },
  {
    Header: 'Phone',
    Footer: 'Phone',
    accessor: 'phone',
  },
  {
    Header: 'Date of Birth',
    Footer: 'Date of Birth',
    accessor: 'date_of_birth',
  },
];

export const GROUPED_COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'Name',
    Footer: 'Name',
    columns: [
      {
        Header: 'First Name',
        Footer: 'First Name',
        accessor: 'first_name',
      },
      {
        Header: 'Last Name',
        Footer: 'Last Name',
        accessor: 'last_name',
      },
    ],
  },
  {
    Header: 'Info',
    Footer: 'Info',
    columns: [
      {
        Header: 'Email',
        Footer: 'Email',
        accessor: 'email',
      },
      {
        Header: 'Phone',
        Footer: 'Phone',
        accessor: 'phone',
      },
      {
        Header: 'Date of Birth',
        Footer: 'Date of Birth',
        accessor: 'date_of_birth',
      },
    ],
  },
];
```

```javascript
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import { useTable } from 'react-table';
import { COLUMNS, GROUPED_COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';

export const TableComponent = () => {
  const columns = useMemo(() => GROUPED_COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
  } = useTable({
    columns,
    data,
  });

  return (
    <table
      className="items-center bg-transparent w-full border-collapse"
      {...getTableProps()}
    >
      <thead>
        {headerGroups.map((headerGroup) => (
          <tr {...headerGroup.getHeaderGroupProps()}>
            {headerGroup.headers.map((column) => (
              <th
                {...column.getHeaderProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Header')}
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody {...getTableBodyProps()}>
        {rows.map((row) => {
          prepareRow(row);
          return (
            <tr {...row.getRowProps()}>
              {row.cells.map((cell) => {
                return (
                  <td
                    {...cell.getCellProps()}
                    className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                  >
                    {cell.render('Cell')}
                  </td>
                );
              })}
            </tr>
          );
        })}
      </tbody>
      <tfoot>
        {footerGroups.map((footerGroup) => (
          <tr {...footerGroup.getFooterGroupProps()}>
            {footerGroup.headers.map((column) => (
              <td
                {...column.getFooterProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Footer')}
              </td>
            ))}
          </tr>
        ))}
      </tfoot>
    </table>
  );
};
```

## Formatting Column

Let's make some cell formatting. We can customize the output in any way we want. In our case, **we focus on the date format**. Before we do that, let's revert our group changes in `TableComponent.jsx` as the following:

```diff
--- a/TableComponent.jsx
+++ b/TableComponent.jsx
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import { useTable } from 'react-table';
-import { COLUMNS, GROUPED_COLUMNS } from './columns';
+import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';

export const TableComponent = () => {
- const columns = useMemo(() => GROUPED_COLUMNS, []);
+ const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
  } = useTable({
    columns,
    data,
  });

  return (
    <table
      className="items-center bg-transparent w-full border-collapse"
      {...getTableProps()}
    >
      <thead>
        {headerGroups.map((headerGroup) => (
          <tr {...headerGroup.getHeaderGroupProps()}>
            {headerGroup.headers.map((column) => (
              <th
                {...column.getHeaderProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Header')}
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody {...getTableBodyProps()}>
        {rows.map((row) => {
          prepareRow(row);
          return (
            <tr {...row.getRowProps()}>
              {row.cells.map((cell) => {
                return (
                  <td
                    {...cell.getCellProps()}
                    className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                  >
                    {cell.render('Cell')}
                  </td>
                );
              })}
            </tr>
          );
        })}
      </tbody>
      <tfoot>
        {footerGroups.map((footerGroup) => (
          <tr {...footerGroup.getFooterGroupProps()}>
            {footerGroup.headers.map((column) => (
              <td
                {...column.getFooterProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Footer')}
              </td>
            ))}
          </tr>
        ))}
      </tfoot>
    </table>
  );
};
```

Another step is adding `Cell` field into our `COLUMNS` array within `columns.js` file and assign a callback function that handle our formatting. Let's see it in the example below.

```diff
+++ a/colums.js
// src/components/columns.js
+import { format } from 'date-fns';

export const COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'First Name',
    Footer: 'First Name',
    accessor: 'first_name',
  },
  {
    Header: 'Last Name',
    Footer: 'Last Name',
    accessor: 'last_name',
  },
  {
    Header: 'Email',
    Footer: 'Email',
    accessor: 'email',
  },
  {
    Header: 'Phone',
    Footer: 'Phone',
    accessor: 'phone',

  },
  {
    Header: 'Date of Birth',
    Footer: 'Date of Birth',
    accessor: 'date_of_birth',
+   Cell: ({ value }) => {
+     return format(new Date(value), 'yyyy-MM-dd');
+   },
  },
];

export const GROUPED_COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'Name',
    Footer: 'Name',
    columns: [
      {
        Header: 'First Name',
        Footer: 'First Name',
        accessor: 'first_name',
      },
      {
        Header: 'Last Name',
        Footer: 'Last Name',
        accessor: 'last_name',
      },
    ],
  },
  {
    Header: 'Info',
    Footer: 'Info',
    columns: [
      {
        Header: 'Email',
        Footer: 'Email',
        accessor: 'email',
      },
      {
        Header: 'Phone',
        Footer: 'Phone',
        accessor: 'phone',
      },
      {
        Header: 'Date of Birth',
        Footer: 'Date of Birth',
        accessor: 'date_of_birth',
      },
    ],
  },
];
```

The latest snapshots look as the following:

```javascript
// src/components/columns.js
import { format } from 'date-fns';

export const COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'First Name',
    Footer: 'First Name',
    accessor: 'first_name',
  },
  {
    Header: 'Last Name',
    Footer: 'Last Name',
    accessor: 'last_name',
  },
  {
    Header: 'Email',
    Footer: 'Email',
    accessor: 'email',
  },
  {
    Header: 'Phone',
    Footer: 'Phone',
    accessor: 'phone',
  },
  {
    Header: 'Date of Birth',
    Footer: 'Date of Birth',
    accessor: 'date_of_birth',
    Cell: ({ value }) => {
      return format(new Date(value), 'yyyy-MM-dd');
    },
  },
];

export const GROUPED_COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'Name',
    Footer: 'Name',
    columns: [
      {
        Header: 'First Name',
        Footer: 'First Name',
        accessor: 'first_name',
      },
      {
        Header: 'Last Name',
        Footer: 'Last Name',
        accessor: 'last_name',
      },
    ],
  },
  {
    Header: 'Info',
    Footer: 'Info',
    columns: [
      {
        Header: 'Email',
        Footer: 'Email',
        accessor: 'email',
      },
      {
        Header: 'Phone',
        Footer: 'Phone',
        accessor: 'phone',
      },
      {
        Header: 'Date of Birth',
        Footer: 'Date of Birth',
        accessor: 'date_of_birth',
      },
    ],
  },
];
```

```javascript
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import { useTable } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
  } = useTable({
    columns,
    data,
  });

  return (
    <table
      className="items-center bg-transparent w-full border-collapse"
      {...getTableProps()}
    >
      <thead>
        {headerGroups.map((headerGroup) => (
          <tr {...headerGroup.getHeaderGroupProps()}>
            {headerGroup.headers.map((column) => (
              <th
                {...column.getHeaderProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Header')}
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody {...getTableBodyProps()}>
        {rows.map((row) => {
          prepareRow(row);
          return (
            <tr {...row.getRowProps()}>
              {row.cells.map((cell) => {
                return (
                  <td
                    {...cell.getCellProps()}
                    className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                  >
                    {cell.render('Cell')}
                  </td>
                );
              })}
            </tr>
          );
        })}
      </tbody>
      <tfoot>
        {footerGroups.map((footerGroup) => (
          <tr {...footerGroup.getFooterGroupProps()}>
            {footerGroup.headers.map((column) => (
              <td
                {...column.getFooterProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Footer')}
              </td>
            ))}
          </tr>
        ))}
      </tfoot>
    </table>
  );
};
```

## Sorting

This is another area that is very easy to implement and bring benefits to the user. Let's add new code into the `TableComponent.jsx` as the following:

```diff
--- a/TableComponent.jsx
+++ b/TableComponent.jsx
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
-import { useTable } from 'react-table';
+import { useTable, useSortBy } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
  } = useTable({
    columns,
    data,
- });
+ },
+ useSortBy
+);

  return (
    <table
      className="items-center bg-transparent w-full border-collapse"
      {...getTableProps()}
    >
      <thead>
        {headerGroups.map((headerGroup) => (
          <tr {...headerGroup.getHeaderGroupProps()}>
            {headerGroup.headers.map((column) => (
              <th
-               {...column.getHeaderProps()}
+               {...column.getHeaderProps(column.getSortByToggleProps())}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Header')}
+               <span>
+                 {column.isSorted ? (column.isSortedDesc ? '⬇' : '⬆') : ''}
+               </span>
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody {...getTableBodyProps()}>
        {rows.map((row) => {
          prepareRow(row);
          return (
            <tr {...row.getRowProps()}>
              {row.cells.map((cell) => {
                return (
                  <td
                    {...cell.getCellProps()}
                    className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                  >
                    {cell.render('Cell')}
                  </td>
                );
              })}
            </tr>
          );
        })}
      </tbody>
      <tfoot>
        {footerGroups.map((footerGroup) => (
          <tr {...footerGroup.getFooterGroupProps()}>
            {footerGroup.headers.map((column) => (
              <td
                {...column.getFooterProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Footer')}
              </td>
            ))}
          </tr>
        ))}
      </tfoot>
    </table>
  );
};
```

There is no need to make any change in the `columns.js`. The latest snapshot looks as the following:

```javascript
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import { useTable, useSortBy } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
  } = useTable(
    {
      columns,
      data,
    },
    useSortBy
  );

  return (
    <table
      className="items-center bg-transparent w-full border-collapse"
      {...getTableProps()}
    >
      <thead>
        {headerGroups.map((headerGroup) => (
          <tr {...headerGroup.getHeaderGroupProps()}>
            {headerGroup.headers.map((column) => (
              <th
                {...column.getHeaderProps(column.getSortByToggleProps())}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Header')}
                <span>
                  {column.isSorted ? (column.isSortedDesc ? '⬇' : '⬆') : ''}
                </span>
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody {...getTableBodyProps()}>
        {rows.map((row) => {
          prepareRow(row);
          return (
            <tr {...row.getRowProps()}>
              {row.cells.map((cell) => {
                return (
                  <td
                    {...cell.getCellProps()}
                    className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                  >
                    {cell.render('Cell')}
                  </td>
                );
              })}
            </tr>
          );
        })}
      </tbody>
      <tfoot>
        {footerGroups.map((footerGroup) => (
          <tr {...footerGroup.getFooterGroupProps()}>
            {footerGroup.headers.map((column) => (
              <td
                {...column.getFooterProps()}
                className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
              >
                {column.render('Footer')}
              </td>
            ))}
          </tr>
        ))}
      </tfoot>
    </table>
  );
};
```

## Global Filter

Global Filter is another useful piece of functionality and usual, very easy to implement. Let's update our `SearchBar.jsx` component first.

```diff
--- a/SearchBar.jsx
+++ b/SearchBar.jsx
// src/components/SearchBar.jsx
-import React from 'react';
+import React, { useState } from 'react';

-export const SearchBar = () => {
+export const SearchBar = ({ filter, setFilter }) => {
  return (
    <div className="flex relative mx-auto mb-2">
      <input
        className="border-2 border-primary bg-red transition h-12 px-5 pr-16 rounded-md focus:outline-none w-full text-black text-lg "
        type="search"
        name="search"
        placeholder="Search"
+       value={filter || ''}
+       onChange={(e) => setFilter(e.target.value)}
      />
      <button type="submit" className="absolute right-2 top-3 mr-4">
        <svg
          className="text-black h-6 w-6 fill-current"
          version="1.1"
          id="Capa_1"
          x="0px"
          y="0px"
          viewBox="0 0 56.966 56.966"
          width="512px"
          height="512px"
        >
          <path d="M55.146,51.887L41.588,37.786c3.486-4.144,5.396-9.358,5.396-14.786c0-12.682-10.318-23-23-23s-23,10.318-23,23  s10.318,23,23,23c4.761,0,9.298-1.436,13.177-4.162l13.661,14.208c0.571,0.593,1.339,0.92,2.162,0.92  c0.779,0,1.518-0.297,2.079-0.837C56.255,54.982,56.293,53.08,55.146,51.887z M23.984,6c9.374,0,17,7.626,17,17s-7.626,17-17,17  s-17-7.626-17-17S14.61,6,23.984,6z" />
        </svg>
      </button>
    </div>
  );
};
```

Now we need to include the change in the `TableComponent.jsx` component. Let's do it as the following:

```diff
--- a/TableComponent.jsx
+++ b/TableComponent.jsx
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
-import { useTable, useSortBy } from 'react-table';
+import { useTable, useSortBy, useGlobalFilter } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';
+import { SearchBar } from './SearchBar';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
+   state,
+   setGlobalFilter,
  } = useTable(
    {
      columns,
      data,
    },
+   useGlobalFilter,
    useSortBy
  );

  return (
+   <>
+     <SearchBar filter={state.globalFilter} setFilter={setGlobalFilter} />
      <table
        className="items-center bg-transparent w-full border-collapse"
        {...getTableProps()}
      >
        <thead>
          {headerGroups.map((headerGroup) => (
            <tr {...headerGroup.getHeaderGroupProps()}>
              {headerGroup.headers.map((column) => (
                <th
                  {...column.getHeaderProps(column.getSortByToggleProps())}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Header')}
                  <span>
                    {column.isSorted ? (column.isSortedDesc ? '⬇' : '⬆') : ''}
                  </span>
                </th>
              ))}
            </tr>
          ))}
        </thead>
        <tbody {...getTableBodyProps()}>
          {rows.map((row) => {
            prepareRow(row);
            return (
              <tr {...row.getRowProps()}>
                {row.cells.map((cell) => {
                  return (
                    <td
                      {...cell.getCellProps()}
                      className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                    >
                      {cell.render('Cell')}
                    </td>
                  );
                })}
              </tr>
            );
          })}
        </tbody>
        <tfoot>
          {footerGroups.map((footerGroup) => (
            <tr {...footerGroup.getFooterGroupProps()}>
              {footerGroup.headers.map((column) => (
                <td
                  {...column.getFooterProps()}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Footer')}
                </td>
              ))}
            </tr>
          ))}
        </tfoot>
      </table>
+   </>
  );
};
```

In case we need to use some debounce mechanism during our typing, we can rely on some built-in functionality as shown in the example below:

```diff
--- a/SearchBar.jsx
+++ b/SearchBar.jsx
// src/components/SearchBar.jsx
import React, { useState } from 'react';
+import { useAsyncDebounce } from 'react-table';

export const SearchBar = ({ filter, setFilter }) => {
+ const [value, setValue] = useState(filter);
+ const onChange = useAsyncDebounce((value) => {
+   setFilter(value || undefined);
+ }, 1000);

  return (
    <div className="flex relative mx-auto mb-2">
      <input
        className="border-2 border-primary bg-red transition h-12 px-5 pr-16 rounded-md focus:outline-none w-full text-black text-lg "
        type="search"
        name="search"
        placeholder="Search"
-       value={filter || ''}
-       onChange={(e) => setFilter(e.target.value)}
+       value={value || ''}
+       onChange={(e) => {
+         setValue(e.target.value);
+         onChange(e.target.value);
+       }}
      />
      <button type="submit" className="absolute right-2 top-3 mr-4">
        <svg
          className="text-black h-6 w-6 fill-current"
          version="1.1"
          id="Capa_1"
          x="0px"
          y="0px"
          viewBox="0 0 56.966 56.966"
          width="512px"
          height="512px"
        >
          <path d="M55.146,51.887L41.588,37.786c3.486-4.144,5.396-9.358,5.396-14.786c0-12.682-10.318-23-23-23s-23,10.318-23,23  s10.318,23,23,23c4.761,0,9.298-1.436,13.177-4.162l13.661,14.208c0.571,0.593,1.339,0.92,2.162,0.92  c0.779,0,1.518-0.297,2.079-0.837C56.255,54.982,56.293,53.08,55.146,51.887z M23.984,6c9.374,0,17,7.626,17,17s-7.626,17-17,17  s-17-7.626-17-17S14.61,6,23.984,6z" />
        </svg>
      </button>
    </div>
  );
};
```

Now we are able to use the search in a debounced way.

The latest snapshots look as the following:

```javascript
// src/components/SearchBar.jsx
import React, { useState } from 'react';
import { useAsyncDebounce } from 'react-table';

export const SearchBar = ({ filter, setFilter }) => {
  const [value, setValue] = useState(filter);

  const onChange = useAsyncDebounce((value) => {
    setFilter(value || undefined);
  }, 1000);

  return (
    <div className="flex relative mx-auto mb-2">
      <input
        className="border-2 border-primary bg-red transition h-12 px-5 pr-16 rounded-md focus:outline-none w-full text-black text-lg "
        type="search"
        name="search"
        placeholder="Search"
        value={value || ''}
        onChange={(e) => {
          setValue(e.target.value);
          onChange(e.target.value);
        }}
      />
      <button type="submit" className="absolute right-2 top-3 mr-4">
        <svg
          className="text-black h-6 w-6 fill-current"
          version="1.1"
          id="Capa_1"
          x="0px"
          y="0px"
          viewBox="0 0 56.966 56.966"
          width="512px"
          height="512px"
        >
          <path d="M55.146,51.887L41.588,37.786c3.486-4.144,5.396-9.358,5.396-14.786c0-12.682-10.318-23-23-23s-23,10.318-23,23  s10.318,23,23,23c4.761,0,9.298-1.436,13.177-4.162l13.661,14.208c0.571,0.593,1.339,0.92,2.162,0.92  c0.779,0,1.518-0.297,2.079-0.837C56.255,54.982,56.293,53.08,55.146,51.887z M23.984,6c9.374,0,17,7.626,17,17s-7.626,17-17,17  s-17-7.626-17-17S14.61,6,23.984,6z" />
        </svg>
      </button>
    </div>
  );
};
```

```javascript
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import { useTable, useSortBy, useGlobalFilter } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';
import { SearchBar } from './SearchBar';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
    state,
    setGlobalFilter,
  } = useTable(
    {
      columns,
      data,
    },
    useGlobalFilter,
    useSortBy
  );

  return (
    <>
      <SearchBar filter={state.globalFilter} setFilter={setGlobalFilter} />
      <table
        className="items-center bg-transparent w-full border-collapse"
        {...getTableProps()}
      >
        <thead>
          {headerGroups.map((headerGroup) => (
            <tr {...headerGroup.getHeaderGroupProps()}>
              {headerGroup.headers.map((column) => (
                <th
                  {...column.getHeaderProps(column.getSortByToggleProps())}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Header')}
                  <span>
                    {column.isSorted ? (column.isSortedDesc ? '⬇' : '⬆') : ''}
                  </span>
                </th>
              ))}
            </tr>
          ))}
        </thead>
        <tbody {...getTableBodyProps()}>
          {rows.map((row) => {
            prepareRow(row);
            return (
              <tr {...row.getRowProps()}>
                {row.cells.map((cell) => {
                  return (
                    <td
                      {...cell.getCellProps()}
                      className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                    >
                      {cell.render('Cell')}
                    </td>
                  );
                })}
              </tr>
            );
          })}
        </tbody>
        <tfoot>
          {footerGroups.map((footerGroup) => (
            <tr {...footerGroup.getFooterGroupProps()}>
              {footerGroup.headers.map((column) => (
                <td
                  {...column.getFooterProps()}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Footer')}
                </td>
              ))}
            </tr>
          ))}
        </tfoot>
      </table>
    </>
  );
};
```

## Local Filter

Alongside a global filter, there is another option for a local one.

```diff
--- a/LocalInput.jsx
+++ b/LocalInput.jsx
// src/components/LocalInput.jsx
import React from 'react';

-export const LocalInput = () => {
+export const LocalInput = ({ column }) => {
+ const { filterValue, setFilter } = column;
  return (
    <div className="flex flex-col">
      <div>
        <label className="input-field inline-flex items-baseline border-none shadow-md bg-white p-2">
          <div className="flex-1 leading-none">
            <input
              id="handle"
              type="text"
              className="placeholder-blue w-full p-0 no-outline text-dusty-blue-darker"
              name="handle"
              placeholder="filter"
+             value={filterValue || ''}
+             onChange={(e) => setFilter(e.target.value)}
            />
          </div>
        </label>
      </div>
    </div>
  );
};
```

Another step is updating `columns.js` by adding the following content:

```diff
--- a/columns.jsx
+++ b/columns.jsx
// src/components/columns.js
import { format } from 'date-fns';
+import { LocalInput } from './LocalInput';

export const COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
+   Filter: LocalInput,
  },
  {
    Header: 'First Name',
    Footer: 'First Name',
    accessor: 'first_name',
+   Filter: LocalInput,
  },
  {
    Header: 'Last Name',
    Footer: 'Last Name',
    accessor: 'last_name',
+   Filter: LocalInput,
  },
  {
    Header: 'Email',
    Footer: 'Email',
    accessor: 'email',
+   Filter: LocalInput,
  },
  {
    Header: 'Phone',
    Footer: 'Phone',
    accessor: 'phone',
+   Filter: LocalInput,
  },
  {
    Header: 'Date of Birth',
    Footer: 'Date of Birth',
    accessor: 'date_of_birth',
+   Filter: LocalInput,
    Cell: ({ value }) => {
      return format(new Date(value), 'yyyy-MM-dd');
    },
  },
];

export const GROUPED_COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'Name',
    Footer: 'Name',
    columns: [
      {
        Header: 'First Name',
        Footer: 'First Name',
        accessor: 'first_name',
      },
      {
        Header: 'Last Name',
        Footer: 'Last Name',
        accessor: 'last_name',
      },
    ],
  },
  {
    Header: 'Info',
    Footer: 'Info',
    columns: [
      {
        Header: 'Email',
        Footer: 'Email',
        accessor: 'email',
      },
      {
        Header: 'Phone',
        Footer: 'Phone',
        accessor: 'phone',
      },
      {
        Header: 'Date of Birth',
        Footer: 'Date of Birth',
        accessor: 'date_of_birth',
      },
    ],
  },
];
```

In the `TableComponents.jsx`, we need to make the following changes:

```diff
--- a/TableComponents.jsx
+++ b/TableComponents.jsx
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
-import { useTable, useSortBy, useGlobalFilter } from 'react-table';
+import { useTable, useSortBy, useGlobalFilter, useFilters } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';
import { SearchBar } from './SearchBar';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
    state,
    setGlobalFilter,
  } = useTable(
    {
      columns,
      data,
    },
+   useFilters,
    useGlobalFilter,
    useSortBy
  );

  return (
    <>
      <SearchBar filter={state.globalFilter} setFilter={setGlobalFilter} />
      <table
        className="items-center bg-transparent w-full border-collapse"
        {...getTableProps()}
      >
        <thead>
          {headerGroups.map((headerGroup) => (
            <tr {...headerGroup.getHeaderGroupProps()}>
              {headerGroup.headers.map((column) => (
                <th
                  {...column.getHeaderProps(column.getSortByToggleProps())}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Header')}
                  <span>
                    {column.isSorted ? (column.isSortedDesc ? '⬇' : '⬆') : ''}
                  </span>
+                 <div>{column.canFilter ? column.render('Filter') : null}</div>
                </th>
              ))}
            </tr>
          ))}
        </thead>
        <tbody {...getTableBodyProps()}>
          {rows.map((row) => {
            prepareRow(row);
            return (
              <tr {...row.getRowProps()}>
                {row.cells.map((cell) => {
                  return (
                    <td
                      {...cell.getCellProps()}
                      className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                    >
                      {cell.render('Cell')}
                    </td>
                  );
                })}
              </tr>
            );
          })}
        </tbody>
        <tfoot>
          {footerGroups.map((footerGroup) => (
            <tr {...footerGroup.getFooterGroupProps()}>
              {footerGroup.headers.map((column) => (
                <td
                  {...column.getFooterProps()}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Footer')}
                </td>
              ))}
            </tr>
          ))}
        </tfoot>
      </table>
    </>
  );
};
```

There might be a situation when we don't want to include a filter for certain column (e.g. id). There is a special flag called `disableFilters: true` that does that. Let's add it into our **columns.js** as the following:

```diff
+++ a/columns.js
// src/components/columns.js
import { format } from 'date-fns';
import { LocalInput } from './LocalInput';

export const COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
    Filter: LocalInput,
+   disableFilters: true,
  },
  {
    Header: 'First Name',
    Footer: 'First Name',
    accessor: 'first_name',
    Filter: LocalInput,
  },
  {
    Header: 'Last Name',
    Footer: 'Last Name',
    accessor: 'last_name',
    Filter: LocalInput,
  },
  {
    Header: 'Email',
    Footer: 'Email',
    accessor: 'email',
    Filter: LocalInput,
  },
  {
    Header: 'Phone',
    Footer: 'Phone',
    accessor: 'phone',
    Filter: LocalInput,
  },
  {
    Header: 'Date of Birth',
    Footer: 'Date of Birth',
    accessor: 'date_of_birth',
    Filter: LocalInput,
    Cell: ({ value }) => {
      return format(new Date(value), 'yyyy-MM-dd');
    },
  },
];

export const GROUPED_COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'Name',
    Footer: 'Name',
    columns: [
      {
        Header: 'First Name',
        Footer: 'First Name',
        accessor: 'first_name',
      },
      {
        Header: 'Last Name',
        Footer: 'Last Name',
        accessor: 'last_name',
      },
    ],
  },
  {
    Header: 'Info',
    Footer: 'Info',
    columns: [
      {
        Header: 'Email',
        Footer: 'Email',
        accessor: 'email',
      },
      {
        Header: 'Phone',
        Footer: 'Phone',
        accessor: 'phone',
      },
      {
        Header: 'Date of Birth',
        Footer: 'Date of Birth',
        accessor: 'date_of_birth',
      },
    ],
  },
];
```

What we can also do is to outsource some of our `columns.js` options into the `TableComponent.jsx` as the following:

```diff
--- a/TableComponents.jsx
+++ b/TableComponents.jsx
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import { useTable, useSortBy, useGlobalFilter, useFilters } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';
import { SearchBar } from './SearchBar';
+import { LocalInput } from './LocalInput';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

+ const defaultColumn = useMemo(() => {
+   return {
+     Filter: LocalInput,
+   };
+ }, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
    state,
    setGlobalFilter,
  } = useTable(
    {
      columns,
      data,
+     defaultColumn
    },
    useFilters,
    useGlobalFilter,
    useSortBy
  );

  return (
    <>
      <SearchBar filter={state.globalFilter} setFilter={setGlobalFilter} />
      <table
        className="items-center bg-transparent w-full border-collapse"
        {...getTableProps()}
      >
        <thead>
          {headerGroups.map((headerGroup) => (
            <tr {...headerGroup.getHeaderGroupProps()}>
              {headerGroup.headers.map((column) => (
                <th
                  {...column.getHeaderProps(column.getSortByToggleProps())}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Header')}
                  <span>
                    {column.isSorted ? (column.isSortedDesc ? '⬇' : '⬆') : ''}
                  </span>
                  <div>{column.canFilter ? column.render('Filter') : null}</div>
                </th>
              ))}
            </tr>
          ))}
        </thead>
        <tbody {...getTableBodyProps()}>
          {rows.map((row) => {
            prepareRow(row);
            return (
              <tr {...row.getRowProps()}>
                {row.cells.map((cell) => {
                  return (
                    <td
                      {...cell.getCellProps()}
                      className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                    >
                      {cell.render('Cell')}
                    </td>
                  );
                })}
              </tr>
            );
          })}
        </tbody>
        <tfoot>
          {footerGroups.map((footerGroup) => (
            <tr {...footerGroup.getFooterGroupProps()}>
              {footerGroup.headers.map((column) => (
                <td
                  {...column.getFooterProps()}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Footer')}
                </td>
              ))}
            </tr>
          ))}
        </tfoot>
      </table>
    </>
  );
};
```

What we can do now is removing the `Filter` option from the `columns.js` and everything should work as expected.

```diff
--- a/columns.js
+++ b/columns.js
// src/components/columns.js
import { format } from 'date-fns';
-import { LocalInput } from './LocalInput';

export const COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
-   Filter: LocalInput,
    disableFilters: true,
  },
  {
    Header: 'First Name',
    Footer: 'First Name',
    accessor: 'first_name',
-   Filter: LocalInput,
  },
  {
    Header: 'Last Name',
    Footer: 'Last Name',
    accessor: 'last_name',
-   Filter: LocalInput,
  },
  {
    Header: 'Email',
    Footer: 'Email',
    accessor: 'email',
-   Filter: LocalInput,
  },
  {
    Header: 'Phone',
    Footer: 'Phone',
    accessor: 'phone',
-   Filter: LocalInput,
  },
  {
    Header: 'Date of Birth',
    Footer: 'Date of Birth',
    accessor: 'date_of_birth',
-   Filter: LocalInput,
    Cell: ({ value }) => {
      return format(new Date(value), 'yyyy-MM-dd');
    },
  },
];

export const GROUPED_COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'Name',
    Footer: 'Name',
    columns: [
      {
        Header: 'First Name',
        Footer: 'First Name',
        accessor: 'first_name',
      },
      {
        Header: 'Last Name',
        Footer: 'Last Name',
        accessor: 'last_name',
      },
    ],
  },
  {
    Header: 'Info',
    Footer: 'Info',
    columns: [
      {
        Header: 'Email',
        Footer: 'Email',
        accessor: 'email',
      },
      {
        Header: 'Phone',
        Footer: 'Phone',
        accessor: 'phone',
      },
      {
        Header: 'Date of Birth',
        Footer: 'Date of Birth',
        accessor: 'date_of_birth',
      },
    ],
  },
];
```

The latest snapshots look as the following:

```javascript
// src/components/columns.js
import { format } from 'date-fns';
import { LocalInput } from './LocalInput';

export const COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
    disableFilters: true,
  },
  {
    Header: 'First Name',
    Footer: 'First Name',
    accessor: 'first_name',
  },
  {
    Header: 'Last Name',
    Footer: 'Last Name',
    accessor: 'last_name',
  },
  {
    Header: 'Email',
    Footer: 'Email',
    accessor: 'email',
  },
  {
    Header: 'Phone',
    Footer: 'Phone',
    accessor: 'phone',
  },
  {
    Header: 'Date of Birth',
    Footer: 'Date of Birth',
    accessor: 'date_of_birth',
    Cell: ({ value }) => {
      return format(new Date(value), 'yyyy-MM-dd');
    },
  },
];

export const GROUPED_COLUMNS = [
  {
    Header: 'Id',
    Footer: 'Id',
    accessor: 'id',
  },
  {
    Header: 'Name',
    Footer: 'Name',
    columns: [
      {
        Header: 'First Name',
        Footer: 'First Name',
        accessor: 'first_name',
      },
      {
        Header: 'Last Name',
        Footer: 'Last Name',
        accessor: 'last_name',
      },
    ],
  },
  {
    Header: 'Info',
    Footer: 'Info',
    columns: [
      {
        Header: 'Email',
        Footer: 'Email',
        accessor: 'email',
      },
      {
        Header: 'Phone',
        Footer: 'Phone',
        accessor: 'phone',
      },
      {
        Header: 'Date of Birth',
        Footer: 'Date of Birth',
        accessor: 'date_of_birth',
      },
    ],
  },
];
```

```javascript
// src/components/LocalInput.jsx
import React from 'react';

export const LocalInput = ({ column }) => {
  const { filterValue, setFilter } = column;
  return (
    <div className="flex flex-col">
      <div>
        <label className="input-field inline-flex items-baseline border-none shadow-md bg-white p-2">
          <div className="flex-1 leading-none">
            <input
              id="handle"
              type="text"
              className="placeholder-blue w-full p-0 no-outline text-dusty-blue-darker"
              name="handle"
              placeholder="filter"
              value={filterValue || ''}
              onChange={(e) => setFilter(e.target.value)}
            />
          </div>
        </label>
      </div>
    </div>
  );
};
```

```javascript
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import { useTable, useSortBy, useGlobalFilter, useFilters } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';
import { SearchBar } from './SearchBar';
import { LocalInput } from './LocalInput';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const defaultColumn = useMemo(() => {
    return {
      Filter: LocalInput,
    };
  }, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
    state,
    setGlobalFilter,
  } = useTable(
    {
      columns,
      data,
      defaultColumn,
    },
    useFilters,
    useGlobalFilter,
    useSortBy
  );

  return (
    <>
      <SearchBar filter={state.globalFilter} setFilter={setGlobalFilter} />
      <table
        className="items-center bg-transparent w-full border-collapse"
        {...getTableProps()}
      >
        <thead>
          {headerGroups.map((headerGroup) => (
            <tr {...headerGroup.getHeaderGroupProps()}>
              {headerGroup.headers.map((column) => (
                <th
                  {...column.getHeaderProps(column.getSortByToggleProps())}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Header')}
                  <span>
                    {column.isSorted ? (column.isSortedDesc ? '⬇' : '⬆') : ''}
                  </span>
                  <div>{column.canFilter ? column.render('Filter') : null}</div>
                </th>
              ))}
            </tr>
          ))}
        </thead>
        <tbody {...getTableBodyProps()}>
          {rows.map((row) => {
            prepareRow(row);
            return (
              <tr {...row.getRowProps()}>
                {row.cells.map((cell) => {
                  return (
                    <td
                      {...cell.getCellProps()}
                      className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                    >
                      {cell.render('Cell')}
                    </td>
                  );
                })}
              </tr>
            );
          })}
        </tbody>
        <tfoot>
          {footerGroups.map((footerGroup) => (
            <tr {...footerGroup.getFooterGroupProps()}>
              {footerGroup.headers.map((column) => (
                <td
                  {...column.getFooterProps()}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Footer')}
                </td>
              ))}
            </tr>
          ))}
        </tfoot>
      </table>
    </>
  );
};
```

## Pagination

We currently have a lot of rows. We can incorporate the pagination to manage the table in a better way.

First of all, let's consider this JSX snippet as the pagination boilerplate.

```html
<div
  className="px-5 py-5 bg-white border-t flex flex-col xs:flex-row items-center xs:justify-between"
>
  <span className="text-xs xs:text-sm text-gray-900">
    Showing 1 to 4 of 50 Entries
  </span>
  <div className="inline-flex mt-2 xs:mt-0">
    <button
      className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-l"
    >
      {'<<'}
    </button>
    <button
      className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-l"
    >
      Prev
    </button>
    <button
      className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-r"
    >
      Next
    </button>
    <button
      className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-r"
    >
      {'>>'}
    </button>
  </div>
</div>
```

Let's add it to the `TableComponent.jsx` as the following:

```diff
--- a/TableComponents.jsx
+++ b/TableComponents.jsx
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import { useTable, useSortBy, useGlobalFilter, useFilters } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';
import { SearchBar } from './SearchBar';
import { LocalInput } from './LocalInput';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const defaultColumn = useMemo(() => {
    return {
      Filter: LocalInput,
    };
  }, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
    state,
    setGlobalFilter,
  } = useTable(
    {
      columns,
      data,
      defaultColumn,
    },
    useFilters,
    useGlobalFilter,
    useSortBy
  );

  return (
    <>
      <SearchBar filter={state.globalFilter} setFilter={setGlobalFilter} />
+     <div
+       className="px-5 py-5 bg-white border-t flex flex-col xs:flex-row items-center xs:justify-between"
+     >
+       <span className="text-xs xs:text-sm text-gray-900">
+         Showing 1 to 4 of 50 Entries
+       </span>
+       <div className="inline-flex mt-2 xs:mt-0">
+         <button
+           className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-l"
+         >
+           {'<<'}
+         </button>
+         <button
+           className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-l"
+         >
+           Prev
+         </button>
+         <button
+           className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-r"
+         >
+           Next
+         </button>
+         <button
+           className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-r"
+         >
+           {'>>'}
+         </button>
+       </div>
+     </div>
      <table
        className="items-center bg-transparent w-full border-collapse"
        {...getTableProps()}
      >
        <thead>
          {headerGroups.map((headerGroup) => (
            <tr {...headerGroup.getHeaderGroupProps()}>
              {headerGroup.headers.map((column) => (
                <th
                  {...column.getHeaderProps(column.getSortByToggleProps())}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Header')}
                  <span>
                    {column.isSorted ? (column.isSortedDesc ? '⬇' : '⬆') : ''}
                  </span>
                  <div>{column.canFilter ? column.render('Filter') : null}</div>
                </th>
              ))}
            </tr>
          ))}
        </thead>
        <tbody {...getTableBodyProps()}>
          {rows.map((row) => {
            prepareRow(row);
            return (
              <tr {...row.getRowProps()}>
                {row.cells.map((cell) => {
                  return (
                    <td
                      {...cell.getCellProps()}
                      className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                    >
                      {cell.render('Cell')}
                    </td>
                  );
                })}
              </tr>
            );
          })}
        </tbody>
        <tfoot>
          {footerGroups.map((footerGroup) => (
            <tr {...footerGroup.getFooterGroupProps()}>
              {footerGroup.headers.map((column) => (
                <td
                  {...column.getFooterProps()}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Footer')}
                </td>
              ))}
            </tr>
          ))}
        </tfoot>
      </table>
    </>
  );
};
```

The next step is to incorporate the pagination elements as the following:

```diff
--- a/TableComponents.jsx
+++ b/TableComponents.jsx
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
-import { useTable, useSortBy, useGlobalFilter, useFilters } from 'react-table';
+import { useTable, useSortBy, useGlobalFilter, useFilters, usePagination } from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';
import { SearchBar } from './SearchBar';
import { LocalInput } from './LocalInput';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const defaultColumn = useMemo(() => {
    return {
      Filter: LocalInput,
    };
  }, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
    state,
    setGlobalFilter,
+   gotoPage,
+   pageCount,
+   setPageSize,
+   page,
+   nextPage,
+   previousPage,
+   canNextPage,
+   canPreviousPage,
+   pageOptions,
  } = useTable(
    {
      columns,
      data,
      defaultColumn,
+     initialState: { pageIndex: 0 },
    },
    useFilters,
    useGlobalFilter,
    useSortBy,
+   usePagination
  );

+ React.useEffect(() => {
+   setPageSize(20);
+ }, []);

  return (
    <>
      <SearchBar filter={state.globalFilter} setFilter={setGlobalFilter} />

      <div className="px-5 py-5 bg-white border-t flex flex-col xs:flex-row items-center xs:justify-between">
        <span className="text-xs xs:text-sm text-gray-900">
-         Showing 1 to 4 of 50 Entries
+         Showing {state.pageIndex + 1} to {state.pageSize} of {pageOptions.length} Entries
        </span>
        <div className="inline-flex mt-2 xs:mt-0">
-         <button className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-l">
+         <button
+           onClick={() => gotoPage(0)}
+           disabled={!canPreviousPage}
+           className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-l"
+         >
            {'<<'}
          </button>
-         <button className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-l">
+         <button
+           onClick={() => previousPage()}
+           disabled={!canPreviousPage}
+           className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-l"
+          >
            Prev
          </button>
-         <button className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-r">
+         <button
+           onClick={() => nextPage()}
+           disabled={!canNextPage}
+           className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-r"
+         >
            Next
          </button>
-         <button className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-r">
+         <button
+           onClick={() => gotoPage(pageCount - 1)}
+           disabled={!canNextPage}
+           className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-r"
+         >
            {'>>'}
          </button>
        </div>
      </div>
      <table
        className="items-center bg-transparent w-full border-collapse"
        {...getTableProps()}
      >
        <thead>
          {headerGroups.map((headerGroup) => (
            <tr {...headerGroup.getHeaderGroupProps()}>
              {headerGroup.headers.map((column) => (
                <th
                  {...column.getHeaderProps(column.getSortByToggleProps())}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Header')}
                  <span>
                    {column.isSorted ? (column.isSortedDesc ? '⬇' : '⬆') : ''}
                  </span>
                  <div>{column.canFilter ? column.render('Filter') : null}</div>
                </th>
              ))}
            </tr>
          ))}
        </thead>
        <tbody {...getTableBodyProps()}>
-         {rows.map((row) => {
+         {page.map((row) => {
            prepareRow(row);
            return (
              <tr {...row.getRowProps()}>
                {row.cells.map((cell) => {
                  return (
                    <td
                      {...cell.getCellProps()}
                      className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                    >
                      {cell.render('Cell')}
                    </td>
                  );
                })}
              </tr>
            );
          })}
        </tbody>
        <tfoot>
          {footerGroups.map((footerGroup) => (
            <tr {...footerGroup.getFooterGroupProps()}>
              {footerGroup.headers.map((column) => (
                <td
                  {...column.getFooterProps()}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Footer')}
                </td>
              ))}
            </tr>
          ))}
        </tfoot>
      </table>
    </>
  );
};
```

The latest snapshot looks as the following:

```javascript
// src/components/TableComponent.jsx
import React, { useMemo } from 'react';
import {
  useTable,
  useSortBy,
  useGlobalFilter,
  useFilters,
  usePagination,
} from 'react-table';
import { COLUMNS } from './columns';
import MOCK_DATA from './MOCK_DATA.json';
import { SearchBar } from './SearchBar';
import { LocalInput } from './LocalInput';

export const TableComponent = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  const defaultColumn = useMemo(() => {
    return {
      Filter: LocalInput,
    };
  }, []);

  const {
    getTableProps,
    getTableBodyProps,
    headerGroups,
    footerGroups,
    rows,
    prepareRow,
    state,
    setGlobalFilter,
    gotoPage,
    pageCount,
    setPageSize,
    page,
    nextPage,
    previousPage,
    canNextPage,
    canPreviousPage,
    pageOptions,
  } = useTable(
    {
      columns,
      data,
      defaultColumn,
      initialState: { pageIndex: 0 },
    },
    useFilters,
    useGlobalFilter,
    useSortBy,
    usePagination
  );

  React.useEffect(() => {
    setPageSize(20);
  }, []);

  return (
    <>
      <SearchBar filter={state.globalFilter} setFilter={setGlobalFilter} />

      <div className="px-5 py-5 bg-white border-t flex flex-col xs:flex-row items-center xs:justify-between">
        <span className="text-xs xs:text-sm text-gray-900">
          Showing {state.pageIndex + 1} to {state.pageSize} of{' '}
          {pageOptions.length} Entries
        </span>
        <div className="inline-flex mt-2 xs:mt-0">
          <button
            onClick={() => gotoPage(0)}
            disabled={!canPreviousPage}
            className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-l"
          >
            {'<<'}
          </button>
          <button
            onClick={() => previousPage()}
            disabled={!canPreviousPage}
            className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-l"
          >
            Prev
          </button>
          <button
            onClick={() => nextPage()}
            disabled={!canNextPage}
            className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-r"
          >
            Next
          </button>
          <button
            onClick={() => gotoPage(pageCount - 1)}
            disabled={!canNextPage}
            className="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-r"
          >
            {'>>'}
          </button>
        </div>
      </div>
      <table
        className="items-center bg-transparent w-full border-collapse"
        {...getTableProps()}
      >
        <thead>
          {headerGroups.map((headerGroup) => (
            <tr {...headerGroup.getHeaderGroupProps()}>
              {headerGroup.headers.map((column) => (
                <th
                  {...column.getHeaderProps(column.getSortByToggleProps())}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Header')}
                  <span>
                    {column.isSorted ? (column.isSortedDesc ? '⬇' : '⬆') : ''}
                  </span>
                  <div>{column.canFilter ? column.render('Filter') : null}</div>
                </th>
              ))}
            </tr>
          ))}
        </thead>
        <tbody {...getTableBodyProps()}>
          {page.map((row) => {
            prepareRow(row);
            return (
              <tr {...row.getRowProps()}>
                {row.cells.map((cell) => {
                  return (
                    <td
                      {...cell.getCellProps()}
                      className="border-t-0 px-6 align-middle border-l-0 border-r-0 text-xs whitespace-nowrap p-4 text-left text-blueGray-700"
                    >
                      {cell.render('Cell')}
                    </td>
                  );
                })}
              </tr>
            );
          })}
        </tbody>
        <tfoot>
          {footerGroups.map((footerGroup) => (
            <tr {...footerGroup.getFooterGroupProps()}>
              {footerGroup.headers.map((column) => (
                <td
                  {...column.getFooterProps()}
                  className="px-6 bg-blueGray-50 text-blueGray-500 align-middle border border-solid border-blueGray-100 py-3 text-xs uppercase border-l-0 border-r-0 whitespace-nowrap font-semibold text-left"
                >
                  {column.render('Footer')}
                </td>
              ))}
            </tr>
          ))}
        </tfoot>
      </table>
    </>
  );
};
```

## Other topics

There are way more things that could be achieved. Two examples out of many:

- Row selection: https://react-table.tanstack.com/docs/examples/row-selection
- Column Hiding: https://react-table.tanstack.com/docs/examples/column-hiding

A list of possibilities is available as a part of the official docs: [https://react-table.tanstack.com/docs/examples/basic](https://react-table.tanstack.com/docs/examples/basic).

With the topics covered in this workshop you should be confident to incorporate things in your tasks.

## Conclusion

Thanks for following this workshop material. Any feedback appreciated (radek.tomasek@gmail.com).
