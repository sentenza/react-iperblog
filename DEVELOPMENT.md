# Development guide

Libraries:

- [ReactJS](https://github.com/facebook/react)
- [React Router](https://github.com/rackt/react-router)
- [RefluxJS](https://github.com/spoike/refluxjs)
- [SASS](http://sass-lang.com/)

## Workflow

1. Install dependencies: `npm install`
2. Make changes, usually contained within `src` directory
3. Build and watch changes in `src/` with `npm start`
4. Lint and build the project to `build/` using `npm run build`
5. Run tests with `npm test`

## Commands

`npm start`

> Run development instance of application on port <%= data.port %> via a
> webpack development server with hot reloading of code via react-hot-loader.

`npm run build`

> Build production-ready static files into a `build` directory.

`npm test`

> Run all the tests for the project in the `tests` directory, as well as
> generation of coverage reports in the `.coverage` directory.

## Configuration

This project does not use any external Node.js building tools, e.g. Grunt, gulp,
instead relying on npm scripts. The build process is managed via Neo's webpack
and different webpack loaders. Webpack is used here because it provides a nice
development environment with hot code reloading. Configuration for development,
testing, and production can be extended as follows.

#### Building

Neo uses Webpack and Webpack Dev Server as the build engine to transpile code,
run tests, and lint code style. If you wish to make changes or override these
settings, you can create your own webpack configuration file in the project and
pass this configuration file to the `npm scripts`. Neo exposes a number of
configurations to manipulate the build environments: `webpack.core`,
`webpack.dev`, `webpack.prod`, and `webpack.test`.

As an example, if you wanted to change the behavior of `npm start`, you could
create a `webpack.start.js` file in the root of the project:

```js
// webpack.start.js
let devConfig = require('mozilla-neo/config/webpack.dev');

devConfig.devtool = 'cheap-inline-source-map';

module.exports = devConfig;
```

then modify the `npm start` script to use this custom configuration:

```json
// package.json
{
  "scripts": {
    "start": "neo start --port <%= data.port %> --config webpack.start.js"
  }
}
```

For reference:

- [`config/webpack.core`](https://github.com/mozilla/neo/blob/master/config/webpack.core.js):
The webpack config base used by all other configurations
- [`config/webpack.dev`](https://github.com/mozilla/neo/blob/master/config/webpack.dev.js):
The webpack config used in the development process `npm start`
- [`config/webpack.test`](https://github.com/mozilla/neo/blob/master/config/webpack.test.js):
The webpack config used when running `npm test`. This config removes any plugins defined in `webpack.core`.
- [`config/webpack.prod`](https://github.com/mozilla/neo/blob/master/config/webpack.prod.js):
The webpack config used when running `npm run build`

Additional configurations to modify or utilize:

- [`config/babel`](https://github.com/mozilla/neo/blob/master/config/babel.js): The Babel presets used internally by default
- [`config/eslint.core`](https://github.com/mozilla/neo/blob/master/config/eslint.core.js): ESLint default plugins, presets, and rules
- [`config/eslint.dev`](https://github.com/mozilla/neo/blob/master/config/eslint.dev.js): ESLint development rules overrides
- [`config/karma`](https://github.com/mozilla/neo/blob/master/config/karma.js): Karma testing and coverage settings

##### HTML template

Neo uses its own [HTML template](https://github.com/mozilla/neo/blob/master/src/template.ejs)
to generate the initial markup for the static page it will render. To specify a
custom template, create a `template.ejs` file in the `src` directory and Neo
will pick it up automatically. In your `package.json` there is a `config.html`
section where you can specify custom variables for your template. These values
can be accessed via `htmlWebpackPlugin.options.custom_variable_name`. See the
default [HTML template](https://github.com/mozilla/neo/blob/master/src/template.ejs)
for an demonstration of this usage.

#### Transpiling

Neo uses Babel to transpile unsupported JavaScript syntax to a supported syntax.
By default Neo uses the following Babel presets to render syntax:

- [ES2015](https://babeljs.io/docs/plugins/preset-es2015/)
- [Stage 0](https://babeljs.io/docs/plugins/preset-stage-0/)
- [React](https://babeljs.io/docs/plugins/preset-react/)

If you would like to supply your own Babel presets, plugins, or configuration,
simply add a `.babelrc` file to the root of this project and Neo will pick it up
automatically. [How to use the babelrc](https://babeljs.io/docs/usage/babelrc/).

You can also make manual changes to Neo's Babel configuration by requiring,
modifying, and passing that directly to your [custom webpack configuration](#Building).

```js
let babelConfig = require('mozilla-neo/config/babel');

// modify babelConfig

module.exports = {
  loaders: [
    {
      loader: 'babel',
      query: babelConfig
    }
  ]
};
```

#### Linting

Neo uses ESLint to enforce code style, and comes pre-configured with several
plugins and rules to get up and running quickly. Code style within the `src/`
directory is enforced using ESLint. This process is run whenever webpack is
invoked, i.e. when running the development server, when building, or running
tests. Using your own rules can be done either by modifying the existing
configuration or providing a completely custom configuration. You may choose to
modify these values directly from the webpack configuration, or a custom
[ESLint Configuration file](http://eslint.org/docs/user-guide/configuring#configuration-file-formats).

To use a custom ESLint configuration file, create it in the root of your
project, then specify this to webpack in your [custom webpack configuration](#Building).

As an example, let's say you would like to change the ESLint configuration for
the `npm run build` script:

```json
// .eslintrc.json
{
  "rules": {
    "semi": [2, "never"]
  }
}
```

```js
// webpack.custom.js
let build = require('mozilla-neo/config/webpack.prod');
let path = require('path');

build.eslint = {
  configFile: path.join(process.cwd(), '.eslintrc.json')
};

module.exports = build;
```

```json
// package.json
{
  "scripts": {
    "build": "neo build --config webpack.custom.js"
  }
}
```

It is recommended if you only have minor changes to the ESLint configuration to just modify the desired config in your
custom ESLint configuration:

```js
// .eslintrc.js
let core = require('mozilla-neo/config/eslint.core');

core.rules.semi = [2, 'never'];

module.exports = core;
```

#### Testing

Neo uses the Karma test runner, along with Mocha and Chai to run tests and
output code coverage. Test build configuration can be done by either modifying
the Neo's presets or providing your own, and sending this to the Karma
configurator. Karma has two configuration files: a `karma` file which configures
the Karma runner and its server, and a `webpack.test` file which configures how
the build is executed when running tests.

As an example, let's say you would like to change the directory for coverage
reports from `.coverage` to `coverage` in a custom `karma.custom.js`:

```js
// karma.custom.js
let config = require('mozilla-neo/config/karma');

config.coverageReporter.dir = 'coverage';

module.exports = (karma) => karma.set(config);
```

```json
// package.json
{
  "scripts": {
    "test": "neo test --config karma.custom.js"
  }
}
```

## References

- [Webpack](https://webpack.github.io/)
- [Webpack Dev Server](https://webpack.github.io/docs/webpack-dev-server.html)
- [React Hot Loader](https://github.com/gaearon/react-hot-loader)
- [Karma](https://karma-runner.github.io)
- [Enzyme](http://airbnb.io/enzyme/)
- [Mocha](http://mochajs.org/)
- [Chai](http://chaijs.com/)
- [ESLint](http://eslint.org/)
- [Babel](http://babeljs.io/)
- [ES2015](https://babeljs.io/learn-es2015/)


### ReactJS

ReactJS is a "declarative, efficient, and flexible JavaScript library for building user interfaces."

- "**Just the UI**: Lots of people use React as the V in MVC. Since React makes no assumptions about the rest of your technology stack, it's easy to try it out on a small feature in an existing project."
- "**Virtual DOM**: React uses a virtual DOM diff implementation for ultra-high performance. It can also render on the server using Node.js — no heavy browser DOM required."
- "**Data flow**: React implements one-way reactive data flow which reduces boilerplate and is easier to reason about than traditional data binding."

The ReactJS files are all located within `/app/js`, structured in the following manner:

```
/components
  - Footer.js (Simple, static footer component rendered on all pages.)
  - Header.js (Simple, static header component rendered on all pages.)
/pages
  - HomePage.js (Example home page, serving as the default route.)
  - NotFoundPage.js (Displayed any time the user requests a non-existent route.)
  - SearchPage.js (Example search page to demonstrate navigation and individual pages.)
/utils
  - APIUtils.js (General wrappers for API interaction via Superagent.)
  - AuthAPI.js (Example functions for user authorization via a remote API.)
App.js (The main container component, rendered to the DOM and then responsible for rendering all pages.)
index.js (The main javascript file watched by Browserify, responsible for requiring the app and running the router.)
Routes.js (Defines the routing structure, along with each individual route path and handler.)
```

Each module you add to your project should be placed in the appropriate directory, and required in the necessary files. Once required, they will be automatically detected and compiled by Browserify (discussed later).

---

### RefluxJS

RefluxJS is a "simple library for unidirectional dataflow architecture inspired by ReactJS Flux."

"The pattern is composed of actions and data stores, where actions initiate new data to pass through data stores before coming back to the view components again. If a view component has an event that needs to make a change in the application's data stores, they need to do so by signalling to the stores through the actions available."

The RefluxJS files are also all locationed within `/app/js`, structured in the following manner:

```
/actions
  - CurrentUserActions.js (Possible actions relevant to the current user. i.e. `checkAuth`, `login`, and `logout`.)
/stores
  - CurrentUserStore.js (Responsible for storing the current user data, while listening to any `CurrentUserActions`.)
```

Each action or store you add to your project should be placed in the appropriate directory, and required in the necessary files. The necessary logic to trigger actions and listen to stores should also be added.

---

### React Router

React Router is a "complete routing library for React." It uses the JSX syntax to easily define route URLs and handlers, providing an easy-to-understand architecture and thus makes it easy to add new pages and routes.

The relevant files are all located within `/app/js`, structured in the following manner:

```
/pages (Each individual page to handle the defined routes and be rendered inside the app.)
App.js (The main component which is rendered to the DOM and responsible for rendering the current page.)
index.js (The main javascript file watched by Browserify, requiring the app and running the router.)
Routes.js (Defines the routing structure, along with each individual route path and handler.)
```

Any pages added to your project should be placed within the `app/js/pages` directory, and be required and assigned to a route inside `Routes.js`. If more complex nesting is required, any page can have a new `RouteHandler` as a child component.

