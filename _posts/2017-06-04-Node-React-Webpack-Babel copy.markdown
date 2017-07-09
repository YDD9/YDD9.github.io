---
layout: post
title:  "Node-React-Webpack-Babel"
date:   2017-06-04 11:49:39 +0100
comments: true
categories: Node Javascript React webpack Babel ES6 
---




Following the React tutorial on [tutorialpoints.com](https://www.tutorialspoint.com/reactjs/reactjs_environment_setup.htm)

Issues you may run into while following the tutorial:

webpack.config.js './' relative path somehow has build error.   
module.loaders.loader must be specified as 'babel-loader' instead of 'babel'.  [further materials](https://stackoverflow.com/questions/43049748/invalid-configuration-object-in-webpack)

```
// const path = require('path')

var config = {
   entry: './main.js',
	
   output: {
      path: __dirname,  // path.resolve(__dirname,'',''),
      filename: 'index.js',
   },
	
   devServer: {
      inline: true,
      port: 8080
   },
	
   module: {
      loaders: [
         {
            test: /\.jsx?$/,
            exclude: /node_modules/,
            loader: 'babel-loader',
				
            query: {
               presets: ['es2015', 'react']
            }
         }
      ]
   }
}

module.exports = config;
```

babel installations:  
instal babel packages with option --save as well, save type efforts later troubleshoot.


After `npm start`, if there's error regarding npm version, you can update npm first, then delete node_modules folder, and install your packages again. [step explained](https://stackoverflow.com/questions/39959900/npm-start-error-with-create-react-app)
  
```
rm -rf node_modules
npm install -g npm@latest
npm install
```

[what is mount in react ?](https://stackoverflow.com/questions/31556450/what-is-mounting-in-react-js)  

[JS Bundler vs JS Task Runner](https://www.youtube.com/watch?v=wy3Pou3Vo04)  
first 14min of the video
* Bundler: Browserify, Webpack
    To compile module system(ES6, ECMAScript2015)
    To concat, minify into one file
* Task Runner: Gulp, Grunt

There's overlapping between bundler and task runner, but they are designed to more focus on their own strength.

### Webpack:
module bundling system 
platform independent
work with Node(npm)
mainly to transpile(compile) JS modules, but you need to config needed loader in webpack.config.js , i.e.:
    loader Babel for react.js

### demo:
15min - min of video

```
# make your app folder
mkdir myApp
cd myApp

# create package.json
npm init

npm install --save jquery
npm install --save-dev webpack
npm install --save-dev babel-core babel-loader babel-preset-2015

# [webpack.config.js creation](https://segmentfault.com/a/1190000004457636) and run
# 28min - 41min of video

```