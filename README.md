# dotenv-multi

<img src="https://raw.githubusercontent.com/patsissons/dotenv-multi/master/dotenv-multi.png" alt="dotenv-multi" align="right" />

[`dotenv-multi`](https://github.com/patsissons/dotenv-multi) is an extension to [`dotenv`](https://github.com/motdotla/dotenv) that loads environment variables from multiple `.env*` files based on `NODE_ENV` into [`process.env`](https://nodejs.org/docs/latest/api/process.html#process_process_env).

[![BuildStatus](https://img.shields.io/travis/patsissons/dotenv-multi/master.svg?style=flat-square)](https://travis-ci.org/patsissons/dotenv-multi)
[![Build status](https://ci.appveyor.com/api/projects/status/github/patsissons/dotenv-multi?svg=true)](https://ci.appveyor.com/project/patsissons/dotenv-multi/branch/master)
[![NPM version](https://img.shields.io/npm/v/dotenv-multi.svg?style=flat-square)](https://www.npmjs.com/package/dotenv-multi)
[![Coverage Status](https://img.shields.io/coveralls/patsissons/dotenv-multi/master.svg?style=flat-square)](https://coveralls.io/github/patsissons/dotenv-multi?branch=coverall-intergration)

## Install

```bash
# with npm
npm install --dev dotenv-multi

# or with Yarn
yarn add -D dotenv-multi
```

## Usage

As early as possible in your application, require and configure `dotenv-multi`.

```javascript
require('dotenv-multi').config();
```

```typescript
import {config} from 'dotenv-multi';

confg();
```

Create a `.env` file in the root directory of your project. Add
environment-specific variables on new lines in the form of `NAME=VALUE`.
For example:

```dosini
DB_HOST=localhost
DB_USER=root
DB_PASS=s1mpl3
```

That's it.

`process.env` now has the keys and values you defined in your `.env` file.

```javascript
const db = require('db');
db.connect({
  host: process.env.DB_HOST,
  username: process.env.DB_USER,
  password: process.env.DB_PASS,
});
```

### What other .env\* files can I use?

`dotenv-multi` will override in the following order (highest defined variable overrides lower):

| Hierarchy Priority | Filename                 | Environment          | Should I `.gitignore`it?                            | Notes                                                                      |
| ------------------ | ------------------------ | -------------------- | --------------------------------------------------- | -------------------------------------------------------------------------- |
| 1st (highest)      | `.env.development.local` | Development          | Yes!                                                | Local overrides of environment-specific settings.                          |
| 1st                | `.env.test.local`        | Test                 | Yes!                                                | Local overrides of environment-specific settings.                          |
| 1st                | `.env.production.local`  | Production           | Yes!                                                | Local overrides of environment-specific settings.                          |
| 2nd                | `.env.local`             | Wherever the file is | Definitely.                                         | Local overrides. This file is loaded for all environments _except_ `test`. |
| 3rd                | `.env.development`       | Development          | No.                                                 | Shared environment-specific settings                                       |
| 3rd                | `.env.test`              | Test                 | No.                                                 | Shared environment-specific settings                                       |
| 3rd                | `.env.production`        | Production           | No.                                                 | Shared environment-specific settings                                       |
| Last               | `.env`                   | All Environments     | Depends (See [below](#should-i-commit-my-env-file)) | The Original®                                                              |

### Preload

You can use the `--require` (`-r`) [command line option](https://nodejs.org/api/cli.html#cli_r_require_module) to preload dotenv. By doing this, you do not need to require and load `dotenv-multi` in your application code. This is the preferred approach when using `import` instead of `require`.

```bash
$ node -r dotenv-multi/config your_script.js
```

<!--
The configuration options below are supported as command line arguments in the format `dotenv_config_<option>=value`

```bash
$ node -r dotenv/config your_script.js dotenv_config_path=/custom/path/to/your/env/vars
```

Additionally, you can use environment variables to set configuration options. Command line arguments will precede these.

```bash
$ DOTENV_CONFIG_<OPTION>=value node -r dotenv/config your_script.js
```

```bash
$ DOTENV_CONFIG_ENCODING=latin1 node -r dotenv/config your_script.js dotenv_config_path=/custom/path/to/.env
```
-->

## Config

_Alias: `load`_

`config` will read your `.env*` files, parse the contents, assign it to
[`process.env`](https://nodejs.org/docs/latest/api/process.html#process_process_env),
and return an Object with a `parsed` key containing the loaded content or an `error` key if it failed.

```js
const result = dotenv.config();

if (result.error) {
  throw result.error;
}

console.log(result.parsed);
```

You can additionally, pass options to `config`.

### Options

#### `basePath`

Default: `path.resolve(process.cwd(), '.env')`

You may specify a custom base path if your file containing environment variables is located elsewhere.

```js
require('dotenv').config({basePath: '/full/custom/path/to/your/env/vars/.env'});
```

#### `encoding`

Default: `utf8`

You may specify the encoding of your file containing environment variables.

```js
require('dotenv').config({encoding: 'latin1'});
```

#### `debug`

Default: `false`

You may turn on logging to help debug why certain keys or values are not being set as you expect.

```js
require('dotenv').config({debug: process.env.DEBUG});
```

## Parse

The engine which parses the contents of your file containing environment
variables is available to use. It accepts a String or Buffer and will return
an Object with the parsed keys and values.

```js
const dotenv = require('dotenv-multi');
const buf = Buffer.from('BASIC=basic');
const config = dotenv.parse(buf); // will return an object
console.log(typeof config, config); // object { BASIC : 'basic' }
```

### Options

#### `debug`

Default: `false`

You may turn on logging to help debug why certain keys or values are not being set as you expect.

```js
const dotenv = require('dotenv-multi');
const buf = Buffer.from('hello world');
const opt = {debug: true};
const config = dotenv.parse(buf, opt);
// expect a debug message because the buffer is not in KEY=VAL form
```

### Rules

The parsing engine currently supports the following rules:

- `BASIC=basic` becomes `{BASIC: 'basic'}`
- empty lines are skipped
- lines beginning with `#` are treated as comments
- empty values become empty strings (`EMPTY=` becomes `{EMPTY: ''}`)
- single and double quoted values are escaped (`SINGLE_QUOTE='quoted'` becomes `{SINGLE_QUOTE: "quoted"}`)
- new lines are expanded if in double quotes (`MULTILINE="new\nline"` becomes

```
{MULTILINE: 'new
line'}
```

- inner quotes are maintained (think JSON) (`JSON={"foo": "bar"}` becomes `{JSON:"{\"foo\": \"bar\"}"`)
- whitespace is removed from both ends of the value (see more on [`trim`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/Trim)) (`FOO=" some value "` becomes `{FOO: 'some value'}`)

## FAQ

### What happens to environment variables that were already set?

We will never modify any environment variables that have already been set. In particular, if there is a variable in your `.env` file which collides with one that already exists in your environment, then that variable will be skipped. This behavior allows you to override all `.env` configurations with a machine-specific environment, although it is not recommended.

If you want to override `process.env` you can do something like this:

```javascript
const fs = require('fs');
const dotenv = require('dotenv-multi');
const envConfig = dotenv.parse(fs.readFileSync('.env.override'));
for (let k in envConfig) {
  process.env[k] = envConfig[k];
}
```

### What about variable expansion?

Try [dotenv-expand](https://github.com/motdotla/dotenv-expand)

## Contributing Guide

See [CONTRIBUTING.md](CONTRIBUTING.md)

## Change Log

See [CHANGELOG.md](CHANGELOG.md)

## License

See [LICENSE](LICENSE.md)
