# Damper Markup

A small web tool for marking damper locations on floor plans. Open a PDF (or a picture of a plan), tap the spot, and it draws a label with the damper ID, a thin line, and a square on the spot. Then export a picture or a marked-up PDF.

**Live:** https://averypaiste.github.io/damper-markup/

It is plain HTML and JavaScript with no build step and no server. Everything runs in the browser. Files and marks stay on the device and are never uploaded.

## What it does

- **Open** a PDF of any length, or a PNG/JPG. PDF pages scroll up and down. The search bar at the top of the page list jumps to a page number, or finds pages whose text mentions a sheet number or room.
- **Mark**: type an ID (for example `FSD-1-12`), then tap the damper's spot. The ID counts up for the next one.
- **Two label styles**: "Square + line" (a small label, a thin line and a square on the spot) or "Circle + arrow" (a red oval, an arrow and a larger box).
- **Edit**: drag a label or its spot. Holding a label at the screen edge slides the plan, so labels can go out to the margin. Tap a label to change its ID, box color (pink, green for pass, yellow, blue, white), line color, note or size, or to delete it.
- **Tidy labels** makes every label on the page the same size and places each one close to its spot without overlapping.
- **Save page** confirms a page is done. Marks are also saved automatically on the device per file.
- **Export**: a picture of what's on screen or of the whole page (PNG), a PDF of only the marked pages, or the full PDF with the marks on it. The PDF keeps the original drawing sharp, because the marks are placed on top of the original pages.
- Works offline once it has been opened, and can be installed to a phone's home screen.

## Files

| File | What it is |
|---|---|
| `index.html` | The whole tool (HTML, CSS and JavaScript in one file) |
| `pdfjs/` | pdf.js 3.11.174 (Mozilla, Apache-2.0), used to show PDF pages |
| `pdf-lib.min.js` | pdf-lib 1.17.1 (MIT), used to write the marked-up PDF |
| `sw.js` | Service worker for offline use (optional) |
| `manifest.webmanifest` | Lets it be installed as an app (optional) |

There are no outside servers, CDNs, fonts or API keys. Both libraries are included in the repo, so it runs from any web server, including GitHub Pages.

## Adding it to another web app

These are the ways to use it, from easiest to most work.

### 1. Link to it

Link to the live page, or to your own copy. Add `?back=` with your app's address, and a "‹ Back" button appears in the top bar:

```
https://averypaiste.github.io/damper-markup/?back=https://your-app.example.com/
```

### 2. Embed it in an iframe (recommended)

Copy the files into a folder of your app (for example `public/markup/`) and show it in an `<iframe>`. Your app can hand it a file and get the finished markup back:

```html
<iframe id="markup" src="/markup/" style="width:100%;height:90vh;border:0"></iframe>
<script>
  const frame = document.getElementById('markup');

  // Open a PDF or picture in the tool (a File or Blob, for example from your own upload field)
  function openInMarkup(file) {
    frame.contentWindow.postMessage({ type: 'markup:open', file, name: file.name || 'plan.pdf' }, '*');
  }

  // Receive what the user exports (sent when they press Download)
  window.addEventListener('message', (e) => {
    if (e.source !== frame.contentWindow) return;
    const d = e.data;
    if (d && d.type === 'markup:export') {
      // d.blob is the PNG or PDF, d.name is a suggested file name, d.kind is 'png' or 'pdf'
      console.log('Markup exported', d.name, d.kind, d.blob.size);
      // e.g. upload d.blob to your server or attach it to a record
    }
  });
</script>
```

You can also open a file by address: `/markup/?src=/files/plan.pdf`. The address must be on the same site, or on a server that allows cross-site requests (CORS).

If you host it on the same site as your app, add `/markup/` to the service worker scope you already have, or delete the last line of `index.html` that registers `sw.js` if you don't want offline mode.

### 3. Build it into the app itself

`index.html` has no framework, so its parts can be moved into a React, Vue or other app. The drawing code is grouped under these comments, which you can search for:

- `geometry of one markup`: label size, line and square/oval (`layout`, `leader`, `drawMark`)
- `drawing the screen`: canvas zoom and pan
- `adding and editing`: tap-to-add, drag, edit panel
- `saving the picture`: PNG and PDF export (`buildOut`, `buildPdf`)

Each mark is a plain object, so marks are easy to store in a database:

```js
{ id: 'FSD-1-12', note: '', color: 'pink', style: 'line', x: 512, y: 340, bx: 600, by: 120, s: 1 }
// x, y   = the damper's spot, in PDF points (or image pixels)
// bx, by = top-left of the label
// s      = size; color = box color; style = 'line' or 'circle'
```

Marks are saved in the browser's `localStorage` under `markup:<file name>:<file size>`, with one list of marks per page number.

## Security notes

- The page has a Content-Security-Policy that only allows its own files. The one exception is that `?src=` may fetch a file from any `https:` address.
- PDFs are opened with pdf.js `isEvalSupported: false`, which closes the known pdf.js font bug (CVE-2024-4367).
- Nothing is sent anywhere. There are no accounts, tokens or tracking.

## Browsers

Current Chrome, Edge, Safari (iPhone and iPad included) and Firefox. Touch gestures: pinch to zoom, drag to pan, tap to add a mark.
