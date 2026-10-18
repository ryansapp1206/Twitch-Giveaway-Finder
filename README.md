# Twitch Giveaway Finder

A Python tool that finds live Twitch channels giving away computer hardware, scores how legitimate each giveaway looks, and opens the best ones in a browser tab so I can enter by hand.

> **Status: in development.**

## Why I'm building it

Hardware giveaways on Twitch are easy to miss and full of scams. I wanted a tool that watches for them, filters out the junk, and tells me which ones are worth my time, while I stay in control of actually entering.

## What it does

1. **Discover:** polls the Twitch Helix API for live streams in chosen categories and languages, then matches titles against hardware and giveaway keywords (whole words only, so "win" doesn't match "window").
2. **Verify:** briefly joins candidate channels in read-only chat over a raw IRC socket and looks for signs of a real giveaway (`!enter`, "winner", "drawing", mod or bot announcements).
3. **Score:** rates each candidate with a number rather than yes/no. Signals that raise it: title and chat agree, a specific model is named, an affiliate or partner channel. Signals that lower it: links in chat, "send your login" messages, a brand-new channel, high viewers with almost no chat.
4. **Remember:** stores flagged channels in SQLite so alerts aren't repeated, and raises trust in channels that have paid out before.
5. **Output:** appends results to a CSV and opens a tab only for high scores, with a cooldown so it never opens a burst of tabs.

## Safety rules

- **It never enters giveaways.** It discovers and alerts only. There is no code that enters, claims, or chats on my behalf.
- **Read-only on Twitch.** It uses the official Helix API with an app token, plus anonymous read-only chat. No scraping and no logging in as me.
- **No secrets in the repo.** API credentials live in a local `.env` file that is git-ignored.

## Tech stack

Python 3, `requests`, `python-dotenv`, SQLite, `csv`, `webbrowser`, and a plain socket for chat. I deliberately avoided async code and chat libraries to keep it simple and to learn how the protocols work. Google Sheets export via `gspread` is an optional last step.

## Setup

Requires Python 3 and a free Twitch developer application ([dev.twitch.tv/console](https://dev.twitch.tv/console)) to get a client ID and secret.

```
git clone https://github.com/ryansapp1206/Twitch-Giveaway-Finder.git
cd Twitch-Giveaway-Finder
python -m venv venv
venv\Scripts\activate
pip install requests python-dotenv
```

Create a `.env` file in the project root:

```
TWITCH_CLIENT_ID=your_client_id
TWITCH_CLIENT_SECRET=your_client_secret
```

Then run it from the project root with the venv active:

```
python main.py
```

*(Setup steps will be updated as the code lands.)*

## Roadmap

- [ ] 1. Authenticate, fetch filtered streams (categories, language, page cap), print title matches
- [ ] 2. Stage 1 scoring plus SQLite dedupe
- [ ] 3. Chat verification over a raw socket
- [ ] 4. Stage 2 scoring (channel age and type), CSV log, tab opening with cooldown
- [ ] 5. Winner tracking, then optional Google Sheets

## Main focuses for personal improvement

HTTP APIs and OAuth client-credentials auth, raw sockets and the IRC protocol, SQLite, rate limiting, and keeping secrets out of version control.

## Notes

`channel-log.md` is my hand-kept list of channels I've come across while testing.
