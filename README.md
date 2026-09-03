# Meme Bench

A single-file web app for making feed-sized caption images. It has two halves:

- **What is climbing** — pulls top posts from Reddit's public JSON feed for a set of
  subreddits and a time window, sorted by score. Any post title can be pushed
  straight into the caption field.
- **Caption studio** — writes captions with Claude, renders the one you pick in
  Archivo 800 over an uploaded or generated image on a 1080 × 1350 canvas, and
  exports it as a PNG.

Captions come from the Claude API (`claude-opus-5`): give it a brief, get six
captions back, each with the object to shoot underneath it. "6 more" asks again
and tells Claude what it already wrote, so the list keeps going without
repeating. Picking a caption also loads a matching image prompt.

Images can be uploaded from disk or generated with the Gemini API.

## Running it

There is no build step and no dependencies. Open `index.html` in a browser, or
serve the directory:

```bash
python3 -m http.server 8000
```

Then visit http://localhost:8000.

Serving over `http://` or `https://` rather than opening the file directly is
recommended — some browsers block `fetch` from `file://` origins, which breaks
the Reddit feed.

## API keys

Two keys, both optional, both entered under **Settings**:

- **Claude** — writes the captions. Get one at
  [console.anthropic.com](https://console.anthropic.com/settings/keys).
- **Gemini** — generates the images. Get one at
  [aistudio.google.com/apikey](https://aistudio.google.com/apikey). Pick a model:
  Gemini 2.5 Flash Image (default), Gemini 3 Pro Image (preview), or Imagen 4.

Each key goes from the browser straight to its provider and nowhere else, and
neither is committed to this repository. Ticking "Remember this key on this
device" stores both in `localStorage`; leaving it off keeps them in
`sessionStorage` for the current tab only.

**A key held in the browser is readable by anything running on the page.** That
is fine for a personal tool you run on your own machine. If you host this
somewhere other people load it from, move both calls behind a small server that
holds the keys and never ship them to the client.

## Known limitations

- Reddit rate-limits and sometimes blocks browser-origin requests. When that
  happens the feed shows an error and returns no posts; refreshing usually
  clears it.
- Requests go to the public `.json` endpoints with no authentication, so there
  is no way to raise the rate limit from the client.
- The canvas draws with `crossOrigin = 'anonymous'`, so a remote image served
  without CORS headers will fail to load rather than tainting the canvas.
- Claude is called from the browser with the
  `anthropic-dangerous-direct-browser-access` header. That header exists for
  exactly this case — a local tool using your own key — and is the wrong shape
  for anything multi-user.

## License

MIT
