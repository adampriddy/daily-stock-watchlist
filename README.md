# Daily Stock Watchlist (headless)

Runs weekdays at ~5:30am UK time on GitHub Actions. Claude Code researches the market, writes a dark-themed HTML briefing, the workflow emails it to you via Resend and pings your phone via ntfy.

## One-time setup

### 1. Create the repo
Create a **private** GitHub repo and push these files:

```
daily-stock-watchlist/
├── SKILL.md
├── README.md
├── output/.gitkeep
└── .github/workflows/watchlist.yml
```

### 2. Resend (email delivery)
1. Sign up at resend.com **using adam.priddy@gmail.com** — on the free tier without a verified
   domain, Resend only delivers to your own account email, sent from `onboarding@resend.dev`.
2. Dashboard → API Keys → create one, copy it.
3. (Optional, nicer) If you own a domain: Domains → add + verify it, then use e.g.
   `watchlist@yourdomain.com` as the FROM_EMAIL secret and send to any address.

### 3. ntfy (free push notifications, no signup)
1. Install the ntfy app on your phone (iOS/Android).
2. Subscribe to a topic with an unguessable name, e.g. `adam-watchlist-x7k2p9`.
   (Topics are public by name — anyone who knows the name can read it, so make it random.)

### 4. Repo secrets
Repo → Settings → Secrets and variables → Actions → New repository secret:

| Secret | Value |
|---|---|
| `ANTHROPIC_API_KEY` | Your Anthropic API key (console.anthropic.com) |
| `RESEND_API_KEY` | From step 2 |
| `FROM_EMAIL` | `onboarding@resend.dev` (or your verified-domain address) |
| `TO_EMAIL` | adam.priddy@gmail.com |
| `NTFY_TOPIC` | Your ntfy topic name from step 3 |

### 5. Test it
Repo → Actions → "Daily Stock Watchlist" → **Run workflow**. Within a few minutes you should get a push notification and an email in your inbox. Check spam the first time — mark it "not spam" and future ones land in the inbox.

## Differences from the Mac version
- **Sent email, not a Gmail draft**: headless runners can't create Gmail drafts (needs OAuth). The briefing arrives in your inbox from Resend instead — it's just there when you wake up.
- **ntfy replaces PushNotification**: same teaser, delivered through the ntfy app.
- **Failure alerts**: if a run fails, you get a high-priority ntfy ping instead of silence.

## Clock changes
The cron is UTC. `30 4 * * 1-5` = 5:30am during British Summer Time. When the clocks go back in late October, either leave it (briefing arrives 4:30am — still before 6) or edit to `30 5 * * 1-5`.

## Costs
- GitHub Actions: free (private repo free tier is 2,000 min/month; this uses roughly 100–200).
- Resend: free (3,000 emails/month; you'll use ~22).
- Anthropic API: pay-per-use — a research-heavy run with 6–10 searches typically costs a few tens of pence per day. Watch usage at console.anthropic.com for the first week.
- ntfy: free.
