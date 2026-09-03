# Meme Bench

A single-file web app for making feed-sized caption images. It has two halves:

- **What is climbing** — pulls top posts from Reddit's public JSON feed for a set of
  subreddits and a time window, sorted by score. Any post title can be pushed
  straight into the caption field.
- **Caption studio** — renders a caption in Archivo 800 over an uploaded or
  generated image on a 1080 × 1350 canvas, and exports it as a PNG.

Images can be uploaded from disk or generated with the Gemini API using your own
API key.

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

## Gemini image generation

1. Get a key at [aistudio.google.com/apikey](https://aistudio.google.com/apikey).
2. Open **Settings** and paste it in.
3. Choose a model: Gemini 2.5 Flash Image (default), Gemini 3 Pro Image
   (preview), or Imagen 4.

The key is sent from the browser directly to Google and to nowhere else. It is
never committed to this repository. Ticking "Remember this key on this device"
stores it in `localStorage`; leaving it off keeps it in `sessionStorage` for the
current tab only. Do not tick it on a shared machine.

## Known limitations

- Reddit rate-limits and sometimes blocks browser-origin requests. When that
  happens the feed shows an error and returns no posts; refreshing usually
  clears it.
- Requests go to the public `.json` endpoints with no authentication, so there
  is no way to raise the rate limit from the client.
- The canvas draws with `crossOrigin = 'anonymous'`, so a remote image served
  without CORS headers will fail to load rather than tainting the canvas.

## License

MIT
