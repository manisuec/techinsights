---
layout: post
title: 'Use NVM and .nvmrc for a better Javascript development'
date: 2023-01-04 00:00:00 +0530
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1672811003/nvm_gyddwb.png']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676698473/nodejs_dark_cjoudy.png'
tags: ['nodejs', 'nvm', 'javascript']
keywords: 'nodejs,nvm,javascript,version,multiple node version,version manager'
categories: ['Nodejs']
url: 'nodejs/nvm-nodejs-nvmrc'
---

Node Version Manager ([NVM](https://github.com/nvm-sh/nvm/blob/master/README.md)) is an open source version manager for Node.js (Node) and allows to easily install & manage different versions of Node and switch between them on a per-shell basis.

**Example:**

```sh
$ nvm use v18.12.1
Now using node v18.12.1 (npm v8.19.2)

$ node -v
v18.12.1

$ nvm use v14.18.0
Now using node v14.18.0 (npm v6.14.15)

$ node -v
v14.18.0
```

## Installation

Use the following cURL or Wget command:

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```
```sh
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

The steps for installation and configuration are easy; refer [nvm installation steps](https://github.com/nvm-sh/nvm/blob/master/README.md#installing-and-updating).

Basic commands and their usage can be referenced from [Usage section](https://github.com/nvm-sh/nvm/blob/master/README.md#usage-1).

## Use nvm like a superhero

i) **Node installation**

For first time Node installation use `$ nvm install v18.12.1`

However, prefer 

```sh
$ nvm install <NEW_NODE_VERSION> --reinstall-packages-from=<OLD_NODE_VERSION>
```

e.g.

```sh
$ nvm install v18.12.1 --reinstall-packages-from=v16.19.0
```

This installs a new version of Node.js and also migrate npm packages from a previous version.

ii) **.nvmrc**

When you are working on different projects and use different versions of Node then **.nvmrc** will make your life very easy. Create a .nvmrc file in the root of the project. 

e.g.

```sh
$ cat .nvmrc
v18.6.0
```

Instead of you remembering which project uses which version and manually setting the node version using `nvm use <node-version>` command; simply use the command `nvm use`

```sh
nvm use
Found '/home/user/project_xyz/.nvmrc' with version <v18.6.0>
Now using node v18.6.0 (npm v9.2.0)
```

If the Node version specified in the .nvmrc file is not installed it will tell you to run the nvm install command to download and install that version of node.

iii) **Shell Integration**

Shell script are there which you can add to your shells configuration file (.bashrc, .zshrc etc) to automatically nvm use and install the node version defined in the .nvmrc file when you change into a directory.

**e.g. zsh**

**Calling `nvm use` automatically in a directory with a `.nvmrc` file**

Put this into your `$HOME/.zshrc` to call `nvm use` automatically whenever you enter a directory that contains an
`.nvmrc` file with a string telling nvm which node to `use`:

```zsh
# place this after nvm initialization!
autoload -U add-zsh-hook
load-nvmrc() {
  local nvmrc_path="$(nvm_find_nvmrc)"

  if [ -n "$nvmrc_path" ]; then
    local nvmrc_node_version=$(nvm version "$(cat "${nvmrc_path}")")

    if [ "$nvmrc_node_version" = "N/A" ]; then
      nvm install
    elif [ "$nvmrc_node_version" != "$(nvm version)" ]; then
      nvm use
    fi
  elif [ -n "$(PWD=$OLDPWD nvm_find_nvmrc)" ] && [ "$(nvm version)" != "$(nvm version default)" ]; then
    echo "Reverting to nvm default version"
    nvm use default
  fi
}
add-zsh-hook chpwd load-nvmrc
load-nvmrc
```

Now when you switch to your project directory

```sh
$ cd project_xyz
Found '/home/user/project_xyz/.nvmrc' with version <v18.6.0>
Now using node v18.6.0 (npm v9.2.0)

$ cd ..
Reverting to nvm default version
Now using node v18.12.1 (npm v8.19.2)
```

Refer [Deeper Shell Integration](https://github.com/nvm-sh/nvm/blob/master/README.md#deeper-shell-integration) section for other shells

## Benefits of nvm & .nvmrc

- **Stability:** Your projects won't break when you install newer versions of node as .nvmrc file saves the required node version for that project.

- **Explicit:** Other developers or users of your application will instantly see the needed node version from the .nvmrc in the root of the project.

- **Speed:** With shell integration you can almost not even think about switching node versions as long as a .nvmrc file exists for the project and manual intervention is not required.

**TIP:**
Save v18.x.x instead of saving the exact version like v18.12.1 in **.nvmrc** file. This will ensure then that whenever minor security updates are release you don't have to worry about upgrading them into your project.

## Conclusion

NVM is one of the most helpful tools and using it with .nvmrc files will make a better Javascript development experience.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

![](https://cdn-images-1.medium.com/max/1600/0*dMZ0BEHDv4MJYYGW.png)

> If you liked the above story, you can buy me a coffee to keep me energized for writing stories like this for you.