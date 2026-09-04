# Meme Bench

A single-file web app for making feed-sized caption images. It has two halves:

- **What is climbing** — the top 100 Imgflip templates, ranked by how many
  captions people made on each this week. Any of them drops onto the canvas,
  carrying its name and text-box count with it. **Ask Claude what is landing**
  answers the same question by searching the web instead: six formats in use
  right now, with the joke structure each carries. Formats the feed also stocks
  get a button to load straight onto the canvas.
- **Caption studio** — writes captions with Claude, renders the one you pick in
  Archivo 800 over an uploaded or generated image on a 1080 × 1350 canvas, and
  exports it as a PNG.

Captions come from the Claude API (`claude-opus-5`) two ways:

- **From a brief** — describe the subject, get six captions back, each with the
  object to shoot underneath it. "6 more" asks again and tells Claude what it
  already wrote, so the list keeps going without repeating. Picking a caption
  loads a matching image prompt.
- **From the image** — with a picture on the canvas, **Read the image** sends it
  to Claude with the web search and web fetch tools and a four-step instruction:
  identify the image, go and read the wording of at least eight real instances
  of it, analyse what those captions have in common, then write six new ones
  running the same mechanism on a fresh subject. When the picture came from the
  feed, the template's Imgflip page is handed over by URL — web fetch will only
  open a link already in the conversation — which also yields the other names
  people call that format by, for the next round of searching.

  It reports what it identified the image as and quotes three of the real
  captions it read, so you can see whether the research actually happened. If it
  found none, it says so rather than pretending. Funniest first, one line per
  text box. The image is downsized to 1024px and sent as JPEG.

  The request is streamed, so a long run cannot die on an HTTP timeout, and the
  status line says what is happening — searching, how many pages it is reading,
  then writing — with a running clock and a **Stop** button. Expect anywhere from
  half a minute to a few minutes depending on how much it reads. Search is capped
  at 4 uses and fetch at 3; each round trip is where the time goes, so raise
  those in `SEARCH_TOOL` and `FETCH_TOOL` only if you want deeper research and
  will wait for it.

  One limit worth knowing: meme captions live inside the pixels, and the fetch
  tool returns page text, so nothing here does OCR on found images. What it can
  read is wording written out as text — Know Your Meme quoting notable examples,
  round-ups quoting good ones, forum and social posts with the caption in the
  title. The prompt sends it to those sources deliberately.

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

One key, entered under **Settings**, plus an optional one. The Imgflip feed needs
neither — its endpoint is open.

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

- The canvas draws with `crossOrigin = 'anonymous'`, so a remote image served
  without CORS headers will fail to load rather than tainting the canvas.
- Claude is called from the browser with the
  `anthropic-dangerous-direct-browser-access` header. That header exists for
  exactly this case — a local tool using your own key — and is the wrong shape
  for anything multi-user.
- Two other feed sources were tried and removed. Reddit answers its public
  `.json` endpoints with 403 and no CORS headers, and its OAuth path needs an app
  registration its own form would not reliably issue. Imgur has taken down
  self-serve registration entirely — `api.imgur.com/oauth2/addclient`, still
  linked from Imgur's own docs, 301s to the homepage, and the account settings
  page lists only apps you have authorised, with no way to create one. Web search
  through Claude replaced both: no key of its own, no registration, no CORS.

## License

MIT
