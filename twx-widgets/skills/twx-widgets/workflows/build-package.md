# Workflow: Build and Package a Widget Extension

Compile, verify, and package a widget extension for deployment to ThingWorx.

## Build Commands

| Command | When to use |
|---------|-------------|
| `npm run testbuild` | Quick compilation check during development |
| `npm run buildDev` | Full build + package with source maps (for testing) |
| `npm run build` | Production build + package (for deployment) |

Upload is the user's responsibility. Do not run `npm run upload` unless the user explicitly asks. When they do, ensure `.env` is configured with valid ThingWorx credentials first. For debugging iterations, the user may prefer `npm run upload` for speed; for production deployments, manual import via Administration > Extensions > Import is safer.

## Step 1: Verify the project

Before building, check:
1. `package.json` exists with `packageName` and `version` fields
2. `webpack.config.js` exists
3. `node_modules/` exists (run `npm install` if not)
4. `src/` contains at least one widget directory

## Step 2: Development build

```bash
cd {{projectPath}} && npm run testbuild
```

### Check for errors

**Common compilation errors:**

| Error | Cause | Fix |
|-------|-------|-----|
| `Cannot find module 'typescriptwebpacksupport'` | Missing dependency | `npm install` |
| `Experimental decorators` | tsconfig missing setting | Add `"experimentalDecorators": true` |
| `Property does not exist on type 'TWComposerWidget'` | Wrong import path | Use `widgetIDESupport` for IDE, `widgetRuntimeSupport` for runtime |
| `Cannot find module '*.html'` | Missing type declaration | Create `src/@types/assets/index.d.ts` |
| `Unexpected token '<'` (in .tsx) | Missing JSX config | Add `"jsx": "react-jsx"` to tsconfig, add `@babel/preset-react` |

### Verify output

Check that the expected files exist in `dist/`:
```bash
ls -la dist/ui/{{widgetname}}/
```

Expected files per widget:
- `{{widgetname}}.ide.js` — Composer code
- `{{widgetname}}.runtime.js` — Runtime code
- `{{widgetname}}.ide.css` — Composer styles
- `{{widgetname}}.runtime.css` — Runtime styles
- `*.chunk.js` — Lazy-loaded chunks (if dynamic imports are used)
- `static/` — Copied static assets (if any)

## Step 3: Package

```bash
npm run buildDev  # or: npm run build (for production)
```

This runs two webpack passes:
1. **Build pass**: Compiles all widgets
2. **Package pass**: Generates `metadata.xml` and creates the zip

### Verify metadata.xml

```bash
cat dist/metadata.xml
```

Check that:
- `ExtensionPackage` has correct `name`, `version`, `vendor`
- Each widget has a `<Widget>` entry with `UIResources` listing all 4 file types (IDE JS, Runtime JS, IDE CSS, Runtime CSS)

### Verify the zip

```bash
ls -la build/distributions/
```

The zip file name follows the pattern: `{{PackageName}}-[debug-]{{version}}.zip`

## Step 4: Verify zip contents

```bash
unzip -l build/distributions/{{PackageName}}*.zip
```

Check that the zip contains:
- `metadata.xml` at the root of `ui/` parent
- `ui/{{widgetname}}/` with all widget files
- `Entities/` with ThingWorx entity definitions (if any)
- All chunk files (if lazy loading is used)

## Step 5: Size check

For production builds, check the runtime file sizes to ensure CombinedExtensions.js impact is minimal:

```bash
ls -lh dist/ui/*//*.runtime.js
```

If a runtime file is larger than ~50KB, investigate:
- Is a large library bundled directly? It should be lazy-loaded
- Are there unnecessary imports?
- Is tree-shaking working? (check webpack externals)

## Step 6: Report to user

Tell the user:
- Build status (success/failure)
- Zip location: `build/distributions/{{PackageName}}-{{version}}.zip`
- File sizes for runtime bundles
- Any warnings or issues noticed
- How to import into ThingWorx: Administration > Extensions > Import

## Troubleshooting

### Build hangs
- Check for circular imports between widgets
- Try building a single widget by temporarily removing others from `src/`

### metadata.xml is empty or wrong
- Check `widgetMetadataGeneratorPlugin.js` can read `src/` directories
- Verify `package.json` has `author`, `description`, `version`, `minimumThingWorxVersion`

### Zip is too large
- Check `dist/` for unnecessary files (leftover from previous builds)
- Verify external libraries are lazy-loaded, not bundled
- Clean and rebuild: `rm -rf dist/ build/ && npm run build`

### Chunk files missing from zip
- The `FileManagerPlugin` archives everything in `dist/`
- Verify chunks exist in `dist/ui/{{widgetname}}/` after the build pass
- Check `chunkFilename` in webpack config
