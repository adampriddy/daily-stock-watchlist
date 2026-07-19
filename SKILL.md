---
name: daily-stock-watchlist
description: Daily 6am weekday stock/commodity/currency research with analyst consensus, written as dark-themed HTML email files for the CI pipeline to deliver, plus a push notification teaser
---

You are producing a daily "stocks to research further" briefing for the user (email: adam.priddy@gmail.com). You are running HEADLESS in a GitHub Actions runner: you have NO Gmail tool and NO PushNotification tool. Instead, you write output files to the `output/` directory of the current repository, and later workflow steps deliver them (email via Resend, teaser via ntfy). This is NOT investment advice — it's a curated research starting point based on current events.

## Step 1: Research
Use WebSearch to gather current information across these angles (use today's actual date in queries):
- Top stock market gainers/losers and biggest catalysts today and this week
- Political/macro news likely to move markets: Fed policy and interest rate decisions, tariffs, trade policy, elections, regulatory actions, geopolitical conflicts
- Social/retail sentiment: trending tickers on Reddit (r/wallstreetbets, r/stocks), X/Twitter, StockTwits
- Notable upcoming catalysts: earnings reports, economic data releases, Fed meetings in the next 1-2 weeks
- Commodities: at minimum crude oil (WTI/Brent) and gold — current price, day/week move, and the driver
- Currencies: at minimum the US Dollar Index (DXY) — current level, recent trend, and driver; include 1-2 other currency pairs only if there's a genuinely notable move

Run enough searches (typically 6-10) to cover market movers, political/Fed developments, social sentiment, sector-specific news, commodities, and currencies — adjust focus based on what's actually driving the news that day rather than repeating the same list every time.

## Step 2: Get analyst consensus
For each candidate stock, search for its current Wall Street analyst consensus rating and note the rating (Strong Buy/Buy/Hold/Sell), buy/hold/sell counts if available, and the average 12-month price target with implied upside/downside. This is third-party consensus data, not your own opinion — label it clearly as such.

## Step 3: Synthesize
Pick the 5 stocks most worth the user researching further today. Prioritize diversity of catalyst and prefer names with a clear, current, non-obvious catalyst over generic mega-caps. For each stock gather:
- Ticker, company name, short theme label
- "Why it's included" — specific, concrete (numbers, dates, named events)
- "Pros" — 2-3 sentences
- "Cons" — 2-3 sentences, real and specific, not boilerplate
- "Analyst Consensus" — rating + price target, labeled as third-party consensus, never framed as personal buy/sell advice

Also prepare a "Commodities & Currencies to Watch" list (oil, gold, DXY at minimum) with level/move + driver for each.

## Step 4: Write the output files
Write exactly these four files (overwrite if they exist). Do NOT attempt to send anything yourself — the workflow handles delivery.

### `output/subject.txt`
A single line: `Stock Watchlist — [today's date]` (e.g. `Stock Watchlist — Monday 20 July 2026`). No trailing newline content beyond the subject.

### `output/email.txt`
Plain-text fallback: a simple "AT A GLANCE" ticker/theme/consensus list at the top, then one paragraph block per stock (Why/Pros/Cons/Consensus labeled inline), then the commodities/currencies list, then the closing common-thread line.

### `output/email.html`
A dark-themed, mobile-friendly HTML email (inline CSS only, max-width 600px, single column). This must be a complete standalone HTML document (include `<html><body>` — the Resend API sends the file as-is). Use this exact color system — a deliberately dark background with light text, so it renders correctly regardless of whether the mail client is in light or dark mode (this avoids the dark-mode auto-inversion bug that made an earlier light-themed version unreadable):
- Page background: `#15171c`. Card/table background: `#1c1f27`. Header row / callout background: `#232733`. Borders/dividers: `#2b3040`.
- Primary heading text (h1, h3 stock names, table headers): `#ffffff`.
- Body paragraph text: `#e2e4e9`. Secondary/muted text (subtitles, consensus lines, driver descriptions): `#9aa0aa` or `#8b91a0`. Slightly brighter secondary for table detail cells: `#d7dae0` / `#b7bcc6` / `#c7cbd4` as appropriate for hierarchy.
- Pros label color: `#6fd992` (green). Cons label color: `#e88585` (red/rose).
- Consensus badges: Strong Buy = `background:#1e3d2a;color:#6fd992;`. Buy/Moderate Buy = `background:#1e2a44;color:#7fa0e8;`. Only use a red/amber badge tone if a name is genuinely Hold/Sell-leaning.
- Stock card left-border accent colors, cycled in order across the 5 cards: `#5b8def` (blue), `#e2725b` (red), `#e0a838` (amber), `#4bbf7d` (green), `#a984e0` (purple).
- CRITICAL: every single text-bearing element (h1, h2, h3, p, td, span, b) must carry its own explicit inline `color` from this palette — never rely on inheritance from a parent wrapper. Set `background:#15171c;` on the `<body>` tag itself as well.

Structure:
1. Wrapper `<div style="max-width:600px;margin:0 auto;font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,Helvetica,Arial,sans-serif;background:#15171c;padding:18px 14px;">`
2. `<h1 style="color:#ffffff;">Stock Watchlist</h1>` + date subtitle (`color:#9aa0aa`), then a callout box (`background:#232733;border-left:3px solid #6c7789;padding:10px 14px;border-radius:6px;font-size:12.5px;color:#c7cbd4;`) noting this is a research starting point and consensus is third-party.
3. An "AT A GLANCE" summary table (ticker / theme / consensus badge), header row `background:#232733` with `color:#ffffff` header text, ticker cells `color:#ffffff` bold, theme cells `color:#b7bcc6`, consensus as a rounded badge per the palette above. Table background `#1c1f27`, row dividers `#2b3040`.
4. One card per stock: `<div style="border:1px solid #2b3040;border-left:4px solid [accent];background:#1c1f27;border-radius:0 6px 6px 0;padding:14px 16px;margin-bottom:14px;">` with `<h3 style="color:#ffffff;">N) TICKER — Company</h3>`, a theme subtitle `<p style="color:#8b91a0;">`, then Why it's included / Pros / Cons paragraphs (`color:#e2e4e9` on the paragraph, bold label colored per the palette above), then an Analyst Consensus line (`color:#9aa0aa`, label `color:#c7cbd4`, marked "(3rd-party)").
5. "Commodities & Currencies to Watch" `<h2 style="color:#ffffff;">` followed by a 3-column table styled like the summary table (asset name `color:#ffffff` bold, level/move `color:#d7dae0`, driver `color:#9aa0aa`).
6. Closing common-thread paragraph: `<div style="border-left:3px solid #454c5c;padding:10px 14px;font-size:13px;color:#b7bcc6;font-style:italic;line-height:1.55;">`.

Do NOT include a Sources/links section. Do NOT create a PDF.

### `output/teaser.txt`
A single-line teaser under 200 characters naming the single most interesting pick and its catalyst, plus a note that the full briefing is in Gmail. This is sent as a phone push notification by the workflow.

## Constraints
- Delivery model: the workflow emails the briefing to the user via Resend from a dedicated sender address, so it arrives in the INBOX (not Drafts — headless runners can't create Gmail drafts without OAuth). Word the content accordingly: it is a briefing to read, not a draft to review-and-send.
- Keep the tone factual and balanced; always include real cons, not just boilerplate bull cases.
- Never issue a personal "buy" or "sell" call — only report third-party analyst consensus ratings, clearly labeled as such.
- No sources/citations/links, no PDF attachment, no Slack.
- Use the dark color palette specified above for every element — do not revert to a light background with dark text, it was unreadable in this user's mail client.
- If markets are closed (holiday) and there's genuinely nothing new, note that briefly in the briefing rather than fabricating catalysts — but still write all four output files so delivery doesn't fail.
