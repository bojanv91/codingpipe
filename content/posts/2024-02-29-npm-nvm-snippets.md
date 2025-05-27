---
title: "NPM and NVM Command Snippets"
date: 2024-02-29
dateUpdated: Last Modified
permalink: /posts/npm-nvm-cheatsheet/
tags:
  - Snippets
  - JavaScript
layout: layouts/post.njk
---

## Installing node.js and npm in Windows via NVM using Chocolatey

Start Command Prompt as Administrator and run this command:

```powershell
choco install nvm

nvm install latest  # or you can replace 'latest' with a specific node.js version 
```

## Using NVM

Display the currently active version:

```powershell
nvm current
```

Install node.js in specific version or in 'latest':

```powershell
nvm install <version>
```

List the node.js installations you can choose in `nvm use`:

```powershell
nvm list
```

Switch to use the specified version:

```powershell
nvm use <version>
```

Examples: switch to use different versions:

```powershell
nvm use 21.6.2
npm install     # uses the 21.6.2 version
nvm use 18.16.0
npm install     # uses the 18.16.0 version
nvm use latest
npm install     # uses the latest version
```

## Using NPM

Install everything in project's `package.json`:

```
npm install 
```

List the installed versions of all dependencies in this project:

```
npm list
```

List only the outdated depedencies in this project:

```
npm outdated
```

## Additional Resources

- <https://github.com/coreybutler/nvm-windows>
- <https://community.chocolatey.org/packages/nvm>
- <https://github.com/nvm-sh/nvm>
