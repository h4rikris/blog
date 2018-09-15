---
layout: post
title: 'Devlog: Webpack config and case sensitive filesystems'
tags:
  - devlog
date: '2018-05-12 12:24 +0530'
published: true
---

I was setting up my dev machine to work on react project. Like any other react project all I have to do is run these commands.
```bash
git clone repo-url
yarn install
yarn dev # What ever the command it is to start a dev server
```
Then I can start of working on developing stories. Most of the times I've not faced any issues to run the react app until yesterday. I did `yarn install` then when I tried to run dev server, it started throwing errors at me.
Initially I hadn't looked closely on error thinking that it might be because I was using old version of `yarn` and `node`. I updated their versions, still it didn't solve the problem. Then I took a closer look at error.
The error was modules weren't able to resolve properly when webpack tried to compile.

We have our module structure like this
```
|
 -moduleName
  |
   - Index.js
  |
   - others.js
  ...
```
We import these modules as below
```js
import Module from './moduleName'
```
Webpack was not able to resolve above `import` statement and started showing error like below
```
ERROR in './app.jsx'
Module not found: Error: Can't resolve './moduleName' in ./src
```
Initially I thought this might be webpack configuration error because it was not able to resolve modules as directory with `Index.js` file. But with the same config whole team was able to run the app except me. I was like

>what's the problem with my machine?

Like any other developer I searched using those error messages and tried different solutions. But those solutions didn't work for me. From `webpack` documentation I found the `mainFiles` config field details.
I added following line to my webpack config
```js
resolve: {
  ...
  mainFiles: ['Index'],
  ...
}
```
This throwed me much bigger list of errors. This time modules in our app resolved properly but modules under `node_modules` hadn't resolved properly. Modules in `node_modules` uses general convention of having `index.js`. But in our app we used `Index.js` (Observe the starting letter capitalized).
Now I have two solutions
  - changing all `Index.js` files to `index.js` in our app or
  - Add `Index` and `index` both to `mainFiles` config field

I chose the second approach since it's just one line change. YESSS..**It worked**. In webpack documentation default value for `mainFiles` field is `['index']`. We have `Index.js` files in our modules as a entry point.
But still I didn't understand how does other team mates able to run the app without any changes to config. Then I realized that other team mates have `case-insensitive` file system in their laptops but I have `case-sensitive` file system. I verified it by using following commands.
```bash
diskutil info /
```
The real happiness is when you find a root cause of the problem.
