# Workflow: Create a New Widget Project

Create a complete widget extension project from scratch.

## Step 1: Gather requirements

Ask the user:
1. **Project name** — Will be used as the npm package name and ThingWorx extension name (PascalCase, e.g., `FCADMyWidgets`)
2. **Widget name** — Name of the first widget (lowercase, e.g., `fcadmywidget`)
3. **Widget type** — TypeScript or React+TypeScript?
4. **Description** — Short description for the package
5. **Target directory** — Where to create the project (default: `~/Documents/01_Sources/01_Databox/01_Widgets/`)
6. **External libraries** — Any large libraries needed (e.g., echarts, fabric.js)?

## Step 2: Study an existing example

Before generating code, read the closest example project to match conventions:

- **TypeScript widget**: Read files from `~/Documents/01_Sources/01_Databox/01_Widgets/databox-twx-displayprops/`
- **React widget**: Read files from `~/Documents/01_Sources/01_Databox/01_Widgets/databox-twx-react/`
- **Widget with large library**: Read files from `~/Documents/01_Sources/01_Databox/01_Widgets/databox-twx-echarts/`

Focus on: `package.json`, `webpack/webpack.common.js`, `tsconfig.json`, and one widget's source files.

## Step 3: Scaffold the project structure

Use the templates from `templates/project-scaffold.md` to create:

1. `package.json` — Based on the template, adapted with project-specific dependencies
2. `tsconfig.json` — Standard config (add `jsx: "react-jsx"` for React projects)
3. `webpack.config.js` — Entry point router
4. `.gitignore`
5. `.env` — With placeholder credentials

## Step 4: Create webpack plugins

Copy the webpack plugins from `templates/webpack-plugins.md`:

1. `webpack/webpack.common.js` — **Adapt from the closest example project**, not from the template. The template is a reference, but real projects may have specific aliases, loaders, or externals
2. `webpack/webpack.package.js`
3. `webpack/moduleSourceUrlUpdaterPlugin.js`
4. `webpack/widgetMetadataGeneratorPlugin.js`
5. `webpack/uploadToThingworxPlugin.js`

## Step 5: Create the first widget

Use the appropriate widget template:
- **TypeScript**: `templates/ts-widget.md`
- **React**: `templates/react-widget.md`

Create the widget files in `src/{{widgetname}}/`:
- `{{widgetname}}.ide.ts`
- `{{widgetname}}.runtime.ts` (or `.tsx` for React)
- `{{widgetname}}.html`
- `{{widgetname}}.ide.scss`
- `{{widgetname}}.runtime.scss` (or `.css`)
- `images/icon.png` — Use a placeholder or copy from an example

For React projects, also create `src/common/`:
- `vendor-core.ts` — Import shared dependencies (see `references/vendor-splitting.md`)
- `vendor-loader.ts` — Caching loader functions
- `theme.ts` — MUI theme configuration (optional)

## Step 6: Create type declarations

Create `src/@types/assets/index.d.ts` with module declarations for `.png`, `.html`, `.svg` imports.

## Step 7: Install dependencies and verify build

Run:
```bash
cd {{projectPath}} && npm install
```

Then verify the build compiles:
```bash
npm run testbuild
```

Check the output:
- `dist/ui/{{widgetname}}/{{widgetname}}.ide.js` exists
- `dist/ui/{{widgetname}}/{{widgetname}}.runtime.js` exists
- No compilation errors

Then verify packaging:
```bash
npm run buildDev
```

Check:
- `dist/metadata.xml` exists and contains the widget definition
- `build/distributions/{{PackageName}}-debug-{{version}}.zip` exists

## Step 8: Present the result

Tell the user:
- The project is ready at `{{projectPath}}`
- How to build: `npm run buildDev`
- How to upload: Configure `.env` with ThingWorx credentials, then `npm run upload`
- The zip file location for manual import into ThingWorx
