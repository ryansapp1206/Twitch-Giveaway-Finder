# Twitch Giveaway Finder

Python tool that finds live Twitch channels giving away computer hardware, logs them to a CSV file (Google Sheets optional later), and opens the channel in a tab so I can enter by hand. It is also my first real Python project, so learning matters as much as the finished tool.

## About me
- College student, beginner in Python. Windows, VS Code, `venv`.
- Explain new concepts in plain language the first time they appear.

## How to help me
- Work in small steps, one new concept at a time. Don't write the whole project.
- Default: you explain the idea, I try it, you review and fix. Give full code only if I ask or I'm stuck.
- Explain why, not just what. When fixing my code, say what was wrong.
- If I paste an error, explain what it means before the fix.
- Keep code simple and readable. No extra classes, abstractions, or libraries.
- Tell me what to run and what output to expect.

## Hard rules
- **Never auto-enter giveaways.** The bot discovers and alerts only. No code that enters, claims, or chats for me.
- **No hardcoded secrets.** Twitch ID and secret live in `.env`, loaded with `python-dotenv`. Keep `.env`, `venv/`, `*.db`, and any Google service account JSON key in `.gitignore`. Never print the secret.
- **Read-only on Twitch.** Official Helix API (client-credentials token) and anonymous read-only chat (`justinfan` nick). No scraping, no logging in as me.
- Rate-limit tab opening. Never open a burst of tabs.

## Stack
Python 3, `requests`, `python-dotenv`, SQLite, `csv` for the log, `webbrowser` for tabs. Chat is a plain socket to `irc.chat.twitch.tv` (no `twitchio`, no async). `gspread` only as an optional last step.

## How it works
1. **Discover:** poll Helix "Get Streams" every few minutes. Filter by `game_id` (look up category IDs once with Search Categories) and `language`, and cap the number of pages (make the cap a setting; print pages used and the lowest viewer count reached). Pages start from top viewers, so the cap misses the smallest channels; narrow categories matter more than the cap. Match titles on hardware plus giveaway keywords. Match whole words ("win" must not match "window").
2. **Verify:** join candidates only, briefly, in read-only chat over a raw IRC socket (`NICK justinfan12345`, `JOIN #channel`, answer `PING` with `PONG`). One connection can join several channels; don't join many at once (Twitch rate-limits joins). Look for `!enter`, "giveaway", "winner", "drawing", and bot or mod announcements.
3. **Score** (a number, not yes/no), in two stages. Do the cheap check before the expensive one:
   - Stage 1 (free): title keywords plus fields already in Get Streams (viewer count, `started_at`).
   - Stage 2 (candidates only): one batched Get Users call for `broadcaster_type` and `created_at`, plus chat observation (activity, links, winner messages).
   - Up: title and chat agree, specific model named, bot or mod announcing, affiliate/partner, organic chat, past winners.
   - Down: links in chat, "send your login" messages, brand-new channel, no hardware named, high viewers with almost no chat.
   - Prefer low-competition giveaways (fewer viewers or entries).
4. **Remember:** store flagged channels in SQLite. Raise a channel's trust when winner messages appear later.
5. **Output:** dedupe alerts, append to a CSV (channel, title, hardware, score, time), open a tab only above a high score.

## Build order
1. Authenticate, fetch filtered streams (categories, language, page cap), print title matches
2. Scoring stage 1 plus SQLite dedupe
3. Raw-socket chat verification
4. Stage 2 scoring (Get Users), CSV log, and tab opening with cooldown
5. Winner tracking, then optionally Google Sheets

## Conventions
- Run `python main.py` from the project root with the venv active.
- Handle API failures (expired token, rate limit) instead of crashing silently.
- Split into `scoring.py`, `db.py`, `chat.py` only when `main.py` gets hard to read.

## Channel log
Notes on channels I've found are in `channel-log.md`. When I tell you about a channel, add an entry there in the format at the top of that file.
