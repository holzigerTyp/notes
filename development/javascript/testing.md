# JavaScript testing

> How can you implement testing into your existing JavaScript application?

# Table of Contents

- [Setup](#setup)
  - [Required files](#required-files)
  - [Install dependencies](#install-dependencies)
  - [Configuration](#configuration)
- [Generic usage](#generic-usage)
- [Testing applications](#testing-applications)
  - [ExpressJS](#expressjs)
- [Sources](#sources)

# Setup

## Required files

```
tests/
  globalSetup.js
  globalTeardown.js
babel.config.cjs
jest.config.js
```

## Install dependencies

**NPM**
```bash
npm install --save-dev jest babel-jest @babel/core @babel/preset-env
```

**Yarn**
```bash
yarn add -D jest babel-jest @babel/core @babel/preset-env
```

> ℹ️ The babel integration is required in order to make imports in tests work.

## Configuration

**package.json**: Make sure to add the script used for testing
```json
{
  ...
  "scripts": {
    // Add this line
    "test": "jest --verbose --coverage=true --runInBand"
  },
  ...
}
```

> To get more informations about the CLI arguments, take a look [here](https://jestjs.io/docs/cli). The script added above is configured for the testing of an express application.

**tests/globalSetup.js** and **tests/globalTeardown.js**: Insert placeholder, files can be used to do things before and after all tests.
```javascript
export default async () => {}
```

**babel.config.cjs**:
```javascript
module.exports = {
  presets: [['@babel/preset-env', {targets: {node: 'current'}}]],
};
```

**jest.config.js**:
```javascript
/*
 * For a detailed explanation regarding each configuration property and type check, visit:
 * https://jestjs.io/docs/configuration
 */

export default {
  // All imported modules in your tests should be mocked automatically
  // automock: false,

  // Stop running tests after `n` failures
  // bail: 0,

  // The directory where Jest should store its cached dependency information
  // cacheDirectory: "/private/var/folders/77/34p8xdqs1hzdpp6kw8b6zrsr0000gp/T/jest_dy",

  // Automatically clear mock calls, instances, contexts and results before every test
  // clearMocks: false,

  // Indicates whether the coverage information should be collected while executing the test
  // collectCoverage: true,

  // An array of glob patterns indicating a set of files for which coverage information should be collected
  // collectCoverageFrom: undefined,

  // The directory where Jest should output its coverage files
  // coverageDirectory: 'coverage'

  // An array of regexp pattern strings used to skip coverage collection
  // coveragePathIgnorePatterns: [
  //   "/node_modules/"
  // ],

  // Indicates which provider should be used to instrument code for coverage
  // coverageProvider: "babel",

  // A list of reporter names that Jest uses when writing coverage reports
  // coverageReporters: [
  //   "json",
  //   "text",
  //   "lcov",
  //   "clover"
  // ],

  // An object that configures minimum threshold enforcement for coverage results
  // coverageThreshold: undefined,

  // A path to a custom dependency extractor
  // dependencyExtractor: undefined,

  // Make calling deprecated APIs throw helpful error messages
  // errorOnDeprecated: false,

  // The default configuration for fake timers
  // fakeTimers: {
  //   "enableGlobally": false
  // },

  // Force coverage collection from ignored files using an array of glob patterns
  // forceCoverageMatch: [],

  // A path to a module which exports an async function that is triggered once before all test suites
  globalSetup: './tests/globalSetup.js',

  // A path to a module which exports an async function that is triggered once after all test suites
  globalTeardown: './tests/globalTeardown.js',

  // A set of global variables that need to be available in all test environments
  // globals: {},

  // The maximum amount of workers used to run your tests. Can be specified as % or a number. E.g. maxWorkers: 10% will use 10% of your CPU amount + 1 as the maximum worker number. maxWorkers: 2 will use a maximum of 2 workers.
  // maxWorkers: "50%",

  // An array of directory names to be searched recursively up from the requiring module's location
  // moduleDirectories: [
  //   "node_modules"
  // ],

  // An array of file extensions your modules use
  // moduleFileExtensions: [
  //   "js",
  //   "mjs",
  //   "cjs",
  //   "jsx",
  //   "ts",
  //   "tsx",
  //   "json",
  //   "node"
  // ],

  // A map from regular expressions to module names or to arrays of module names that allow to stub out resources with a single module
  // moduleNameMapper: {},

  // An array of regexp pattern strings, matched against all module paths before considered 'visible' to the module loader
  // modulePathIgnorePatterns: [],

  // Activates notifications for test results
  // notify: false,

  // An enum that specifies notification mode. Requires { notify: true }
  // notifyMode: "failure-change",

  // A preset that is used as a base for Jest's configuration
  // preset: undefined,

  // Run tests from one or more projects
  // projects: undefined,

  // Use this configuration option to add custom reporters to Jest
  // reporters: undefined,

  // Automatically reset mock state before every test
  // resetMocks: false,

  // Reset the module registry before running each individual test
  // resetModules: false,

  // A path to a custom resolver
  // resolver: undefined,

  // Automatically restore mock state and implementation before every test
  // restoreMocks: false,

  // The root directory that Jest should scan for tests and modules within
  // rootDir: undefined,

  // A list of paths to directories that Jest should use to search for files in
  roots: [
    'tests/'
  ]

  // Allows you to use a custom runner instead of Jest's default test runner
  // runner: "jest-runner",

  // The paths to modules that run some code to configure or set up the testing environment before each test
  // setupFiles: [],

  // A list of paths to modules that run some code to configure or set up the testing framework before each test
  // setupFilesAfterEnv: [],

  // The number of seconds after which a test is considered as slow and reported as such in the results.
  // slowTestThreshold: 5,

  // A list of paths to snapshot serializer modules Jest should use for snapshot testing
  // snapshotSerializers: [],

  // The test environment that will be used for testing
  // testEnvironment: "jest-environment-node",

  // Options that will be passed to the testEnvironment
  // testEnvironmentOptions: {},

  // Adds a location field to test results
  // testLocationInResults: false,

  // The glob patterns Jest uses to detect test files
  // testMatch: [
  //   "**/__tests__/**/*.[jt]s?(x)",
  //   "**/?(*.)+(spec|test).[tj]s?(x)"
  // ],

  // An array of regexp pattern strings that are matched against all test paths, matched tests are skipped
  // testPathIgnorePatterns: [
  //   "/node_modules/"
  // ],

  // The regexp pattern or array of patterns that Jest uses to detect test files
  // testRegex: [],

  // This option allows the use of a custom results processor
  // testResultsProcessor: undefined,

  // This option allows use of a custom test runner
  // testRunner: "jest-circus/runner",

  // A map from regular expressions to paths to transformers
  // transform: {
  //   '^.+\\.tsx?$': [
  //     'ts-jest',
  //     {}
  //   ]
  // }

  // An array of regexp pattern strings that are matched against all source file paths, matched files will skip transformation
  // transformIgnorePatterns: [
  //   "/node_modules/",
  //   "\\.pnp\\.[^\\/]+$"
  // ],

  // An array of regexp pattern strings that are matched against all modules before the module loader will automatically return a mock for them
  // unmockedModulePathPatterns: undefined,

  // Indicates whether each individual test should be reported during the run
  // verbose: undefined,

  // An array of regexp patterns that are matched against all source file paths before re-running tests in watch mode
  // watchPathIgnorePatterns: [],

  // Whether to use watchman for file crawling
  // watchman: true,
}
```

> If you are using exactly this configuration, the `tests` directory will contain all your test files. Of course, you can modify the configuration as you like.

# Generic usage

1. Create a test file in the following schema `<descriptor>.test.js`
2. Add your tests, e. g. this example:
   ```javascript
   describe('basic', () => {
     test('adds 1 + 2 to equal 3', () => {
       expect(1 + 2).toBe(3);
     })
   })
   ```
3. Run the tests using `npm test` / `yarn test`

# Testing applications

## ExpressJS

### Configuration
- Make sure to use the CLI argument `--runInBand`, if you are working with tests that depend on each other.

### Preparation

**Separate your app and server**

> Testing requires that the app does not listen to any ports. The "listening" is handled by the testing framework.

Example `server.js`
```javascript
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.status(200).send("Hello World!");
});

module.exports = app;
```

Example `app.js`
```javascript
const app = require("./app");

app.listen(5678, () => {
  console.log("Example app listening on port 5678!");
});
```

### Writing tests

```javascript
import request from 'supertest'
import app from '../src/app'

describe('express', () => {
  it('should return 200 at /', async () => {
    const response = await request(app).get('/')
    expect(response.status).toBe(200)
  })
})
```

## MongoDB (using `mongodb-memory-server`)

**Documentation:** [https://nodkz.github.io/mongodb-memory-server/](https://nodkz.github.io/mongodb-memory-server/docs/guides/quick-start-guide)
**Repository:** [https://github.com/nodkz/mongodb-memory-server](https://github.com/nodkz/mongodb-memory-server)

### Preparation

**Install dependency**
```bash
npm install --save-dev mongodb-memory-server
```
```bash
yarn add -D mongodb-memory-server
```

**Add global setup**
```javascript
import { MongoMemoryServer } from 'mongodb-memory-server'

export default async () => {
  const mongoServer = await MongoMemoryServer.create()
  const mongoUri = mongoServer.getUri('rbac')

  // Save the mongoUri as a global variable, so that it can be accessed in test files.
  process.env.__MONGO_URI__ = mongoUri

  // Save the mongoServer as a global variable, so that it can be accessed in globalTeardown.
  global.__MONGO_SERVER__ = mongoServer

}
```

**Add global teardown**
```javascript
export default async () => {
  const mongoServer = global.__MONGO_SERVER__
  await mongoServer.stop()
}
```

# Sources

- [How to test Express.js with Jest and Supertest, Albert Gao](https://www.albertgao.xyz/2017/05/24/how-to-test-expressjs-with-jest-and-supertest/)
