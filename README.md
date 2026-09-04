# Meme Bench

A single-file web app for making feed-sized caption images. It has two halves:

- **What is climbing** — two sources. **Imgflip templates** (default) ranks the
  top 100 meme templates by how many captions people made this week, and any of
  them can be dropped straight onto the canvas as the image. **Reddit** pulls top
  posts from the subreddits you pick. Either way a title can be pushed into the
  caption field.
- **Caption studio** — writes captions with Claude, renders the one you pick in
  Archivo 800 over an uploaded or generated image on a 1080 × 1350 canvas, and
  exports it as a PNG.

Captions come from the Claude API (`claude-opus-5`) two ways:

- **From a brief** — describe the subject, get six captions back, each with the
  object to shoot underneath it. "6 more" asks again and tells Claude what it
  already wrote, so the list keeps going without repeating. Picking a caption
  loads a matching image prompt.
- **From the image** — with a picture on the canvas, **Read the image** sends it
  to Claude, which works out what is actually in the frame and writes six
  captions against that specific detail rather than a generic version of the
  object. Sharpest first. The image is downsized to 1024px and sent as JPEG.

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

## Keys and IDs

All optional, all entered under **Settings**. Imgflip needs nothing.

- **Claude** — writes the captions. Get one at
  [console.anthropic.com](https://console.anthropic.com/settings/keys).
- **Reddit client ID** — only for the Reddit source. Reddit stopped serving its
  public `.json` endpoints to browsers (403 with no CORS headers), so the feed
  now goes through Reddit's OAuth API. Register a free **installed app** at
  [reddit.com/prefs/apps](https://www.reddit.com/prefs/apps) and paste the ID
  shown under the app name. Installed apps have no secret, so nothing sensitive
  is stored — the app fetches a one-hour application-only token with the
  `installed_client` grant and a random device ID kept in `localStorage`.
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

- Reddit's anonymous `.json` feed is gone; without a client ID the Reddit source
  shows a setup message instead of posts. Imgflip works with no setup.
- Reddit application-only tokens last one hour and carry no refresh token, so the
  app requests a fresh one when the old one is close to expiring.
- The canvas draws with `crossOrigin = 'anonymous'`, so a remote image served
  without CORS headers will fail to load rather than tainting the canvas.
- Claude is called from the browser with the
  `anthropic-dangerous-direct-browser-access` header. That header exists for
  exactly this case — a local tool using your own key — and is the wrong shape
  for anything multi-user.

## License

MIT
