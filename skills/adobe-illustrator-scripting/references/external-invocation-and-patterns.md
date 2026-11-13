# External Invocation, Common Patterns, and Troubleshooting

## External Invocation & Argument Passing

Illustrator scripts are routinely launched from outside the application —
shell scripts, schedulers, build pipelines, ExtendScript Toolkit, or
`BridgeTalk` messages from other Creative Cloud apps. The execution
environment under those launchers differs from the in-application *File >
Scripts* path in several ways that frequently break otherwise-correct code.

### `arguments[]` Is Unreliable Under External Launchers

ExtendScript Toolkit's `-run` invocation and `BridgeTalk.send()` do not
forward arbitrary launcher arguments into the script's top-level
`arguments[]` array. In many configurations the array contains a single
`[object BridgeTalk]` element instead of the values the caller passed, as
demonstrated below:

```javascript
// At top of script
var passed = (typeof arguments !== "undefined") ? arguments : [];
for (var i = 0; i < passed.length; i++) {
    $.writeln("arg[" + i + "] = " + passed[i]);
    // Often prints: arg[0] = [object BridgeTalk]
}
```

**Do not rely on `arguments[]` for required inputs when the script is
launched externally.** Use one of the following more reliable channels.

### Sidecar File for Parameters

When a script fails under an external launcher and the source of the error
is not obvious, fall back to a sidecar file: have the caller write a small
text file at a known absolute path, and read it on startup. This works
regardless of launcher quirks and is easy to inspect after a failed run.

```javascript
var SIDECAR_PATH = "C:/Users/userName/job.args.txt";

function readSidecar(path) {
    var f = new File(path);
    if (!f.exists || !f.open("r")) return null;
    var lines = [];
    while (!f.eof) {
        var ln = f.readln();
        if (ln && !/^\s*$/.test(ln)) lines.push(ln);
    }
    f.close();
    return {
        input:  lines[0],
        output: lines[1],
        mode:   lines[2]
    };
}
```

A `key=value` format is equally workable and avoids positional fragility:

```text
input=C:/path/to/input.ai
output=C:/path/to/output.pdf
mode=preview
```

### Environment Variables

`$.getenv("NAME")` returns environment variables visible to **Illustrator's
process**, not the launcher's. If the launcher needs Illustrator to see a
value, it must set the variable system-wide or in Illustrator's parent
environment before launching. For per-invocation values, prefer a sidecar
file.

### `$.fileName` and `File($.fileName).parent`

Under in-application execution, `$.fileName` is the absolute path of the
running script and `File($.fileName).parent` yields the script's folder.
Under some external launchers (notably ESTK `-run`) `$.fileName` can be
empty, causing relative path resolution to silently fail.

```javascript
// Fragile: returns null under some launchers
var here = $.fileName ? File($.fileName).parent : null;
var sidecar = here ? new File(here.fsName + "/job.args.txt") : null;

// Robust: hardcode a known absolute path or fall back to a stable location
var sidecar = new File("C:/Users/userName/job.args.txt");
if (!sidecar.exists) sidecar = new File(Folder.temp.fsName + "/job.args.txt");
```

### Diagnostic Logging to an Absolute Path

Silent failures are common because dialogs are suppressed and the launcher
may not surface `$.writeln` output. Write a plain-text log to a known
absolute path so a run can be inspected after the fact. Create the parent
folder on demand so the first call cannot fail for a missing directory.

```javascript
var LOG_PATH = "C:/Users/userName/logs/job.log";

function log(msg) {
    try {
        var f = new File(LOG_PATH);
        try { if (!f.parent.exists) f.parent.create(); } catch (eDir) {}
        if (f.open("a")) {
            f.writeln("[" + new Date() + "] " + msg);
            f.close();
        }
    } catch (e) {}
}
```

### Wrap the Entry Point in `try { ... } catch`

Externally launched scripts often fail without any visible indication. A
top-level `try`/`catch` that writes the error to the log file converts
silent failures into a single inspectable line.

```javascript
try {
    main();
} catch (err) {
    log("FATAL: " + err + (err && err.line ? " line=" + err.line : ""));
}
```

### Suppress User Interaction

External callers cannot answer dialogs. Disable them before any DOM work
and avoid `alert()` / `confirm()` / `prompt()` entirely in scripts that may
be launched headlessly.

```javascript
app.userInteractionLevel = UserInteractionLevel.DONTDISPLAYALERTS;
```

### Save Explicitly

Closing or letting Illustrator return to its idle state does not save the
working file. After all DOM edits, call `doc.saveAs(...)` (or `doc.save()`)
explicitly and log whether it succeeded.

