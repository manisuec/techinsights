---
background: /img/posts/02.png
date: "2018-08-24T00:00:00Z"
title: 'Love thy fellow programmer as thyself: Setup ESLint and Prettier in VSCode'
---

This is a short guide to configure VS Code for a consistent and reusable development set-up.

# ESLint Setup

ESLint is an open source JavaScript linting utility. Code linting is a type of static analysis that is frequently used to find problematic patterns or code that doesn’t adhere to certain style guidelines. The primary reason ESLint was created was to allow developers to create their own linting rules. ESLint is designed to have all rules completely pluggable.

Install ESLint locally and create a .eslintrc.json config file by running the below commands in the workspace folder —

```
$ npm install eslint --save-dev
$ ./node_modules/.bin/eslint --init
```

I prefer local setup for each project however, it can be installed globally too. Run the below commands for a global installation and generating a .eslintrc.json config file —

```
$ npm install -g eslint
$ eslint --init
```

Few configuration related questions will be asked, choose the options appropriately according to your preferences

```
? How would you like to configure ESLint? Use a popular style guide
? Which style guide do you want to follow? Standard
? What format do you want your config file to be in? JSON
```

Install ESLint VS Code extension using command `ext install vscode-eslint`. For more information and configuration settings refer [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)

# Prettier Setup

Prettier is an opinionated code formatter with support for JavaScript, including ES2017. It removes all original styling* and ensures that all outputted code conforms to a consistent style.

Run below command to install Prettier

```
$ npm install --save-dev prettier
```

# Integrating with ESLint

ESLint can run Prettier for you, just add Prettier as an ESLint rule using

```
$ npm install eslint-plugin-prettier eslint-config-prettier --save-dev
```

**eslint-config-prettier** is a config that disables rules that conflict with Prettier.

**eslint-plugin-prettier** is a plugin that adds a rule that formats content using Prettier.

**eslint-plugin-prettier** exposes a "recommended" configuration that configures both eslint-plugin-prettier and eslint-config-prettier in a single step.

Add below lines in ‘.eslintrc.json’

```
{
  "extends": [
    "prettier",
    "plugin:prettier/recommended"
  ],
  "plugins": [
    "prettier"
  ],
  "rules": {
    "prettier/prettier": "error"
  }
}
```

**‘eslint-config-prettier’** also ships with a little CLI tool to help you check if your configuration contains any rules that are unnecessary or conflict with Prettier.


Add this srcipt in ‘package.json’

```
"eslint-check": "eslint --print-config .eslintrc.json | eslint-config-prettier-check"
```

Then run ```npm run eslint-check```

# Importance

Prettier essentially compliments ESLint rules concerning styling. They both help in ensuring a consistent and clean code. This speeds up the development process by improving the quality of code-reviews. Developers will focus more on implementation detail rather than complain about syntax errors and different coding styles. For more information read [Why Prettier?](https://prettier.io/docs/en/why-prettier.html)

## Tip
a) Pre-commit hook can be set-up so that the code is always formatted before the commit. [Pre-commit hook](https://gist.github.com/cadebward/c26e218220d653385d876f9a81308140)

b) Enable auto fix: View → Command Pallete → Open User Settings → add: ```"eslint.autoFixOnSave": true``` line to the settings

# Conclusion

This sets up VS Code with ESLint and Prettier for development with a consistent configuration.