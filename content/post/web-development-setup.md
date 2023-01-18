---
layout: post
title: "A Beginner's Guide to Macbook setup for web development"
date: 2023-01-18 00:00:00 +0530
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1674035486/macbook_development_b0rlfv_vi6k9t.jpg']
tags: ['nodejs', 'javascript']
keywords: 'nodejs,nvm,macbook,web development,javascript,vs code,git,zsh'
categories: ['Javascript']
url: 'javascript/web-development-setup'
---

This is a beginner's guide to setup Macbook for web development. The motivation for writing this guide is to help people get started with programming on a Mac, and as also a personal reference for myself. If any mistakes/updates are there; do let me know...

## List of tools
- [iTerm2](#item_1)
- [Git](#item_2)
- [nvm & Nodejs](#item_3)
- [zsh & oh-my-zsh](#item_4)
- [VS Code](#item_5)
- [Google Chrome & Firefox](#item_6)
- [A Soft Murmur](#item_7)

## iTerm2 {#item_1}

I highly recommend to install [iterm2](https://iterm2.com/downloads.html) which is an alternative to Mac's default terminal.

Few settings which I find very useful

a) **New Session from previous let off:** iTerm Preferences → Profiles → select your profile → General tab → Working Directory section → Reuse previous session’s directory option

b) **Increase window size:** iTerm Preferences → Profiles → select your profile → Window tab → Settings for New Windows → Increase values of Columns (140) and Rows (40)

c) **Increase scrollback limit:** iTerm Preferences → Profiles → select your profile → Terminal tab → Scrollback lines → Increase the value to say 2000

d) **Open files in VS Code:** iTerm Preferences → Profiles → select your profile → Advanced → Semantic History → Open with editor → VS Code

## Git {#item_2}

- Follow steps from [Getting Started - Installing Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- validate with `git --version`

## nvm & Nodejs {#item_3}

Use the following cURL or Wget command:

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```
```sh
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

I have written a detailed blog ['Use NVM and .nvmrc for a better Javascript development'](https://techinsights.manisuec.com/nodejs/nvm-nodejs-nvmrc/)

## zsh & oh-my-zsh {#item_4}

- [Install oh-my-zsh now](https://ohmyz.sh/#install) 
- validate using `echo $SHELL` should print `/bin/zsh`

As of MacOS 10.15, `zsh` is the default over `bash`

## VS Code {#item_5}

- Refer [Installation](https://code.visualstudio.com/docs/setup/mac#_installation)
- Drag into your Applications folder before launching
- [Launching from the command line](https://code.visualstudio.com/docs/setup/mac#_launching-from-the-command-line)
- run `code .` from a project directory to launch

## Google Chrome & Firefox {#item_6}

- [Google Chrome](https://www.google.com/chrome/) is the defacto browser now for development. [React Devtools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en) is a very useful add-on
- [Firefox](https://www.mozilla.org/en-GB/firefox/new/) defacto number two browser

## A Soft Murmur {#item_7}

- [A Soft Murmur](https://asoftmurmur.com) Stream ambient sounds which helps relax & focus.

## Conclusion

You are all set up with your development setup. Enjoy the programming journey !

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.
