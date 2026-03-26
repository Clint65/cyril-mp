# Webpack Plugins Templates

These plugins are shared across all widget projects. Copy them to `webpack/` in the project root.

## webpack.config.js (Entry Point)

```javascript
export default async (env, argv) => {
  if (env.package) {
    console.log("Packaging ...");
    const { default: pkg } = await import("./webpack/webpack.package.js");
    return pkg(env, argv);
  } else {
    console.log("Building ...");
    const { default: build } = await import("./webpack/webpack.common.js");
    return build(env, argv);
  }
};
```

## moduleSourceUrlUpdaterPlugin.js

Fixes source map URLs in development mode for proper Chrome DevTools debugging:

```javascript
import pkg from "webpack-sources";
import { Compilation } from "webpack";

class ModuleSourceUrlUpdaterPlugin {
  constructor(options) {
    this.options = options;
  }

  apply(compiler) {
    const { RawSource } = pkg;
    const options = this.options;
    compiler.hooks.compilation.tap("ModuleSourceUrlUpdaterPlugin", (compilation) => {
      compilation.hooks.processAssets.tap(
        {
          name: "ModuleSourceUrlUpdaterPlugin",
          stage: Compilation.PROCESS_ASSETS_STAGE_ADDITIONS,
        },
        () => {
          for (const chunk of compilation.chunks) {
            for (const file of chunk.files) {
              compilation.updateAsset(file, (old) => {
                return new RawSource(
                  old.source().replace(
                    /\/\/# sourceURL=webpack-internal:\/\/\//g,
                    "//# sourceURL=webpack-internal:///" + options.context + "/"
                  )
                );
              });
            }
          }
        }
      );
    });
  }
}

export default ModuleSourceUrlUpdaterPlugin;
```

## webpack.package.js

Packaging configuration — generates metadata.xml and creates the deployment zip. Place in `webpack/webpack.package.js`:

```javascript
import fs from "fs";
import { CleanWebpackPlugin } from "clean-webpack-plugin";
import dotenv from "dotenv";
import path from "path";
import { fileURLToPath } from "url";
import UploadToThingworxPlugin from "./uploadToThingworxPlugin.js";
import FileManagerPlugin from "filemanager-webpack-plugin";
import WidgetMetadataGenerator from "./widgetMetadataGeneratorPlugin.js";

const packageJson = JSON.parse(fs.readFileSync("./package.json"));

export default (env, argv) => {
  const __filename = fileURLToPath(import.meta.url);
  const __dirname = path.dirname(__filename);

  const uploadEnabled = env ? env.upload : false;
  const packageName = packageJson.packageName || `${packageJson.name}_ExtensionPackage`;
  dotenv.config({ path: ".env" });
  const isProduction = argv.mode == "production";
  const zipFile = `${packageName}-${isProduction ? "" : "debug-"}${packageJson.version}.zip`;

  const result = {
    entry: {},
    output: {
      path: path.resolve(__dirname, "build"),
    },
    plugins: [
      new CleanWebpackPlugin({
        cleanOnceBeforeBuildPatterns: [path.resolve("build/**")],
      }),
      // Scans src/ and dist/ to generate dist/metadata.xml
      new WidgetMetadataGenerator({ packageName, packageJson }),
      // Archives dist/ into a deployable zip
      new FileManagerPlugin({
        events: {
          onEnd: {
            mkdir: ["./zip"],
            archive: [{ source: "./dist", destination: `./build/distributions/${zipFile}` }],
          },
        },
      }),
    ],
  };

  if (uploadEnabled) {
    result.plugins.push(
      new UploadToThingworxPlugin({
        thingworxServer: process.env.TARGET_THINGWORX_SERVER,
        thingworxUser: process.env.TARGET_THINGWORX_USER,
        thingworxPassword: process.env.TARGET_THINGWORX_PASSWORD,
        packageVersion: packageJson.version,
        packageName: packageName,
        isProduction: isProduction,
      })
    );
  }

  return result;
};
```

## widgetMetadataGeneratorPlugin.js

Auto-generates `dist/metadata.xml` from the project structure:

```javascript
import xml2js from "xml2js";
import path from "path";
import fs from "fs";
import { glob } from "glob";

const XML_FILE_TEMPLATE = `
<?xml version="1.0" encoding="UTF-8"?><Entities>
<ExtensionPackages>
  <ExtensionPackage name="" description="" vendor="" packageVersion="" minimumThingWorxVersion="" buildNumber=""/>
</ExtensionPackages>
<Widgets>
  <Widget name="">
    <UIResources/>
  </Widget>
