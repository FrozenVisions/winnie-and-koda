# How to add the next Winnie & Koda video to the website

The live site is a static GitHub Pages site from `FrozenVisions/winnie-and-koda`, branch `main`. The working copy of the site is `/workspace/winnie-koda/site/` (that folder is what gets committed to the repo root). Do not change DNS. Do not post the video anywhere else from this process. Only add a video Chris has already approved and posted (or has asked to put on the site).

## 1. Make a web-sized copy

Source videos live in `/workspace/winnie-koda/videos/`. Do not commit the full-size masters (they are 20–40 MB). Encode a 720×1280 H.264 + AAC copy with a faststart moov atom, aiming under 10 MB, plus a poster JPG.

Use kebab-case names that match the story, and keep the date prefix:

```bash
SRC=/workspace/winnie-koda/videos/YYYY-MM-DD_slug.mp4
STEM=YYYY-MM-DD_short-slug
OUT=/workspace/winnie-koda/site/videos

ffmpeg -y -i "$SRC" \
  -vf "scale=720:1280:force_original_aspect_ratio=decrease,pad=720:1280:(ow-iw)/2:(oh-ih)/2,setsar=1,format=yuv420p" \
  -c:v libx264 -preset slow -crf 27 -profile:v high -level 4.0 \
  -movflags +faststart \
  -c:a aac -b:a 96k -ac 2 \
  "$OUT/${STEM}.mp4"

ffmpeg -y -ss 0.5 -i "$OUT/${STEM}.mp4" -frames:v 1 -q:v 3 "$OUT/${STEM}.jpg"
```

If the MP4 is over ~10 MB, raise CRF (try 28 or 29) and re-encode. Check that `moov` comes before `mdat`, and that ffprobe shows `h264` 720×1280 plus `aac`. If Composio’s `GITHUB_COMMIT_MULTIPLE_FILES` returns HTTP 413, the base64 payload is too large — re-encode at CRF 30 so the MP4 is about 3 MB or under (base64 under ~3.8 MB) and commit that file on its own.

## 2. Add a card at the top of the Latest videos grid

In `/workspace/winnie-koda/site/index.html`, find `<div class="video-grid">` inside `<section id="videos">`. New cards go first (newest first). Copy an existing `<article class="video-card">` and fill in:

- `poster="./videos/${STEM}.jpg"`
- `<source src="./videos/${STEM}.mp4" type="video/mp4">`
- Keep `controls playsinline preload="none"` and `width="720" height="1280"`
- Date, a short title, a one-line factual summary from the approved caption, and a source link (use the real source URL; do not invent facts)

CSS for the grid already lives in `css/styles.css` (`.video-grid`, `.video-card`, `.video-frame`). Do not redesign the page.

## 3. Bump the homepage sitemap date

In `sitemap.xml`, set `<lastmod>` on `https://winnieandkoda.com/` to the video’s date.

## 4. Commit and push to `main`

Commit the new MP4, poster JPG, the `index.html` and `sitemap.xml` edits, and any other site files you actually changed. Push to `main` (GitHub Pages deploys from that branch). A reliable method is Composio `GITHUB_COMMIT_MULTIPLE_FILES` as the FrozenVisions GitHub account: text files as `utf-8`, binary files as `encoding: "base64"`. If the payload is large, commit the HTML first, then one or two videos at a time.

## 5. Verify after deploy

Wait for Pages (often 30–90 seconds, sometimes a couple of minutes). Then check:

- `https://winnieandkoda.com/#videos` shows the new card at the top
- `https://winnieandkoda.com/videos/${STEM}.mp4` returns HTTP 200 with `Content-Type: video/mp4`
- `https://winnieandkoda.com/videos/${STEM}.jpg` returns HTTP 200

If a CDN or browser cache still shows the old homepage, retry with a cache-buster query on the HTML request, or check the raw GitHub Pages host `https://frozenvisions.github.io/winnie-and-koda/` only if the custom domain lags.

## Puppy Koda photos (already on the site)

The Puppy days gallery is `<section id="puppy-koda">` on the homepage, with images in `images/puppy-koda/`. It has 10 photos from album1: `055.jpg`, `058.jpg`, `059.jpg`, `061.jpg`, `062.jpg`, `064.jpg`–`067.jpg` and `069.jpg` (Koda as a puppy, with Winnie). The last cafe-floor shot (`070.jpg`) was removed at Chris’s request. Near-duplicate frames were dropped, keeping the sharpest, best-framed shot of each moment. They were web-optimised to a 1200 px long side, JPEG quality 80, with EXIF/GPS stripped. To add more real photos later: resize the same way, drop them in that folder, add a `<figure>` in album order (use `class="landscape"` for wider shots), and write accurate alt text in the form “Puppy Koda with Winnie, …”. Skip photos that show a person’s face (a hand or arm at the edge is fine). Do not invent scenes, dates or markings, and do not retouch the dogs.
