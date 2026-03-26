# Project Scaffold Template

Use this template to scaffold a complete new widget extension project. Replace all `{{placeholders}}` with actual values.

## Directory Structure

```
{{projectName}}/
├── src/
│   ├── @types/
│   │   └── assets/
│   │       └── index.d.ts
│   └── {{widgetName}}/
│       ├── {{widgetName}}.ide.ts
│       ├── {{widgetName}}.runtime.ts
│       ├── {{widgetName}}.ide.scss
│       ├── {{widgetName}}.runtime.scss
│       ├── {{widgetName}}.html
│       └── images/
│           └── icon.png
├── Entities/                    (optional: ThingWorx XML entities)
├── webpack/
│   ├── webpack.common.js
│   ├── webpack.package.js
│   ├── moduleSourceUrlUpdaterPlugin.js
│   ├── widgetMetadataGeneratorPlugin.js
│   └── uploadToThingworxPlugin.js
├── webpack.config.js
├── package.json
├── tsconfig.json
├── .env                         (for upload credentials — add to .gitignore)
└── .gitignore
```

## package.json

```json
{
  "name": "{{PackageName}}",
  "version": "1.0.0",
  "description": "{{description}}",
  "author": "{{author}}",
  "type": "module",
  "minimumThingWorxVersion": "8.5.0",
  "packageName": "{{PackageName}}",
  "scripts": {
    "testbuild": "webpack --mode development",
    "build": "webpack --mode production && webpack --mode production --env package",
    "buildDev": "webpack --mode development && webpack --mode development --env package",
    "package": "webpack --env package",
    "upload": "webpack --mode development && webpack --mode development --env package --env upload",
    "postinstall": "patch-package"
  },
  "license": "ISC",
  "devDependencies": {
    "@babel/core": "^7.22.5",
    "@babel/plugin-proposal-class-properties": "^7.18.6",
    "@babel/plugin-proposal-decorators": "^7.21.0",
    "@babel/preset-env": "^7.20.2",
    "@types/jquery": "^3.5.16",
    "@types/node": "^22.2.0",
    "@types/webpack-env": "^1.18.0",
    "babel-loader": "^9.1.2",
    "clean-webpack-plugin": "^4.0.0",
    "copy-webpack-plugin": "^12.0.2",
    "css-loader": "^7.1.2",
    "dotenv": "^16.0.3",
    "extract-loader": "^5.1.0",
    "file-loader": "^6.2.0",
    "filemanager-webpack-plugin": "^8.0.0",
    "form-data": "^4.0.0",
    "got": "^14.4.2",
    "html-loader": "^5.1.0",
    "mini-css-extract-plugin": "^2.7.2",
    "patch-package": "^8.0.0",
    "sass": "^1.58.3",
    "sass-loader": "^16.0.0",
    "style-loader": "^4.0.0",
    "terser-webpack-plugin": "^5.3.0",
    "ts-loader": "^9.4.2",
    "typescript": "^5.1.6",
    "typescriptwebpacksupport": "^2.2.1",
    "webpack": "^5.75.0",
    "webpack-cli": "^5.0.1",
    "webpack-remove-empty-scripts": "^1.0.1",
    "webpack-sources": "^3.2.3",
    "xml2js": "^0.6.0"
  },
  "dependencies": {
    "typescriptwebpacksupport": "^2.2.1"
  },
  "peerDependencies": {
    "jquery": "^3.6.3"
  }
}
```

**For React projects**, add to `dependencies`:
```json
{
  "react": "^19.2.4",
  "react-dom": "^19.2.4",
  "@mui/material": "^7.3.9",
  "@mui/icons-material": "^7.3.9",
  "@emotion/react": "^11.0.0",
  "@emotion/styled": "^11.0.0"
}
```
And add to `devDependencies`:
```json
{
  "@babel/preset-react": "^7.22.5"
}
```

## tsconfig.json

```json
{
  "compilerOptions": {
    "outDir": "./dist/",
    "allowJs": true,
    "sourceMap": true,
    "noImplicitAny": false,
    "target": "es2017",
    "module": "es2015",
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "experimentalDecorators": true,
    "lib": ["es2017", "dom"],
    "typeRoots": [
      "./src/@types",
      "./node_modules/@types"
    ]
  },
  "include": ["./src/**/*"],
  "exclude": []
}
```

**For React projects**, add:
```json
{
  "compilerOptions": {
    "jsx": "react-jsx"
  }
}
```

## .gitignore

```
node_modules/
dist/
build/
.env
*.zip
```

## .env

```
TARGET_THINGWORX_SERVER=https://your-server.com
TARGET_THINGWORX_USER=Administrator
TARGET_THINGWORX_PASSWORD=yourpassword
```

## src/@types/assets/index.d.ts

```typescript
declare module "*.png" {
  const value: string;
  export default value;
}
declare module "*.jpg" {
  const value: string;
  export default value;
}
declare module "*.svg" {
  const value: string;
  export default value;
}
declare module "*.html" {
  const value: string;
  export default value;
}
```