</Widgets>
</Entities>
`;

class WidgetMetadataGenerator {
  constructor(options) {
    this.options = options;
  }

  apply(compiler) {
    compiler.hooks.afterEmit.tap("WidgetMetadataGeneratorPlugin", () => {
      const options = this.options;
      xml2js.parseString(XML_FILE_TEMPLATE, function (err, result) {
        if (err) console.log("Error parsing metadata file" + err);
        const attrs = result.Entities.ExtensionPackages[0].ExtensionPackage[0].$;
        attrs.name = options.packageName;
        attrs.description = options.packageJson.description;
        attrs.vendor = options.packageJson.author;
        attrs.minimumThingWorxVersion = options.packageJson.minimumThingWorxVersion;

        const fullVersion = options.packageJson.version;
        const dashIndex = fullVersion.indexOf("-");
        if (dashIndex !== -1) {
          attrs.packageVersion = fullVersion.substring(0, dashIndex);
          attrs.buildNumber = fullVersion.substring(dashIndex + 1);
        } else {
          attrs.packageVersion = fullVersion;
          if (options.packageJson.autoUpdate) {
            attrs.buildNumber = JSON.stringify(options.packageJson.autoUpdate);
          }
        }

        const getDirectories = (source) =>
          fs.readdirSync(source, { withFileTypes: true })
            .filter((d) => d.isDirectory() && d.name !== "@types" && d.name !== "common")
            .map((d) => d.name);

        const widgets = getDirectories("./src/");
        result.Entities.Widgets[0].Widget = [];

        widgets.forEach((widget) => {
          const js = glob.sync("./src/" + widget + "/*.{ts,tsx,js}");
          const css = glob.sync("./dist/ui/" + widget + "/*.css");
          let xmlWidget = {};

          if (js.length > 0) {
            xmlWidget = {
              $: { name: widget },
              UIResources: {
                FileResource: [
                  { $: { type: "JS", file: `${widget}.ide.js`, description: "", isDevelopment: "true", isRuntime: "false" } },
                  { $: { type: "JS", file: `${widget}.runtime.js`, description: "", isDevelopment: "false", isRuntime: "true" } },
                  { $: { type: "CSS", file: `${widget}.ide.css`, description: "", isDevelopment: "true", isRuntime: "false" } },
                  { $: { type: "CSS", file: `${widget}.runtime.css`, description: "", isDevelopment: "false", isRuntime: "true" } },
                ],
              },
            };
          }

          if (css.length > 0) {
            if (Object.keys(xmlWidget).length === 0) {
              xmlWidget = { $: { name: widget }, UIResources: { FileResource: [] } };
            }
            css.forEach((file) => {
              if (!file.includes("runtime.css") && !file.includes("ide.css")) {
                xmlWidget.UIResources.FileResource.push({
                  $: { type: "CSS", file: path.basename(file), description: "", isDevelopment: "true", isRuntime: "true" },
                });
              }
            });
          }

          result.Entities.Widgets[0].Widget.push(xmlWidget);
        });

        const xml = new xml2js.Builder().buildObject(result);
        fs.writeFile("dist/metadata.xml", xml, function (err) {
          if (err) return console.log(err);
        });
      });
    });
  }
}

export default WidgetMetadataGenerator;
```

## uploadToThingworxPlugin.js

Uploads the packaged extension to a ThingWorx server:

```javascript
import got from "got";
import FormData from "form-data";
import fs from "fs";
import path from "path";

class UploadToThingworxPlugin {
  constructor(options) {
    this.options = options;
    if (!this.options.thingworxServer) {
      throw "No target ThingWorx server declared for upload. Create a .env file";
    }
    if (this.options.thingworxServer.endsWith("/")) {
      this.options.thingworxServer = this.options.thingworxServer.slice(0, -1);
    }
    this.authorizationHeader =
      "Basic " + Buffer.from(`${this.options.thingworxUser}:${this.options.thingworxPassword}`).toString("base64");
  }

  apply(compiler) {
    compiler.hooks.done.tap("UploadToThingworxPlugin", async () => {
      console.info("Starting widget upload");
      try {
        await this.deleteExtension();
      } catch (ex) {
        console.warn(`Failed to delete extension: ${ex}. Not critical.`);
      }
      try {
        await this.importExtension();
        console.log(`Uploaded widget version ${this.options.packageVersion} to Thingworx!`);
      } catch (ex) {
        console.error(`Failed to import extension: ${ex}`);
      }
    });
  }

  async deleteExtension() {
    return await got.post({
      url: `${this.options.thingworxServer}/Thingworx/Subsystems/PlatformSubsystem/Services/DeleteExtensionPackage`,
      headers: {
        "X-XSRF-TOKEN": "TWX-XSRF-TOKEN-VALUE",
        Authorization: this.authorizationHeader,
      },
      json: { packageName: this.options.packageName },
      responseType: "json",
    });
  }

  async importExtension() {
    const form = new FormData();
    const zipName = `${this.options.packageName}-${this.options.isProduction ? "" : "debug-"}${this.options.packageVersion}.zip`;
    form.append("file", fs.createReadStream(path.join(process.cwd(), "build/distributions", zipName)));
    return await got.post({
      url: `${this.options.thingworxServer}/Thingworx/ExtensionPackageUploader?purpose=import`,
      headers: {
        "X-XSRF-TOKEN": "TWX-XSRF-TOKEN-VALUE",
        Authorization: this.authorizationHeader,
      },
      body: form,
    });
  }
}

export default UploadToThingworxPlugin;
```
