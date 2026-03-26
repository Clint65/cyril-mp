# Webpack Build Configuration Reference

The widget build pipeline uses a **two-pass Webpack strategy** with ES modules (`"type": "module"` in package.json).

**Why two passes?** The packaging step (Pass 2) needs to scan the built output files from Pass 1 to generate `metadata.xml` — it reads `dist/ui/` to discover which CSS and JS files were actually produced, including any generated chunks. Running both in one pass would mean the metadata generator runs before the files exist. The two-pass design also keeps concerns separated: Pass 1 is pure compilation (source → dist), Pass 2 is pure packaging (dist → zip).

## Architecture Overview

```
webpack.config.js          → Entry point: routes to common or package
webpack/webpack.common.js  → Pass 1: Builds widget source code
webpack/webpack.package.js → Pass 2: Generates metadata.xml and creates zip
webpack/moduleSourceUrlUpdaterPlugin.js  → Dev: fixes source URLs for debugging
webpack/widgetMetadataGeneratorPlugin.js → Generates metadata.xml from src/ structure
webpack/uploadToThingworxPlugin.js       → Optional: uploads zip to ThingWorx
```

## Entry Point (webpack.config.js)

Routes between build and package based on the `--env package` flag:

```javascript
export default async (env, argv) => {
  if (env.package) {
    const { default: pkg } = await import("./webpack/webpack.package.js");
    return pkg(env, argv);
  } else {
    const { default: build } = await import("./webpack/webpack.common.js");
    return build(env, argv);
  }
};
```

## Pass 1: webpack.common.js

### Multi-widget support
Scans `src/` for subdirectories (excluding `@types` and `common`). Each subdirectory is treated as a separate widget, producing its own build configuration:

```javascript
const widgets = fs.readdirSync("./src/", { withFileTypes: true })
  .filter(d => d.isDirectory() && d.name !== "@types" && d.name !== "common")
  .map(d => d.name);
```

### Entry points per widget
For each widget directory, all `*.ts` and `*.js` files become entry points. Typically this produces `{widget}.ide.js` and `{widget}.runtime.js`:

```javascript
entry: glob.sync("./src/" + widget + "/*.{ts,js}").reduce((acc, path) => {
  let entry = path.split("/")[3].replace(".ts", "").replace(".js", "");
  acc[entry] = path;
  return acc;
}, {})
```

### Output configuration

```javascript
output: {
  path: path.join(process.cwd(), "dist", "ui", widget),
  filename: "[name].js",
  chunkFilename: "[name].chunk.js",
  chunkLoadingGlobal: `webpackJsonp${packageName}`,
  publicPath: `/Thingworx/Common/extensions/${packageName}/ui/${widget}/`,
  libraryTarget: "window",
  devtoolNamespace: packageName
}
```

Key settings:
- **`publicPath`**: Critical for lazy loading. Must point to where ThingWorx serves extension files
- **`chunkFilename`**: Pattern for dynamically imported chunks
- **`chunkLoadingGlobal`**: Unique name to avoid conflicts between extensions
- **`libraryTarget: "window"`**: Exposes widget registration on the window object

### Externals
jQuery and moment are provided by ThingWorx — exclude them from the bundle:
```javascript
externals: {
  jquery: "jQuery",
  moment: "moment"
}
```

### Loaders

| Loader | Files | Purpose |
|--------|-------|---------|
| `ts-loader` | `*.ts`, `*.tsx` | TypeScript compilation |
| `babel-loader` | `*.js`, `*.jsx` | JS with decorator support |
| `sass-loader` | `*.scss` | SASS → CSS compilation |
| `css-loader` | `*.css` | CSS modules |
| `MiniCssExtractPlugin.loader` | CSS output | Extracts CSS to separate files |
| `html-loader` | `*.html` | Imports HTML as strings |
| Asset modules | images, fonts | Handles static assets |

### Babel configuration (for JS or React)
```javascript
{
  presets: ["@babel/preset-env"],  // Add "@babel/preset-react" for React
  plugins: [
    ["@babel/plugin-proposal-decorators", { decoratorsBeforeExport: true }],
    ["@babel/plugin-proposal-class-properties", { loose: true }]
  ]
}
```

### Development vs Production

**Development:**
- `devtool: "eval-source-map"` for debugging
- `ModuleSourceUrlUpdaterPlugin` for proper source mapping in Chrome

**Production:**
- TerserPlugin with `keep_fnames: true` (widget class names must be preserved)
- No source maps

### Plugins
- `MiniCssExtractPlugin` — Extracts CSS from JS bundles
- `webpack.ProvidePlugin` — Makes jQuery globally available
- `CopyWebpackPlugin` — Copies static assets and XML entities
- `RemoveEmptyScriptsPlugin` — Cleans up empty JS files from SCSS-only widgets
- `ModuleSourceUrlUpdaterPlugin` — Dev only: fixes source URLs for debugging

## Pass 2: webpack.package.js

Runs after build to package the extension:

1. **`WidgetMetadataGenerator`** — Scans `src/` directories and `dist/` outputs to generate `dist/metadata.xml`
2. **`FileManagerPlugin`** — Creates zip: `build/distributions/{packageName}-{version}.zip`
3. **`UploadToThingworxPlugin`** (optional, with `--env upload`) — Uploads zip to ThingWorx server using credentials from `.env`

### .env file for upload
```
TARGET_THINGWORX_SERVER=https://your-server.com
TARGET_THINGWORX_USER=Administrator
TARGET_THINGWORX_PASSWORD=yourpassword
```

## Build Commands (package.json scripts)

| Command | Description |
|---------|-------------|
| `npm run testbuild` | Development build only (no package) |
| `npm run build` | Production build + package |
| `npm run buildDev` | Development build + package (with source maps) |
| `npm run upload` | Development build + package + upload to ThingWorx |

## Output Structure

```
dist/
├── metadata.xml              ← Generated by WidgetMetadataGenerator
├── ui/
│   ├── widget1/
│   │   ├── widget1.ide.js
│   │   ├── widget1.runtime.js
│   │   ├── widget1.ide.css
│   │   ├── widget1.runtime.css
│   │   ├── *.chunk.js         ← Lazy-loaded chunks
│   │   └── static/            ← Copied static assets
│   └── widget2/
│       └── ...
└── Entities/                  ← ThingWorx XML entities (if any)

build/
└── distributions/
    └── PackageName-version.zip  ← Deployable extension
```
