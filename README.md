# Threads Scraper (Python)

[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue.svg)](https://www.python.org/)
[![Last verified](https://img.shields.io/badge/last%20verified-2026--10--13-brightgreen.svg)](#how-we-keep-it-working)

Scrape public Threads profiles, posts, replies and keyword search with Python. No login, no API key, one file.

> **Need this at scale?** The script is for learning and small jobs. The [ToolzerHub Threads API](https://toolzerhub.com/apis/threads?utm_source=github&utm_medium=readme&utm_campaign=threads-scraper) returns the same data as JSON from any server, plus tag search results, with no blocks to handle. 100 free credits a month, no card needed.

## What it scrapes

| Command | What you get |
|---|---|
| `profile <username>...` | Bio, bio links, follower count, verified and private flags, profile picture |
| `posts <username>` | A profile's latest posts, paged with `--limit`: text, date, likes, replies, reposts, media and links |
| `post <url>...` | One post with its replies: text, date, counts, author, media and links |
| `search <keyword>` | Public posts for a keyword (about 20), same fields as `posts` |

## Quick start

```bash
git clone https://github.com/toolzerhub/threads-scraper.git
cd threads-scraper
pip install -r requirements.txt
python threads_scraper.py profile nasa
```

More examples:

```bash
python threads_scraper.py profile nasa natgeo -o profiles.json
python threads_scraper.py posts nasa --limit 40 -o posts.json
python threads_scraper.py post https://www.threads.com/@nasa/post/DeaX_rOGaz- -o post.json
python threads_scraper.py search astronomy --limit 20 -o search.json
```

Every command takes `-o file.json` to save the result and `--proxy URL` to route through a proxy. The script also reads `HTTPS_PROXY`.

## Example output

`python threads_scraper.py profile nasa`

```json
{
  "id": "28829152388",
  "username": "nasa",
  "url": "https://www.threads.com/@nasa",
  "full_name": "NASA",
  "biography": "Making the seemingly impossible, possible. ✨",
  "bio_links": [],
  "followers_count": 12772513,
  "is_verified": true,
  "is_private": false,
  "profile_pic_url": "https://instagram.fcgr10-1.fna.fbcdn.net/v/t51.82787-19/6697..."
}
```

`python threads_scraper.py post https://www.threads.com/@nasa/post/DeaX_rOGaz-` (replies left out here; this post had 9)

```json
{
  "id": "4006620349303729406",
  "code": "DeaX_rOGaz-",
  "url": "https://www.threads.com/@nasa/post/DeaX_rOGaz-",
  "text": "Space news, hot off the trail: NASA’s Perseverance Mars rover has driven farther than any vehicle ever has on the surface of another world! \n\nOn Oct. 11, the...",
  "created_at": "2026-10-12T23:06:27+00:00",
  "username": "nasa",
  "full_name": "NASA",
  "is_verified": true,
  "like_count": 1771,
  "reply_count": 42,
  "repost_count": 94,
  "quote_count": 10,
  "is_reply": false,
  "reply_to_username": null,
  "type": "image",
  "media": [
    {
      "type": "image",
      "url": "https://instagram.fepa1-1.fna.fbcdn.net/v/t51.82787-15/84830..."
    }
  ],
  "links": [],
  "replies": [ ... ]
}
```

`python threads_scraper.py posts nasa --limit 5` (one post shown, first media item only)

```json
{
  "id": "4004368993196788713",
  "code": "DeSYGIdFUvp",
  "url": "https://www.threads.com/@nasa/post/DeSYGIdFUvp",
  "text": "POV: returning to Earth after spending 237 days living and working in orbit!\n\nNASA’s SpaceX Crew-12 mission safely splashed down Thursday, Oct. 8, in the Pac...",
  "created_at": "2026-10-09T20:33:24+00:00",
  "username": "nasa",
  "full_name": "NASA",
  "is_verified": true,
  "like_count": 2405,
  "reply_count": 26,
  "repost_count": 71,
  "quote_count": 5,
  "is_reply": false,
  "reply_to_username": null,
  "type": "carousel",
  "media": [
    {
      "type": "image",
      "url": "https://instagram.ftpe7-2.fna.fbcdn.net/v/t51.82787-15/84109..."
    }
  ],
  "links": []
}
```

Full outputs for every command are in [examples/](examples).

## Fields

### profile

| Field | Description |
|---|---|
| id | Threads user id |
| username | Handle |
| url | Link to the profile |
| full_name | Display name |
| biography | Bio text |
| bio_links | Links in the bio |
| followers_count | Number of followers |
| is_verified | Verified badge flag |
| is_private | Private account flag |
| profile_pic_url | Profile picture URL (signed; it expires) |

### posts, post and search

`posts` and `search` return a list of these. `post` returns one, with a `replies` list of the same shape.

| Field | Description |
|---|---|
| id | Post id |
| code | The code in the post URL |
| url | Link to the post |
| text | Post text |
| created_at | Post time, ISO 8601 UTC |
| username, full_name, is_verified | Author |
| like_count | Likes |
| reply_count | Replies |
| repost_count | Reposts |
| quote_count | Quotes |
| is_reply | `true` for a reply, including an author's follow-up posts in their own thread |
| reply_to_username | Who the reply answers |
| type | `text`, `image`, `video` or `carousel` |
| media | List of `{"type": "image" or "video", "url": ...}` (signed URLs; they expire) |
| links | Links in the text and the link preview, if any |

## How it works

1. It opens the Threads page as Chrome would. [curl_cffi](https://github.com/lexiforest/curl_cffi) matches the browser's TLS fingerprint as well as its headers, which Threads checks.
2. The page already carries its data as JSON in `<script type="application/json">` tags. `profile` reads the user record, `post` reads the post and its replies, and `search` reads the result list. The script finds each one by its shape instead of by a fixed path.
3. `posts` takes the first few posts from the profile page. For more, it sends the same query the Threads website sends for the next page, using the cursor from the page.
4. Each record is cleaned into the fields above.

A longer walkthrough with timed runs, a CSV example and datacenter vs residential results: [How to scrape Threads with Python](https://toolzerhub.com/blog/how-to-scrape-threads?utm_source=github&utm_medium=readme&utm_campaign=threads-scraper).

## Limits

- Public data only.
- Search returns about 20 posts and no second page. A post shows its first replies only (around 10 threads), not the full list.
- `posts` includes the account's replies, not only top-level posts. Check `is_reply`.
- Threads blocks or rate-limits some IPs, cloud and datacenter ones most. The error says so (HTTP 403 or 429, or a redirect). Run it from a home connection, or pass `--proxy` (a sticky residential proxy works best). A redirect also means a user or post that does not exist, so the "not found" error mentions both.
- No retries, no concurrency. That is on purpose: the script stays short enough to read in one sitting.
- It breaks when Threads changes its pages. See "How we keep it working".

## Scaling up

The script is for learning and small jobs. At scale you run into blocks, changing page formats and proxy costs. The [ToolzerHub Threads API](https://toolzerhub.com/apis/threads?utm_source=github&utm_medium=readme&utm_campaign=threads-scraper) handles all of that:

| | This script | ToolzerHub API |
|---|---|---|
| Data | Profiles, posts, a post's first replies, keyword search | 4 Threads endpoints: profile, profile posts (paged by cursor), post with replies, search (posts or tags) |
| Blocks and proxies | Your problem | Handled |
| When Threads changes | Wait for a fix here | Fixed on our side |
| Cost | Free, plus your proxy | 100 free credits a month, then from $0.99 per 1,000 calls |

## How we keep it working

Every command is run against live Threads pages before each update, and the "last verified" badge shows the last day they all passed. If something breaks, [open an issue](https://github.com/toolzerhub/threads-scraper/issues).

## FAQ

**Is scraping Threads legal?**
Scraping publicly available data is generally allowed in many places, but it depends on your country, what you collect and how you use it. Read the Legal section below.

**Why am I getting blocked?**
Threads blocks or rate-limits some IP addresses, cloud and datacenter ones most. Use a home connection or a residential proxy, and slow down.

**Can it search Threads tags?**
No. The [ToolzerHub API](https://toolzerhub.com/apis/threads?utm_source=github&utm_medium=readme&utm_campaign=threads-scraper) can.

## Related scrapers

- [instagram-scraper](https://github.com/toolzerhub/instagram-scraper): public Instagram profiles, posts, comments and search
- [tiktok-scraper](https://github.com/toolzerhub/tiktok-scraper): public TikTok profiles, videos, comments and hashtags
- [reddit-scraper](https://github.com/toolzerhub/reddit-scraper): public subreddits, posts, comments, search and user posts
- [linkedin-scraper](https://github.com/toolzerhub/linkedin-scraper): public LinkedIn jobs, companies, profiles and posts
- [google-maps-scraper](https://github.com/toolzerhub/google-maps-scraper): public Google Maps places, details and reviews
- [facebook-ad-library-scraper](https://github.com/toolzerhub/facebook-ad-library-scraper): public Meta Ad Library ads by keyword or advertiser, with their videos and images
- [zillow-scraper](https://github.com/toolzerhub/zillow-scraper): public Zillow listings, property details, Zestimates and price history

## Legal and responsible use

This project is for education and for collecting **publicly available** data. You are responsible for how you use it.

- **Public data only.** The script reads pages anyone can see without an account. Do not use it to get around logins, paywalls, CAPTCHAs or other access controls.
- **Terms and robots rules.** Threads' terms of service may restrict automated access. Read them and decide for yourself; following them is your responsibility.
- **Personal data.** Profiles and replies can contain personal data. Laws such as the GDPR (EU and UK) and the CCPA (California) may apply. Collect only what you need, have a lawful basis, keep it secure, delete it when you no longer need it, and respect people's requests to access or erase their data.
- **Copyright.** Posts, images and videos belong to their creators. Do not republish them without permission.
- **Be gentle.** Keep request rates low and do not disrupt the service.
- **No warranty.** The software is provided "as is" under the MIT License, without warranty of any kind. The authors are not liable for any claim or damage arising from its use.
- **Not affiliated.** This project is not affiliated with, endorsed by or sponsored by Meta Platforms, Inc. Threads is a trademark of Meta Platforms, Inc.
- **Takedown requests.** If you own rights in something here and want it changed or removed, email contact@toolzerhub.com.

This section is not legal advice. If you are unsure, talk to a lawyer.

## License

[MIT](LICENSE) © ToolzerHub
