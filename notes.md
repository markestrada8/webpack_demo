# WEBPACK NOTES

https://www.youtube.com/watch?v=3On5Z0gjf4U&list=PLblA84xge2_zwxh3XJqy6UVxS60YdusY8

https://webpack.js.org/

Initially using a file of JS code

After setting up ES6 import / export, we can set up webpack to "bundle" the code into an output asset, which is then the object file that will be run

### INITIAL CONFIGURATION
```
const path = require('path')

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "main.js",
    path: path.resolve(__dirname, "dist")
  }
}
```

### LOADERS

Process different file types for inclusion with the project code.

Handle '.css' file types with specific loader utilities:

- compile CSS as JS (last index)
- load JS style into document (<style></style>) (in reverse / countdown order)
```
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"]
      }
    ]
  }
```

Handle '.scss' file types with specific loader utilities:

(node-sass is deprecated and won't install anymore which I *think* is a node version issue - so we can use sass instead, and sass-loader ~7.2.0 will use sass instead of node-sass as a dependency)

```
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: ["style-loader", "css-loader", "sass-loader"]
      }
    ]
  }
```

### CACHE BUSTING

Prevent unwanted caching - filenames are marked with generated hash to determine if they have changed / whether they can be loaded from memory

```
  output: {
    filename: "main.[contentHash].js",
    path: path.resolve(__dirname, "dist")
  },
```

### PLUGINS

Dynamically inject the generated file names to be included in the index.html

Also generate the index.html from a template with specified additions such as reading from the output file after bundling

```
  plugins: [new HtmlWebpackPlugin({ template: "./src/template.html" })],

```

### SEPARATING DEVELOPMENT / PRODUCTION

webpack.common.js - common processes / configurations
webpack.dev.js - include dev server and dev utilities
webpack.prod.js - create minfied production bundle

```
webpack.dev.js -

const common = require('./webpack.common')

module.exports = merge(common, {
  ...configObject
  }
)
```

### VERSION CONFLICTS

More dependency issues with the various packages seemingly having changed after initial release instead of those changes being incorporated in new release versions, so looking at version history won't always resolve this because they were changed after the fact... which is not great.

With webpack, using LTS installs was the key to resolving this, and is probably the key to alot of version issues as I'm guessing some of those wafflings were worked out for the LTS dependencies.
```
  "devDependencies": {
    "bootstrap": "^4.6.2",
    "css-loader": "^2.1.0",
    "html-webpack-plugin": "^3.2.0",
    "sass": "^1.63.6",
    "sass-loader": "^7.3.1",
    "style-loader": "^0.23.1",
    "webpack": "^4.29.6",
    "webpack-cli": "^4.10.0",
    "webpack-dev-server": "^4.15.1",
    "webpack-merge": "^4.2.1"
  }
```

Along with:
```
  "scripts": {
    "start": "webpack serve --config webpack.dev.js",
    "build": "webpack --config webpack.prod.js"
  },
```

### MORE LOADERS

Move assets directory into src/ and update path in template

html-loader will update <img> tags to a require statement in the JS bundle

file-loader will handle the compilation of asset files during bundling

Config loader rules
```
      {
        test: /\.html$/,
        use: ["html-loader"]
      },
      {
        test: /\.(svg|png|jpg|gif)$/,
        use: {
          loader: "file-loader",
          options: {
            name: "[name].[hash].[ext]",
            outputPath: "images"
          }
        }
      }
```

### CLEANING BUILD FILES

Another plugin to wipe the build directory on each new build

const CleanWebpackPlugin = require('clean-webpack-plugin')
plugins: [new CleanWebpackPlugin()]

*NOTE: Keeping track of dev vs production dependencies can get tricky, but almost all of this so far is development only.

### MULTIPLE ENTRY POINTS

Handling multiple entry points for vendor and source code (this does not seem to matter either with newer versions of bootstrap or webpack, and it's unclear at this point how the determination is made on where and when a split would happen - it seems like bootstrap is just being delivered via CDN wholly and continuously)

```
webpack.common.js -

  entry: {
    main: "./src/index.js",
    vendor: "./src/vendor.js"
  },

webpack.prod.js

  output: {
    filename: "[name].[contentHash].bundle.js",
    path: path.resolve(__dirname, "dist")
  },

```

### EXTRACT CSS FOR PRODUCTION BUILD

Split out the css (after compiling it from scss) and inject it into the document rather than injecting the JS at the end of the process

```
webpack.prod.js -

  module: {
    rules: [
      {
        test: /\.scss$/,
        use: [
          MiniCSSExtractPlugin.loader,
          "css-loader",
          "sass-loader"
        ]
      }
    ]
  }

webpack.dev.js -

  module: {
    rules: [
      {
        test: /\.scss$/,
        use: [
          "style-loader", //3. Inject styles into DOM
          "css-loader", //2. Turns css into commonjs
          "sass-loader" //1. Turns sass into css
        ]
      }
    ]
  }
```

### MINIFY THE CSS CODE
```
const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin")

  optimization: {
    minimizer: [
      new OptimizeCSSAssetsPlugin()
    ]
  },
```

Then reset the minification of the JS code which has been now been overriden from default

```
const TerserPlugin = require('terser-webpack-plugin')

  optimization: {
    minimizer: [
      new OptimizeCSSAssetsPlugin(),
      new TerserPlugin()
    ]
  },
```

Then setup minification of html

```
const HtmlWebpackPlugin = require('html-webpack-plugin')


  new HtmlWebpackPlugin({
    template: "./src/template.html",
    minify: {
      removeAttributeQuotes: true,
      collapseWhitespace: true,
      removeComments: true
    }
  })
```


### VERSION CONFLICTS

More dependency issues with the various packages seemingly having changed after initial release instead of those changes being incorporated in new release versions, so looking at version history won't always resolve this because they were changed after the fact... which is not great.

With webpack, using LTS installs was the key to resolving this, and is probably the key to alot of version issues as I'm guessing some of those wafflings were worked out for the LTS dependencies.
```
  "devDependencies": {
    "bootstrap": "^4.6.2",
    "clean-webpack-plugin": "^2.0.0",
    "css-loader": "^2.1.0",
    "file-loader": "^3.0.1",
    "html-loader": "^0.5.5",
    "html-webpack-plugin": "^3.2.0",
    "mini-css-extract-plugin": "^0.5.0",
    "optimize-css-assets-webpack-plugin": "^5.0.1",
    "sass": "^1.63.6",
    "sass-loader": "^7.3.1",
    "style-loader": "^0.23.1",
    "webpack": "^4.29.6",
    "webpack-cli": "^4.10.0",
    "webpack-dev-server": "^4.15.1",
    "webpack-merge": "^4.2.1"
  }

```

Along with:
```
  "scripts": {
    "start": "webpack serve --config webpack.dev.js --open",
    "build": "webpack --config webpack.prod.js"
  },
```