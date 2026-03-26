# Workflow: Debug a Widget in the Browser

Debug a running ThingWorx widget using Chrome DevTools MCP.

## Prerequisites

### Chrome DevTools MCP Setup

The Chrome DevTools MCP server enables direct browser interaction from Claude Code.

**Installation:**

1. Add to Claude Code MCP settings (`.claude/settings.json` or project-level):
```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

3. Launch Chrome with remote debugging enabled:
```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
```

### Development Build

The widget must be built in **development mode** for meaningful debugging:
```bash
cd {{projectPath}} && npm run buildDev
```

Development mode enables:
- Source maps (`eval-source-map`)
- `ModuleSourceUrlUpdaterPlugin` for readable source URLs in Chrome DevTools
- Non-minified code with preserved function names

## Debug Workflow

### Step 1: Prepare the widget

Ask the user to:
1. Upload the development build to ThingWorx (`npm run upload` or manual import)
2. Open Chrome (with remote debugging port 9222)
3. Navigate to the mashup that contains the widget to debug
4. Confirm when the page is loaded and the widget is visible

### Step 2: Connect via Chrome DevTools MCP

Once the user confirms the page is loaded, use the Chrome DevTools MCP tools to:

1. **List available tabs** to find the ThingWorx mashup page
2. **Connect to the tab** running the mashup
3. **Inspect the DOM** to find the widget's elements

### Step 3: Investigate the issue

Common debugging scenarios:

#### Widget not rendering
- Check the console for JavaScript errors
- Verify the widget element exists in the DOM (search for `widget-{{widgetname}}`)
- Check if `afterRender()` was called (add a console.log or set a breakpoint)
- Verify data bindings are arriving (check `updateProperty` calls)

#### Data not displaying
- Inspect the property values: evaluate `TW.Runtime.Widgets` in the console to find the widget instance
- Check if bound data has arrived: look for the widget's internal state variables
- Verify the render guard conditions: all required data must be non-null

#### Styling issues
- Inspect the widget's DOM elements and computed styles
- Check for CSS conflicts with ThingWorx's own styles
- Verify CSS files are loaded (check Network tab)

#### Chunk loading failures
- Check the Network tab for 404 errors on `*.chunk.js` files
- Verify `publicPath` in webpack config matches the actual extension path
- Check the Console for webpack chunk loading errors

#### Event/service not working
- Verify the event is declared in both IDE and Runtime files
- Check the binding configuration in the Mashup Builder
- Add console.log in the event handler to confirm it fires

### Step 4: Apply fixes

Based on the debugging findings:
1. Modify the widget source code (see `workflows/modify-widget.md`)
2. Rebuild: `npm run testbuild`
3. Re-upload to ThingWorx
4. Ask the user to refresh the mashup page
5. Verify the fix via Chrome DevTools MCP

### Step 5: Iterate

Repeat the debug cycle until the issue is resolved. For persistent issues, consider:
- Adding temporary `console.log` statements at key lifecycle points
- Using Chrome DevTools breakpoints via the MCP
- Examining the ThingWorx runtime internals (`TW.Runtime.Widgets`)

## Useful Console Expressions

These can be evaluated via the Chrome DevTools MCP console:

```javascript
// List all widget instances on the mashup
Object.keys(TW.Runtime.Widgets)

// Get a specific widget instance
TW.Runtime.Widgets["{{widgetname}}-1"]

// Check a widget's properties
TW.Runtime.Widgets["{{widgetname}}-1"].getProperty("Data")

// Manually trigger a service on a widget
TW.Runtime.Widgets["{{widgetname}}-1"].serviceInvoked("Refresh")

// Check all loaded extensions
TW.Runtime.GetInstalledExtensions && TW.Runtime.GetInstalledExtensions()
```

## Debugging Tips

- **Source maps**: In Chrome DevTools Sources tab, look under `webpack-internal://{{PackageName}}/` for the original TypeScript source
- **Network tab**: Filter by the package name to see all loaded widget resources
- **Console filtering**: Use the package name as a filter to isolate widget logs
- **DOM breakpoints**: Set "subtree modifications" breakpoints on the widget container to catch unexpected DOM changes
