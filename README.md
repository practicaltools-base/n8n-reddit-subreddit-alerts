# Subreddit Alerts — n8n Workflow

Sends a Telegram, Discord or Slack message for every new post in the subreddits you follow, within the hour. No Reddit API app, no OAuth, and you pay only per post alerted.

Submitted to the n8n template library as *Send subreddit alerts to Telegram, Discord and Slack with Apify* (link added once approved).

---

## What it does

Runs every hour, fetches the newest posts from each subreddit you list, drops posts it has already alerted on, applies your optional filters (keyword, ignored authors, NSFW, ads), and sends one message per new post with the subreddit, title, excerpt, upvotes, comment count and link.

## The pattern worth stealing

**Pay per alert, not per poll.** Polling a feed on a schedule normally re-fetches the same posts every cycle and bills you for each of them. This workflow asks the scraper for only the last hour (`time: hour`) and caps items per run, so a quiet subreddit costs cents a month and a busy one costs exactly what it alerts. If you change the trigger interval, change the lookback with it — every 30 minutes can keep `hour`; every 6 hours needs `day`.

**Dedupe in the database, not in static data.** `Skip Already-Seen Posts` is n8n's Remove Duplicates node in *previous executions* mode. It stores seen post IDs in n8n's own `processed_data` table, so it persists on manual test runs, is shared across workers in queue mode, and is capped at 10,000 entries. A second test run returning 0 items is correct, not broken; the node has a *Clear deduplication history* operation for resets.

Both are source-agnostic.

## How it works

| Node | Role |
|---|---|
| `Every Hour` / `Run Once to Test` | Schedule trigger plus a manual trigger so you can see real data before wiring credentials |
| `Configure Subreddits & Channels` | All config lives here: subreddits, lookback, per-run cap, filters, chat targets. The only node you edit. |
| `Fetch New Subreddit Posts` | Runs the Apify Fast Reddit Scraper on each subreddit's *new* feed. Capped at 25 posts and $0.50 per run. |
| `Skip Already-Seen Posts` | Cross-execution dedupe by post ID |
| `Keep Matching Posts` | Drops ads, ignored authors (AutoModerator by default), NSFW, and — if set — posts without your keyword |
| `Send Alert to Telegram` | Enabled. Discord and Slack branches included and disabled. |

## Setup

1. **Install the Apify community node.** On n8n Cloud an Install button appears inside the node — workspace owner only. Do this first.
2. **Add your Apify API key.** Free at [console.apify.com](https://console.apify.com) → Settings → Integrations.
3. **Open `Configure Subreddits & Channels`.** Set `subreddits` (comma-separated, without r/) and `telegram_chat_id`. Filters are optional.
4. **Click Execute workflow.** The newest posts appear in the Apify node's output before you connect anything else.
5. **Add a Telegram bot credential** to `Send Alert to Telegram` (create the bot with @BotFather and add it to your chat or channel).
6. **Activate.**

To use Discord or Slack instead, enable that branch, set its target in the settings node, and delete the others.

## Cost

The actor charges per post returned (about $0.004 on Apify's free plan, $0.003 on paid) and nothing per run. A subreddit with ~20 new posts a day is about **$2.40/month**, inside Apify's free credit; ~100 posts a day is about **$12/month**. `max_posts_per_run` caps the worst case at $0.10 per run.

## Requirements

- n8n (Cloud or self-hosted)
- Apify account and API key — free plan works
- Telegram bot token, or a Discord webhook / Slack credential if you switch branches

## Customization

- Faster alerts: trigger every 30 minutes; `lookback` can stay `hour`.
- Cheaper: trigger every 6 hours and set `lookback` to `day`.
- Focused alerts: put a keyword in `only_if_title_contains` to turn a busy subreddit into a topic alert.
- Several audiences: duplicate the workflow per subreddit group, each with its own chat.
- Log everything: add a Google Sheets append node next to the delivery branches.

## Install

Import `subreddit-alerts-telegram-discord-slack-apify.json` into n8n: **Workflows → Import from File**.

## Disclosure

This workflow calls the [Fast Reddit Scraper](https://apify.com/practicaltools/apify-reddit-api) actor on Apify, which I maintain and which is metered per post. The workflow itself is free and the patterns above work with any polled data source.

## License

MIT
