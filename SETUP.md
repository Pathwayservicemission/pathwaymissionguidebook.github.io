# Slideshow → PDF Viewer Setup

## Two ways to feed the viewer — pick one

### Option A (what you asked for): pull straight from a Google Slides link
Use `.github/workflows/fetch-slides.yml`. It downloads your Google Slides
deck as a PDF straight from Google's export link and commits it as `slides.pdf`
— no PowerPoint file needed at all. It's already wired to your link:
`https://docs.google.com/presentation/d/12bPmPI0msUm4tOTMLfZaZZflP1M4uYmMHkzZ5aBR8H8/edit`

**Required:** the Google Slides file must be shared as **"Anyone with the link
can view"** (Share button → General access → Anyone with the link). If it's
restricted, the download step will fail with a clear error in the Actions log.

It runs automatically once a day, and you can also trigger it manually anytime
from the **Actions** tab → "Fetch Slides PDF from Google Slides" → **Run workflow**
(useful right after you edit the slides).

### Option B: convert an uploaded PowerPoint file
Use `.github/workflows/convert-pptx.yml` instead if you'd rather push an actual
`.pptx` file to the repo. It converts it to PDF using LibreOffice on push.
You don't need both — delete whichever workflow you're not using.

## Files in this package
- `.github/workflows/fetch-slides.yml` — pulls from your Google Slides link (Option A)
- `.github/workflows/convert-pptx.yml` — converts an uploaded `.pptx` (Option B)
- `index.html` — your viewer page (same design as before), set to auto-load
  a PDF named `slides.pdf` unless you tell it otherwise.

## One-time setup (using Option A — Google Slides link)
1. Set your Google Slides file to "Anyone with the link can view" (see above).
2. Copy these files into your existing GitHub repo, preserving folder structure:
   - `.github/workflows/fetch-slides.yml`
   - `index.html`
3. Commit and push.
4. Go to the **Actions** tab → "Fetch Slides PDF from Google Slides" → **Run workflow**
   (don't wait for the daily schedule the first time).
5. Once it finishes (usually under a minute), `slides.pdf` will appear in your
   repo root, committed automatically by the workflow.
6. Open your GitHub Pages URL — the viewer loads `slides.pdf` by default, no
   extra config needed.

## Living-document behavior (auto-updating)
The viewer now tries two sources, in order, every time someone opens the page:

1. **Live**: fetches directly from your Google Slides export link, no delay.
   If you just edited the slides, this shows the change instantly on next page load.
   This is doing most of the work — for viewers where it succeeds, they always
   see the current version, regardless of how often the backup below runs.
2. **Cached fallback**: if the live fetch fails for any reason (some browsers
   or networks block cross-site requests like this), it falls back to
   `slides.pdf` — a copy GitHub Actions refreshes **once a month**.

So the worst case (live fetch blocked, and it's been weeks since the monthly
refresh) is someone seeing a stale cached copy. Every Actions run here is
short (well under a minute), so a monthly schedule uses a trivial amount of
your minutes budget either way.

**Force a refresh anytime for free:** Actions tab → "Fetch Slides PDF from
Google Slides" → **Run workflow**. Do this manually right after a big edit if
you want the fallback copy current immediately, instead of waiting for the
next month.

**Tuning the schedule:** edit the `cron` line in `fetch-slides.yml`.
`0 13 1 * *` = 1:00 PM UTC on the 1st of each month.

## Pointing the viewer at your file
The viewer looks for a PDF in this order:
1. A `?pdf=yourfile.pdf` query string in the URL, e.g.
   `https://yourname.github.io/yourrepo/?pdf=Missionary%20Guidebook.pdf`
2. The `DEFAULT_PDF` constant near the top of `index.html`'s `<script>` block —
   set it to your exact filename, e.g. `const DEFAULT_PDF = "Missionary Guidebook (1).pdf";`
3. Otherwise it defaults to a file literally named `slides.pdf`

Easiest long-term fix: rename your PowerPoint file to `slides.pptx` before
pushing — then the converted PDF will be `slides.pdf` and the viewer picks it
up with zero configuration.

## Notes
- The workflow only runs when a `.pptx` file changes — editing `index.html` or
  other files won't trigger a pointless conversion.
- If you ever rename the PowerPoint file, the old PDF isn't deleted automatically —
  delete it manually in the repo if you don't want stale files sitting around.
- LibreOffice's conversion is very faithful for standard slide layouts, but
  highly custom PowerPoint animations/transitions won't carry over — only the
  visual layout of each slide.
