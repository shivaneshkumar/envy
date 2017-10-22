# envy [![Build status for Envy](https://img.shields.io/circleci/project/sholladay/envy/master.svg "Build Status")](https://circleci.com/gh/sholladay/envy "Builds")

> Load .env files and environment variables

Secure and friendly alternative to [dotenv](https://npmjs.com/package/dotenv), using functional programming. With strong tests to prove its safety.

Follows the [Twelve Factor App](https://12factor.net) methodology.

*Looking for the old `envy` that reads JSON? See [eliOcs/node-envy](https://github.com/eliOcs/node-envy).*

## Why?

 - Safest environment loader around.
 - Validates secure file permissions.
 - Validates that required variables are defined.
 - No side effects, does not modify [`process.env`](https://nodejs.org/api/process.html#process_process_env).
 - Returns property names in [camelCase](https://github.com/sindresorhus/camelcase).

Have you spent a day rotating passwords because a developer accidentally pushed them to the repository? Yeah, that actually happens and it can cause chaos at companies. It doesn't matter if you delete the commit. Once it's out there, game over. People who are authorized to read the repository may not be authorized to have those credentials, including third party tools and services. If the repository is public, search engines may have crawled it. Consider everything to be compromised.

The `envy` module helps you prevent that situation by providing a convenient mechanism for everyone to store credentials and other config locally and validate that it is correct without commiting anything to the repository. It verifies that all relevant files have secure permissions and _coming soon_ that the secrets file is explicitly ignored so that it cannot accidentally be commited.

## Install

```sh
npm install envy --save
```

## Usage

Get it into your program.

```js
const envy = require('envy');
```

Load the environment variables.

```js
const env = envy();
// {
//     foo : 'bar'
// }
```

A `.env.example` file is used as a template to determine which environment variables are required by your application. This file should be commited to your repository.

```sh
# .env.example
FOO=
```

A `.env` file is used to provide the actual environment values. You should add this file to a local [`.gitignore`](https://help.github.com/articles/ignoring-files).

```sh
# .env
FOO=bar
```

Environment files look just like normal shell files (e.g. `.bashrc` and friends). Values with spaces may be optionally wrapped in `'` single quotes. This is recommended for shell compatibility.

```sh
# This is a comment.
FOO=bar
SENTENCE='Hi, there!'
```

After assembling the environment from `.env` and [`process.env`](https://nodejs.org/api/process.html#process_process_env), it is filtered by a union of the properties in `.env` and `.env.example`. In other words, variables defined in either file will be returned and variables not defined in either file will be ignored. This is done to prevent surprising behavior, since `process.env` often contains a wide variety of variables, some of which are implicitly set without the user's direct knowledge. It is better to be explicit about the variables you use, which improves safety and debugging.

## Tips

### Mixing with command line options

Make a CLI with [meow](https://github.com/sindresorhus/meow) and use [joi](https://github.com/hapijs/joi) to further validate and parse the returned values.

```js
// cli.js
const meow = require('meow');
const envy = require('envy');
const joi = require('joi');

const cli = meow();

const config = Object.assign({}, envy(), cli.flags);
const { error, value } = joi.validate(config, {
    port : joi.number().positive().integer().min(0).max(65535)
});
if (error) {
    throw error;
}
// value.port has been parsed as a number
console.log('port:', value.port);
```
```sh
# .env.example
PORT=
```
```sh
# .env
PORT=1000
```

```console
$ node cli.js
port: 1000
$ PORT=2000 node cli.js
port: 2000
$ node cli.js --port=3000
port: 3000
$ PORT=2000 node cli.js --port=3000
port: 3000
```

## API

### envy(filepath)

Returns an `object` with environment variables derived from [`process.env`](https://nodejs.org/api/process.html#process_process_env) and the contents at `filepath`. If a variable is defined in both places, `process.env` takes precedence so that users can easily override values on the command line. If all required variables are present in `process.env`, then the `.env` file need not exist. All property names are returned in camelcase.

#### filepath

Type: `string`<br>
Default: `.env`

Path to the file where environment variables are kept. Must be [hidden](https://en.wikipedia.org/wiki/Hidden_file_and_hidden_directory#macOS) (start with a `.`). Will also be used to compute the path of the example file (by appending `.example`).

## Resources

 - [Think about sensitive data](http://blog.arvidandersson.se/2013/06/10/credentials-in-git-repos)
 - [Manage secrets safely](https://digitalocean.com/community/tutorials/an-introduction-to-managing-secrets-safely-with-version-control-systems)
 - [Remove secrets from a repository](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)
 - [Store config in the environment](https://12factor.net/config)

## Contributing

See our [contributing guidelines](https://github.com/sholladay/envy/blob/master/CONTRIBUTING.md "Guidelines for participating in this project") for more details.

1. [Fork it](https://github.com/sholladay/envy/fork).
2. Make a feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. [Submit a pull request](https://github.com/sholladay/envy/compare "Submit code to this project for review").

## License

[MPL-2.0](https://github.com/sholladay/envy/blob/master/LICENSE "License for envy") © [Seth Holladay](https://seth-holladay.com "Author of envy")

Go make something, dang it.
