# lerna-semantic-versioning-example

This document contains a step by step guide to implement a minimal example to test how **lerna - semantic versioning** works.

Inspired by this [blog post](https://michaljanaszek.com/blog/lerna-conventional-commits/).

## Lerna project setup

I always prefer to not install dependencies as global if it is possible. So **here i'm using lerna as a dev dependency**:

```
yarn init
yarn add lerna -D
yarn lerna init --independent
```

**_Note_**: on initialization it is essential to set the _independent_ flag. It allows us to manage sub-projects versioning independently instead of having a single version for all of them.

## Lerna configs setup

With [Semantic Versioning](https://semver.org/) there is not one and only rule that establishes how the version of a project should be increased. However, lerna in combination with the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) format proposes a **solution to automatically determine semantic version bumps** and generate change logs. In summary, if a project (or sub-project in our case) has been updated and its commit message implies:

- a **BREAKING CHANGE**, then bumps the **MAJOR** number
- a **FEATURE**, then bumps the **MINOR** number
- a **REFACTOR** / **BUG-FIX**, then bumps the **PATCH** number

### Conventional Commits flag setup

In the process of releasing changes, that is achieved by running

```bash
yarn lerna publish
```

lerna will check the updated sub-projects and will create and push a new commit with version bumps and change logs (it will also update packages inter-dependencies).

To instruct lerna that we have chosen to follow the Conventional Commits format we must specify:

```json
// lerna.json
...
"command": {
  "publish": {
    "conventionalCommits": true
  }
}
...
```

### Publish message setup

It can be also useful to customize the publish commit message like this

```json
// lerna.json
...
"command": {
  "publish": {
    ...
    "message": "chore(release): publish"
    ...
  }
}
...
```

Otherwise, it will be "Publish" by default

### Github private package registry setup (optional)

As final step, updated packages (if public, that is "private" field is setted to false or just missing in the relative package.json) will be published to a registry. To have them published on our private github registry you simply need to add this:

```json
// lerna.json
...
"command": {
  "publish": {
    ...
    "registry": "https://npm.pkg.github.com/"
    ...
  }
}
```

Then, before a release, make sure you are logged into your github-npm account. Otherwise run

```bash
npm login --registry=https://npm.pkg.github.com/
```

Where your password will be a repo and packages scoped github personal access token.

**_Note_**: other configurations related to this step will also be made at sub-projects level. We will see it in the next section ...

### An important note on the lerna publish command

As mentioned above when "private" field in a package.json is missing or setted to false lerna will try to publish your package on a registry.
If it is so and there is no configuration related to the registry url, lerna will try to publish it in the npm public registry by default.

This could be an unwanted behavior! :scream_cat:

If you don't want to publish any package then consider using one of these two commands for the project release:

```bash
yarn lerna publish --skip-npm
```

Or

```bash
yarn lerna version
```

Otherwise make sure you wrote all the necessary configurations: define registry urls for what you want to publish or set packages as private for what you don't want to publish!

### Final result

Your final config file will look like this:

```json
// lerna.json
{
  "packages": ["packages/*"],
  "version": "independent",
  "command": {
    "publish": {
      "conventionalCommits": true,
      "message": "chore(release): publish",
      "registry": "https://npm.pkg.github.com/"
    }
  }
}
```

## Coding packages

Here is the code for our three subprojects (thanks to which I will explain some concepts in the next section):

- alpha
- beta
- usage

Where _alpha_ and _beta_ are independent packages, while _usage_ depends on _alpha_ and _beta_.

**_Note_**: remember to change _@ your-username_ and github repo occurrencies

### alpha

```bash
mkdir packages/alpha
touch packages/alpha/index.js
touch packages/alpha/package.json
```

```js
// packages/alpha/index.js

module.exports = "alpa";
```

```json
// packages/alpha/package.json

{
  "name": "@your-username/alpha",
  "repository": "git@github.com:your-username/your-repo-name.git",
  "version": "0.0.0",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com/"
  }
}
```

### beta

```bash
mkdir packages/beta
touch packages/beta/index.js
touch packages/beta/package.json
```

```js
// packages/beta/index.js

module.exports = "beta";
```

```json
// packages/beta/package.json

{
  "name": "@your-username/beta",
  "repository": "git@github.com:your-username/your-repo-name.git",
  "version": "0.0.0",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com/"
  }
}
```

### usage

```bash
mkdir packages/usage
touch packages/usage/index.js
touch packages/usage/package.json
```

```js
// packages/usage/index.js

const alpha = require("@your-username/alpha");
const beta = require("@your-username/beta");
console.log(alpha + " " + beta);
```

```json
// packages/usage/package.json

{
  "name": "@your-username/usage",
  "version": "0.0.0",
  "repository": "git@github.com:your-username/your-repo-name.git",
  "dependencies": {
    "@your-username/alpha": "^0.0.0",
    "@your-username/beta": "^0.0.0"
  },
  "publishConfig": {
    "registry": "https://npm.pkg.github.com/"
  }
}
```

From the project root directory, run

    yarn lerna bootstrap

to install dependencies. Note that internal dependencies will be automatically linked by lerna.

Now cd to the usage package and run

    node index

You should see "alpa beta" logged into the terminal.

## Working with repo

Because commit messages affects how version is bumped and change logs are generated, each commit will be scoped to a particular feat or fix for a particular package.

### Monorepo structure

We are going to commit monorepo boilerplate first

```bash
git add .
git reset -- packages
```

so that we are staging all files except our packages. Then

```bash
git commit -m "feat: created a lerna monorepo"
```

Note that for this particular commit the Conventional Commits format is irrelevant, but it can be a good exercise to apply it right now.

### alpha package

```bash
git add packages/alpha
git commit -m "feat: added export of package name"
```

### beta package

```bash
git add packages/beta
git commit -m "feat: added export of package name"
```

### usage package

```bash
git add packages/usage
git commit -m "feat: added export of concatenated package dependencies name"
```

### First publish

```bash
git push origin master
yarn lerna publish
```

Now we have our packages published at version 0.1.0 (remember how bumping works) and change logs generated per sub-project level.

Check it out!

### Time to fix

You may have already noticed that the alpha package has a typo in the name it exports. is "alpa" instead of "alpha". We need to fix it

```js
// packages/alpha/index.js

module.exports = "alpha";
```

Now we can release a patched version for alpha

```bash
git commit -am "fix: typo in message"
git push origin master
yarn lerna publish
```

You can see also usage package has bumped because of a dependency to alpha that has been automatically updated by lerna.

## Conclusions

If you work in a team on a fairly complex project, I think having a code versioned according to these standards can simplify your life and save it in some cases! First, you have a history to go through and if the commits are self-explanatory you also have good documentation for free. Second (valid in general for any type of versioning), in case of emergency in production you can always switch back to a stable version of your product!