```javascript
var opts = new IllustratorSaveOptions();
opts.compatibility = Compatibility.ILLUSTRATOR17;
doc.saveAs(new File(doc.fullName.fsName), opts);
```

## Common Patterns

### Iterate All Page Items in a Document

```javascript
function processAllItems(doc) {
    for (var i = 0; i < doc.pageItems.length; i++) {
        var item = doc.pageItems[i];
        // Process based on type
        switch (item.typename) {
            case "PathItem":
                // handle path
                break;
            case "TextFrame":
                // handle text
                break;
            case "GroupItem":
                // handle group (may contain nested items)
                break;
        }
    }
}
```

### Recursively Unlock Layers and Groups Before Editing

A locked layer or any locked ancestor (parent group, clip group, sublayer)
will cause edits to throw `Error: Target layer cannot be modified`. Walk the
full hierarchy and clear `locked` / `hidden` flags before performing DOM
modifications.

```javascript
function unlockAll(doc) {
    function visitLayers(layers) {
        for (var i = 0; i < layers.length; i++) {
            var lyr = layers[i];
            try { lyr.locked = false; lyr.visible = true; } catch (e) {}
            visitItems(lyr);
            if (lyr.layers && lyr.layers.length) visitLayers(lyr.layers);
        }
    }
    function visitItems(container) {
        var items = container.pageItems;
        for (var j = 0; j < items.length; j++) {
            var it = items[j];
            try { it.locked = false; it.hidden = false; } catch (e) {}
            if (it.typename === "GroupItem") visitItems(it);
        }
    }
    visitLayers(doc.layers);
}
```

### Replacing the File Behind a Linked Image (Relink)

`PlacedItem.file = newFile` replaces a linked image while preserving the
parent, stacking order, and (after re-applying) the bounds. **`RasterItem`
does not expose a writable `file` property**, so when a placeholder is a
raster you must add a fresh `PlacedItem` in the same parent, copy the bounds,
then remove the original.

```javascript
function relinkOrRebuild(item, newFile) {
    var bounds = item.geometricBounds.slice();
    var parent = item.parent;
    var name   = item.name;

    if (item.typename === "PlacedItem") {
        item.file = newFile;
        item.geometricBounds = bounds;
        return item;
    }

    // RasterItem path: rebuild as a linked PlacedItem in the same parent.
    var fresh = parent.placedItems.add();
    fresh.file = newFile;
    fresh.geometricBounds = bounds;
    if (name) try { fresh.name = name; } catch (e) {}
    fresh.move(item, ElementPlacement.PLACEBEFORE);
    item.remove();
    return fresh;
}
```

### Placing SVG Content (Copy/Paste Pattern)

`PlacedItem.file` accepts raster formats and AI/PDF, **but not SVG**. Setting
it to an `.svg` File throws `Unable to set placed item's file, is the file
path provided valid?`. The reliable way to bring SVG artwork into a document
is to open the SVG as a separate document, select all, copy, close, and paste
into the working document.

```javascript
function placeSVG(targetDoc, svgFile, targetLayer) {
    var donor = app.open(svgFile);
    app.executeMenuCommand("selectall");
    app.executeMenuCommand("copy");
    donor.close(SaveOptions.DONOTSAVECHANGES);

    app.activeDocument = targetDoc;
    targetDoc.activeLayer = targetLayer;
    app.executeMenuCommand("pasteFront");

    var sel = targetDoc.selection;
    if (!sel || sel.length === 0) return null;
    if (sel.length === 1) return sel[0];

    // Multiple pasted items: group them so callers get a single handle.
    var group = targetLayer.groupItems.add();
    for (var i = sel.length - 1; i >= 0; i--) {
        sel[i].move(group, ElementPlacement.PLACEATBEGINNING);
    }
    return group;
}
```

### Finding a Clipping Path Inside a Mask Group

Clip groups expose their clipping shape as a child `PathItem` (or, less
commonly, a child of a `CompoundPathItem`) with `clipping === true`. The
clip's `geometricBounds` give the visible frame to size or center content
against.

```javascript
function findClipPath(group) {
    var items = group.pageItems;
    for (var i = 0; i < items.length; i++) {
        var it = items[i];
        try {
            if (it.typename === "PathItem" && it.clipping) return it;
            if (it.typename === "CompoundPathItem") {
                for (var j = 0; j < it.pathItems.length; j++) {
                    if (it.pathItems[j].clipping) return it;
                }
            }
        } catch (e) {}
    }
    return null;
}
```

