# Facebook Downloader × Rio

Flask app: paste a link to a **public** Facebook video or reel, then save the video (with or without
the Rio ghost watermark), an MP3, the original file, the cover image or a details TXT.

Built the same way as the TikTok Downloader × Rio project: same pages, same tabs, same Render setup.

## Folder layout (Flask needs exactly this)

```
app.py
requirements.txt
render.yaml
templates/index.html
static/app.js
static/style.css
static/watermark.png     <- ghost logo burned into videos
static/ghost.png         <- logo in the header / hero
static/favicon.png
```

## Run locally

```bash
pip install -r requirements.txt
python app.py            # http://localhost:5000
```

## Deploy on Render

Push this folder to GitHub, create a **Web Service** from the repo (render.yaml is picked up).

## Links that work

`facebook.com/watch/?v=...`, `facebook.com/reel/...`, `facebook.com/<page>/videos/...`,
`facebook.com/share/v/...`, `facebook.com/share/r/...`, `m.facebook.com`, `fb.watch/...`

Only **public** videos work. Private videos, friends-only posts, group videos and anything that needs a
login cannot be saved (the app never asks for a Facebook password or cookies).

## Downloads, in short

| Tab | What you get | How |
|---|---|---|
| Video | MP4 in the chosen quality. Watermark switch + corner picker. | Watermark on: ffmpeg job. Off: direct stream. |
| Audio | Real **MP3** with title/creator tags. | ffmpeg job, using the video's own sound. |
| Document | **Original video file** (no watermark, no re-encode, plain file), **cover image**, **details TXT**. | Direct stream (or a quick join job, see below). |

### Why some qualities say "takes longer"

Facebook serves its best qualities (often 1080p) as two separate streams: picture only and sound only.
For those options the server downloads both and joins them with ffmpeg **without re-encoding**
(`-c copy`), so quality is not lost and it takes only a few seconds. Qualities that already contain
sound are direct downloads. If no sound stream exists, picture-only options are hidden (they would be silent).

Every download starts with `POST /api/prepare`, so any failure shows as a message in the page.

## Environment variables (all optional)

| Name | Default | Meaning |
|---|---|---|
| `PROVIDERS` | `ytdlp,own` | Order of lookup providers. `ytdlp` is the main one; `own` reads the video links from the public page. |
| `WATERMARK` | `1` | `0` switches the watermark feature off. |
| `WATERMARK_POSITION` | `br` | Default corner: `br`, `bl`, `tr`, `tl`. |
| `WATERMARK_SIZE` | `0.30` | Logo width as a share of the video width. |
| `WATERMARK_OPACITY` | `0.92` | 0.1 to 1. |
| `WATERMARK_IMAGE` | `static/watermark.png` | Use another PNG (transparent background). |
| `RATE_LIMIT` / `RATE_WINDOW` | `12` / `60` | Requests per visitor per window (seconds). |
| `GLOBAL_RATE_LIMIT` | `240` | Requests per window for the whole site. |
| `PROXY_HOPS` | `2` | See below. |
| `MAX_SOURCE_MB` | `100` | Largest video that ffmpeg will process. |
| `MAX_ACTIVE_JOBS` | `5` | ffmpeg jobs allowed at once. |
| `FFMPEG_PRESET` | `ultrafast` | `veryfast` gives smaller files but uses more CPU. |
| `PROXY_URL` | - | Proxy for Facebook requests. |
| `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN` | - | Keeps the download counter across restarts. |
| `COUNTER_KEY` | `fb_downloads` | Counter name (separate from the TikTok site's counter). |
| `DEBUG_ERRORS` | - | `1` shows the real error in the UI and enables `/api/debug/ip`. |

### Rate limit and `PROXY_HOPS`

`X-Forwarded-For` can be forged by visitors, so the app trusts only the entry added by your own
proxies, counted from the right. On Render this is normally 2. To check: set `DEBUG_ERRORS=1`,
open `/api/debug/ip` and confirm `seen_as` is **your** IP, then remove `DEBUG_ERRORS`.

## Notes

* **Facebook changes often.** If lookups suddenly start failing, redeploy so `yt-dlp` updates
  (it is not version-pinned on purpose). Render free plan: Manual Deploy > "Clear build cache & deploy".
* Facebook may block some datacenter IPs. If that happens, set `PROXY_URL` to a residential proxy.
* Render's free plan sleeps after 15 min idle (first request is slow) and wipes temp files on restart.
* Replace `static/watermark.png` with any transparent PNG to rebrand.
* Only save videos you made or have permission to use.
