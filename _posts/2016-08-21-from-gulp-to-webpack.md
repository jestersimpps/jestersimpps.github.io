---
layout: post
title: "From Gulp to Webpack"
description: "A practical guide to migrating your Angular 2 project from Gulp/SystemJs to Webpack for better performance and developer experience."
date: 2016-08-21
author: "Jester Simpps"
categories: [Development]
tags: [webpack, gulp, angular2, javascript, build-tools]
image: /images/gg-vs-webpack.png
---

I recently started working on a big Angular 2 webapp. The company I joined has been working on this project since Oktober 2015. It is one of the first companies in Belgium that jumps on the ng2 bandwagon and it has been a joy and challenge to learn and work with this framework every day.

Because gulp experience was already in house, the team has implemented a gulp build coupled with SystemJs. These days (August 2016), partly popularized by React, [Webpack](https://webpack.github.io/) has become a 'thing'. My colleagues at the company have been following the trend and have decided to switch to [Webpack](https://webpack.github.io/) for their next project. Secretly, there's a common desire to implement it in the current project as well, but... there is this tight deadline, so...

...I decided to give it a shot myself and tried to hack something together this Saturday and see how far I would get. To be honest, I didn't know what to expect myself. I've heard all the buzz but haven't actually given Webpack any time as I have been working relentlessly on my own startup [krackzee](http://www.krackzee.com/search?sortby=distance&order=&radius=50&classes=11th%20Commerce&locality=Sector%2025&city=Pune#) over the past 8 months, which is a php Laravel project. So I have been 'out' of the front-end world for quite some time. To all non-javascript developers; 8 months in the javascript realm is like a decade in your language. ([Read: A JS framework on every table by Allen Pike](https://www.allenpike.com/2015/javascript-framework-fatigue/))

![javascript fatigue](https://www.allenpike.com/images/2015/cube-drone-angular.jpg)

## Webpack

### First impressions

The installation has a lot of simalarity to installing grunt or gulp. You pull in some repositories, make sure you save them as devDependencies in your package.json and create some config files to configure your build process. Here's the interesting part though; It's almost like a plug and play build process tool. Forget about writing those [insane](http://jestersimpps.github.io/experimenting-with-google-spreadsheets-assemble-io-and-internationalisation/), customized grunt and gulp scripts. Here's a tool that just works out of the box with minimal configuration. And it works well!

### Getting started

The [Angular 2](https://angular.io/docs/ts/latest/guide/webpack.html) team has done a pretty good job explaining Webpack on their website. It has also become their build tool of choice. I started by basically going through their guide and adapting their implementation in a seperate branch to our current project. Don't do this! I made this mistake and I might have learned a bit more about what the Webpack config files do, but it will fail miserably unless you have implemented webpack before and you know how everything works together. My suggestion is to start a different repo alltogether with a clean version of one of the [Angular 2 webpack starters](https://github.com/search?utf8=%E2%9C%93&q=angular+2+webpack+starter&type=Repositories&ref=searchresults) out there. Just browse [Github](http://github.com) a bit, there's plenty of them.

I chose for the [angular2-webpack-starter](https://github.com/AngularClass/angular2-webpack-starter) from [AngularClass](https://angularclass.com/)

### Clone your favorite starter

Start by cloning your favorite Webpack starter repo somewhere on your hard drive.

```bash
git clone https://github.com/AngularClass/angular2-webpack-starter.git
cd angular2-webpack-starter
npm i
```

If you choose to use the same starter as me, make sure you have Node version >= 5.0 and NPM >= 3 as mentioned in their readme.

### Move your code

After having a local version of the starter, I removed all of their awesomely, perfected, genius code from the `src/app` folder and copied in our own. Remove the git directory if you haven't yet, and set up your own repo. Maybe make a first commit and push. From this moment onward there's a lot of nitty gritty changes to the Webpack configuration, but basically you are almost done!

### Reconfigure Webpack

#### Folders and files

Depending on how you have set up your Angular 2 application you will have a different `main.ts` file.(assuming you are coding Typescript, I'll just ignore anyone who isn't) The `main.ts` file is that file that holds your main app component (everything starts from there, your JS app big bang). Webpack best practice commands you to split it up into 3 files:

```javascript
entry: {

  'polyfills': './src/polyfills.browser.ts',
  'vendor': './src/vendor.browser.ts',
  'main': './src/main.browser.ts'

},
```

It's pretty obvious but, define your polyfill imports in the `src/polyfills.browser.ts` file, your vendor modules in the `src/vendor.browser.ts` file and bootstrap your app in the `src/main.browser.ts` file. I just left the polyfill file alone because it was mostly the same of what we already had. Then, in the `config/webpack.common.js` file, set your entry points as depicted above. In the platform folder, define your environment & provider settings if you have them.

Remove all script tags in your `index.html` file that point to any application code. These will get injected by a Webpack plugin called [HtmlWebpackPlugin](https://github.com/ampedandwired/html-webpack-plugin). Compare the index files of the starter with your own and basically resolve any merge conflicts :)

Set your own constants in the `config/webpack.common.js`:

```javascript
const METADATA = {
  title: 'Yourapp',
  baseUrl: '/',
  isDevServer: helpers.isWebpackDevServer()
};
```

If you work at a special company and have a designated designer that takes care of the scss for you, there's a chance you might have to set up some build configuration using the [ExtractTextPlugin](http://webpack.github.io/docs/stylesheets.html#separate-css-bundle) that enables you to use an import to define the to-be-converted scss files. (I will not cover this as we are currently moving our styling into a separate repo)

#### Environment config

Depending on your environment configuration you will have to make changes to the `config/webpack.dev.js`,`config/webpack.prod.js`,`config/webpack.test.js` files. We previously set the environment in a json during the gulp build. In the new Webpack configuration, I am using a constant I set in the 'DefinePlugin' that is then used in my code to determine the Api endpoint and such.

#### Testing config

If you were already using Karma and Jasmine, just compare the starter's `config/karma.conf.js` file with your own. Make changes to the already existing file - if any. Make sure you have a `karma.conf.js` pointing to the file in the config folder.

#### Troubleshooting

After all this, cross your fingers and run the Webpack dev server using `npm start`. In my case there were issues because we are using html template files for our components and have referenced them to our app root directory.

Refer the HTML template files in relation to the current typescript file:

```typescript
@Component({
    selector: `component1`,
    templateUrl: './component1.component.tmpl.html'
})
```

instead of:

```typescript
@Component({
    selector: `component1`,
    templateUrl: 'app/../blablablabla/../component1.component.tmpl.html'
})
```

### Conclusion

#### In memory live reloading

Make the move asap, it won't take too much of your time! Especially if your grunt/gulp build process is currently not sporting any in memory live reloading. The time you save with this feature alone easily makes up for the time invested in figuring out implementing Webpack.

#### Lazy loading modules

Also, Webpack will pack your modules so that they will be lazy loaded (async) in your application. Often you don't need all of the dependencies at runtime. Lazy loading will boost your application significantly. Read more about it [here](http://survivejs.com/webpack/advanced-techniques/understanding-chunks/#lazy-loading-with-webpack).

Anyways, you won't be disappointed. Webpack is magic ;)