### Cover-Fit and Contain-Fit Sizing

To make an image fully cover a rectangle (any overflow hidden by a mask), use
the larger of the width/height ratios. To make it fit entirely inside, use
the smaller. A bleed factor (e.g. `1.10`) lets a cover image extend slightly
past the clip edge.

```javascript
function fitItemToRect(item, rect, mode, bleed) {
    // rect = [L, T, R, B] (Illustrator: T > B)
    var rw = rect[2] - rect[0];
    var rh = rect[1] - rect[3];
    var ib = item.geometricBounds;
    var iw = ib[2] - ib[0];
    var ih = ib[1] - ib[3];
    if (iw <= 0 || ih <= 0) return;

    var sx = rw / iw;
    var sy = rh / ih;
    var s  = (mode === "cover" ? Math.max(sx, sy) : Math.min(sx, sy))
           * (bleed || 1);
    item.resize(s * 100, s * 100);

    var cx = (rect[0] + rect[2]) / 2;
    var cy = (rect[1] + rect[3]) / 2;
    var b  = item.geometricBounds;
    var w  = b[2] - b[0];
    var h  = b[1] - b[3];
    item.position = [cx - w / 2, cy + h / 2];
}
```



### Batch Process Files in a Folder

```javascript
var folder = Folder.selectDialog("Select folder of .ai files");
if (folder) {
    var files = folder.getFiles("*.ai");
    for (var i = 0; i < files.length; i++) {
        var doc = app.open(files[i]);
        // Process each document...
        doc.close(SaveOptions.DONOTSAVECHANGES);
    }
}
```

### Error Handling

```javascript
try {
    var doc = app.activeDocument;
    var layer = doc.layers.getByName("NonExistentLayer");
} catch (e) {
    alert("Error: " + e.message);
    // e.message, e.line, e.fileName available
}
```

## Troubleshooting

- **"undefined is not an object"**: Usually means the collection is empty or the index is out of bounds. Check `.length` before accessing items.
- **Script runs but nothing changes visually**: Call `app.redraw()` to force a screen refresh after modifications.
- **Color mode mismatch**: Document color space (RGB vs CMYK) must match color objects. Use `doc.documentColorSpace` to check.
- **Position seems wrong**: Remember scripted documents use bottom-left origin with Y increasing upward. The `position` property is the top-left of the bounding box.
- **Text not appearing**: Ensure the text frame has a non-zero size. For point text, set `position`; for area text, provide a valid path to `areaText()`.
- **File paths on Windows**: Use forward slashes (`/`) or double backslashes (`\\`) in path strings, or use the `File` object constructor.
- **Dialog boxes interrupting batch scripts**: Set `app.userInteractionLevel = UserInteractionLevel.DONTDISPLAYALERTS` before batch operations.
- **Collections use `getByName()`**: Many collection objects support `getByName("name")` which throws an error if not found; wrap in try/catch.
- **"Target layer cannot be modified"**: A locked layer, sublayer, or parent group (often a clip group like `Cover_Mask`) is blocking the edit. Recursively clear `locked` and `hidden` across the document before modifying. See [Recursively Unlock Layers and Groups](#recursively-unlock-layers-and-groups-before-editing).
- **"Unable to set placed item's file, is the file path provided valid?"**: The file exists and the path is correct, but `PlacedItem.file` does not accept the format. SVG is the most common cause — use the [open / copy / paste pattern](#placing-svg-content-copypaste-pattern) instead.
- **`RasterItem.file = newFile` does nothing or throws**: `RasterItem` does not expose a writable `file` property. Add a new `PlacedItem` to the same parent, restore the bounds and name, then `.remove()` the raster.
- **`arguments[0]` is `[object BridgeTalk]`** (or empty): The script was launched through ESTK `-run` or a `BridgeTalk` message; positional arguments are not forwarded. Use a sidecar file at a known absolute path. See [External Invocation & Argument Passing](#external-invocation--argument-passing).
- **`$.fileName` is empty**: Same external-launcher cause. Do not derive resource paths from `$.fileName` in scripts that may be invoked headlessly — use absolute paths or `Folder.temp`.
- **Script appears to do nothing**: Almost always either a locked ancestor, suppressed dialogs swallowing the error, or a missing explicit `saveAs` after edits. Add a top-level `try`/`catch` that logs to an absolute path to confirm execution and capture the error.
- **`item.resize(sx, sy)` recentered the artwork unexpectedly**: `resize` defaults to scaling around the item's center (`Transformation.CENTER`). Pass an explicit `scaleAbout` argument or follow with `translate(dx, dy)` to reposition.
