# Meme Bench

A single-file web app for making feed-sized caption images. It has two halves:

- **What is climbing** — two sources. **Imgflip templates** ranks the top 100
  meme templates by how many captions people made on each this week; any of them
  drops onto the canvas, carrying its name and text-box count with it.
  **Imgur** shows the most viral posts of the moment, for ideas.
- **Caption studio** — writes captions with Claude, renders the one you pick in
  Archivo 800 over an uploaded or generated image on a 1080 × 1350 canvas, and
  exports it as a PNG.

Captions come from the Claude API (`claude-opus-5`) two ways:

- **From a brief** — describe the subject, get six captions back, each with the
  object to shoot underneath it. "6 more" asks again and tells Claude what it
  already wrote, so the list keeps going without repeating. Picking a caption
  loads a matching image prompt.
- **From the image** — with a picture on the canvas, **Read the image** sends it
  to Claude along with the web search tool. Claude identifies the template,
  searches for how it is actually used and what the joke structure carries,
  works out the mechanism, then writes six new captions using it. Funniest
  first, one line per text box. The image is downsized to 1024px and sent as
  JPEG; the round trip takes around half a minute because of the searching.

The prompt is explicit about what kills a caption — proverb voice, explaining
the joke, being about people in general — because without that, models default
to writing fridge magnets.

Images can be uploaded from disk or generated with the Gemini API.

## Layout

The caption box takes one line per text box. **Layout** picks how they land:

- **Match the template** (default) — text over the image when there is more than
  one line, caption above the image when there is one.
- **Caption above the image** — the plain-object format: Archivo 800 on white,
  picture underneath.
- **Text over the image** — the classic format: white with a heavy black outline,
  first line at the top, last at the bottom, any others spread between.

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

Two keys, both optional, both entered under **Settings**. The trending feed needs
neither — Imgflip's endpoint is open.

- **Claude** — writes the captions. Get one at
  [console.anthropic.com](https://console.anthropic.com/settings/keys).
- **Imgur client ID** — only for the Imgur source. Register at
  [api.imgur.com](https://api.imgur.com/oauth2/addclient), choose *anonymous
  usage without user authorization*, copy the Client ID. Issued immediately, not
  a secret.
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

- The canvas draws with `crossOrigin = 'anonymous'`, so a remote image served
  without CORS headers will fail to load rather than tainting the canvas.
- Claude is called from the browser with the
  `anthropic-dangerous-direct-browser-access` header. That header exists for
  exactly this case — a local tool using your own key — and is the wrong shape
  for anything multi-user.
- A Reddit source existed briefly. Reddit now answers its public `.json`
  endpoints with 403 and no CORS headers, and its OAuth path requires an app
  registration that its own form would not reliably issue, so it was removed.

## License

MIT
