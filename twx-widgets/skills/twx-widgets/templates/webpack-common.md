# webpack.common.js Template

This is the main build configuration. Place in `webpack/webpack.common.js`.

Adapt externals, aliases, and loaders based on the project's specific needs. Read the closest example project in `~/Documents/01_Sources/01_Databox/01_Widgets/` to match any project-specific patterns.

## TypeScript Project

```javascript
import webpack from "webpack";
import path from "path";
import glob from "glob";
import fs from "fs";
import TerserPlugin from "terser-webpack-plugin";
import CopyWebpackPlugin from "copy-webpack-plugin";
import MiniCssExtractPlugin from "mini-css-extract-plugin";
import ModuleSourceUrlUpdaterPlugin from "./moduleSourceUrlUpdaterPlugin.js";
import RemoveEmptyScriptsPlugin from "webpack-remove-empty-scripts";
import * as sass from "sass";

const packageJson = JSON.parse(fs.readFileSync("./package.json"));

export default (env, argv) => {
  const packageName = packageJson.packageName || `${packageJson.name}_ExtensionPackage`;
  const distDir = "./dist/";
  if (fs.existsSync(distDir)) fs.rmSync(distDir, { recursive: true, force: true });

  const isProduction = argv.mode == "production";

  const getDirectories = (source) =>
    fs.readdirSync(source, { withFileTypes: true })
      .filter((d) => d.isDirectory() && d.name !== "@types" && d.name !== "common")
      .map((d) => d.name);

  const widgets = getDirectories("./src/");
  const configs = [];

  widgets.forEach((widget) => {
    const js = glob.sync("./src/" + widget + "/*.{ts,tsx,js}");
    const sassOnly = glob.sync("./src/" + widget + "/*.scss");
    if (js.length > 0) {
      configs.push(getWidgetConfig(packageName, widget, isProduction));
    } else if (sassOnly.length > 0) {
      configs.push(getSassConfig(packageName, widget, isProduction));
    }
  });

  return configs;
};

const getWidgetConfig = (packageName, widget, isProduction) => {
  const result = {
    entry: glob.sync("./src/" + widget + "/*.{ts,tsx,js}").reduce((acc, filePath) => {
      let entry = filePath.split("/")[3].replace(/\.(ts|tsx|js)$/, "");
      acc[entry] = filePath;
      return acc;
    }, {}),

    output: {
      path: path.join(process.cwd(), "dist", "ui", widget),
      filename: "[name].js",
      chunkFilename: "[name].chunk.js",
      chunkLoadingGlobal: `webpackJsonp${packageName}`,
      publicPath: `/Thingworx/Common/extensions/${packageName}/ui/${widget}/`,
      libraryTarget: "window",
      devtoolNamespace: packageName,
    },

    externals: {
      jquery: "jQuery",
      moment: "moment",
    },

    plugins: [
      new MiniCssExtractPlugin(),
      new webpack.ProvidePlugin({
        $: "jquery",
        jquery: "jQuery",
        "window.jQuery": "jquery",
      }),
      new CopyWebpackPlugin({
        patterns: [
          { from: `src/${widget}/static`, to: "static", noErrorOnMissing: true },
          { from: "Entities/**/*.xml", to: "../../", noErrorOnMissing: true },
        ],
      }),
    ],

    devtool: isProduction ? undefined : "eval-source-map",

    resolve: {
      extensions: [".ts", ".tsx", ".js", ".json"],
    },

    module: {
      rules: [
        {
          test: /\.(jsx|js)$/,
          exclude: /node_modules/,
          use: {
            loader: "babel-loader",
            options: {
              presets: ["@babel/preset-env"],
              plugins: [
                ["@babel/plugin-proposal-decorators", { decoratorsBeforeExport: true }],
                ["@babel/plugin-proposal-class-properties", { loose: true }],
              ],
            },
          },
        },
        // html-loader is REQUIRED — widgets import HTML templates as strings
        // (e.g., import widgetHtml from "./mywidget.html"). Do not remove.
        { test: /\.html$/, use: "html-loader" },
        { test: /\.tsx?$/, use: "ts-loader", exclude: /node_modules/ },
        { test: /\.(woff|woff2|eot|ttf|otf)$/, type: "asset/resource" },
        { test: /\.(png|jp(e*)g|svg|xml|gif)$/, type: "asset/resource" },
        { test: /\.css$/, use: [MiniCssExtractPlugin.loader, "css-loader"] },
        {
          test: /\.scss$/,
          use: [
            MiniCssExtractPlugin.loader,
            "css-loader",
            {
              loader: "sass-loader",
              options: {
                implementation: sass,
                webpackImporter: false,
                sassOptions: { includePaths: ["./node_modules"] },
              },
            },
          ],
        },
      ],
    },
  };

  if (isProduction) {
    result.mode = "production";
    result.optimization = {
      minimizer: [
        new TerserPlugin({
          parallel: true,
          terserOptions: {
            compress: true,
            mangle: true,
            toplevel: false,
            keep_fnames: true,
          },
        }),
      ],
    };
  } else {
    result.mode = "development";
    result.plugins.push(new ModuleSourceUrlUpdaterPlugin({ context: packageName }));
  }

  return result;
};

const getSassConfig = (packageName, widget, isProduction) => {
  const result = {
    entry: { styles: glob.sync("./src/" + widget + "/*.scss") },
    output: {
      path: path.join(process.cwd(), "dist", "ui", widget),
      publicPath: `../Common/extensions/${packageName}/ui/${widget}/`,
    },
    plugins: [
      new CopyWebpackPlugin({
        patterns: [
          { from: `src/${widget}/static`, to: "static", noErrorOnMissing: true },
          { from: "Entities/**/*.xml", to: "../../", noErrorOnMissing: true },
        ],
      }),
      new RemoveEmptyScriptsPlugin(),
    ],
    devtool: isProduction ? undefined : "eval-source-map",
    module: {
      rules: [
        {
          test: /\.scss$/,
          use: [
            { loader: "file-loader", options: { name: "[name].css" } },
            "extract-loader",
            "css-loader",
            "sass-loader",
          ],
        },
      ],
    },
  };

  if (isProduction) {
    result.optimization = {
      minimizer: [new TerserPlugin({ parallel: true, terserOptions: { compress: true, mangle: false, toplevel: false, keep_fnames: true } })],
    };
  } else {
    result.plugins.push(new ModuleSourceUrlUpdaterPlugin({ context: packageName }));
  }

  return result;
};
```

## React Project Additions

For React projects, add `@babel/preset-react` to the Babel presets:

```javascript
// In the babel-loader options:
presets: ["@babel/preset-env", "@babel/preset-react"],
```

And add emotion support if using MUI:
```javascript
// Add this rule for emotion CSS-in-JS:
{
  test: /\.(jsx|tsx)$/,
  exclude: /node_modules/,
  use: {
    loader: "babel-loader",
    options: {
      presets: ["@babel/preset-env", "@babel/preset-react"],
      plugins: [
        ["@babel/plugin-proposal-decorators", { decoratorsBeforeExport: true }],
        ["@babel/plugin-proposal-class-properties", { loose: true }],
      ],
    },
  },
},
```
