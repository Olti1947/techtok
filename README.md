# TechTok (working title)

A mobile app for reading good writing one card at a time. Each card is a recent post from a curated list of about 200 tech and general-knowledge blogs, with its title, source, reading time and a short AI summary. Tap a card to read the article in a clean reader view.

It is built to be the thing you open instead of a social feed, and to let you go once you've read enough.

> **Status:** planning. No code has been written yet. The work is tracked in [issues #1 to #25](https://github.com/Olti1947/techtok/issues). The final app name hasn't been chosen; "TechTok" is a placeholder.

## Principles

The app must not become another feed people can't put down. These rules apply to every feature:

- **A daily limit.** You get 30 cards a day by default. When you reach the limit, the app shows a clear end screen ("That's 30 good reads for today") and stops. There is no "just 5 more".
- **The feed ends.** It is not infinite and never autoplays.
- **No engagement metrics.** No like counts, follower counts or streaks.
- **No engagement-bait notifications.** The app sends no notifications at all.
- **Saving beats scrolling.** Saving an article to read later is encouraged, and reading from Saved doesn't count toward the daily limit.

## Design

The visual direction is **Reading Room**: serif titles, a calm sage-and-ink palette, and the cover image framed inside the card rather than filling it. Light and dark themes are both designed.

- **[Figma file](https://www.figma.com/design/ocCGMOut27pLCwn5ONB43n):** components, color and spacing variables, text styles, and 11 screens in light and dark.
- **[Design spec](https://claude.ai/artifact/QhqporcCvQFzHRwQaxyd7L):** the token tables, a React Native theme object, and notes on each screen.

| Screen | Figma frame | Issue |
| --- | --- | --- |
| Onboarding (topics and daily limit) | 01 Onboarding | [#13](https://github.com/Olti1947/techtok/issues/13) |
| Swipe feed card, with and without a cover image | 02, 03 | [#14](https://github.com/Olti1947/techtok/issues/14) |
| Not interested, with undo | 04 | [#15](https://github.com/Olti1947/techtok/issues/15) |
| Reader and end of article | 05, 06 | [#16](https://github.com/Olti1947/techtok/issues/16) |
| Saved and its empty state | 07, 08 | [#17](https://github.com/Olti1947/techtok/issues/17) |
| Settings | 09 | [#20](https://github.com/Olti1947/techtok/issues/20) |
| Offline | 10 | [#17](https://github.com/Olti1947/techtok/issues/17) |
| Daily end screen | 11 | [#19](https://github.com/Olti1947/techtok/issues/19) |

Once the mobile app exists, its theme file is the source of truth for colors and type. Figma is the reference for layout.

## Stack

Everything is TypeScript, in a pnpm workspaces monorepo.

| Area | Choice |
| --- | --- |
| Mobile | React Native with Expo and Expo Router, FlashList for the paging feed, TanStack Query, react-native-mmkv, react-native-webview for the reader |
| API | Node.js 22 (strict TypeScript), Fastify with @fastify/jwt, pino for logs |
| Database | PostgreSQL 16 with pgvector, Drizzle ORM and drizzle-kit for migrations |
| Background jobs | pg-boss (cron and retries in Postgres, no Redis) |
| Feeds and extraction | rss-parser, @mozilla/readability, linkedom, sanitize-html |
| AI | Claude via @anthropic-ai/sdk for summaries, tags and category, with responses validated by Zod; an embeddings model for "more like this" |
| Shared types | Zod schemas in `packages/shared`, used by both the API and the app |
| Testing and errors | Vitest, Sentry |
| Hosting | Docker on a small VPS, or Railway or Fly.io |

## Planned layout

```text
apps/
  mobile/          Expo app (feed, reader, saved, settings)
  api/
    src/
      server.ts
      db/          Drizzle schema and migrations
      ingestion/   feed polling, parsing, URL canonicalization
      enrichment/  article extraction, summaries, embeddings
      feed/        ranking and feed routes
      users/       device auth, interactions, saved articles
      jobs/        pg-boss job definitions
packages/
  shared/          Zod schemas and types shared by api and mobile
```

## How it works

1. **Ingestion.** Every 30 minutes, each active feed is fetched with conditional requests (ETag and Last-Modified). New posts are canonicalized and deduplicated by URL.
2. **Enrichment.** Each new article's readable text is extracted and sanitized. Claude then writes a three-sentence summary and picks tags and a category. An embedding is computed from the title and summary. If extraction fails, the card falls back to the RSS excerpt and an "Open on site" link.
3. **Ranking.** Each card is scored on category interest (35%), freshness with a roughly 7-day half-life (25%), similarity to articles you recently opened or saved (25%), and source quality (15%). No source gets more than 2 cards in 10, and 2 of every 10 cards come from topics you haven't picked.
4. **Daily limit.** The server counts the cards you've seen today. Once the count reaches your limit, the feed returns nothing until tomorrow.

The API lives under `/api/v1`. Users start anonymous with a device token; Google and Apple sign-in come later.

## Roadmap

| Phase | Goal | Issues |
| --- | --- | --- |
| Weekend 1 | Backend core: monorepo, database, ingestion, enrichment, auth, basic feed | [#1 to #12](https://github.com/Olti1947/techtok/issues?q=is%3Aissue+%22W1%22) |
| Weekend 2 | The swipe app: onboarding, feed, interactions, reader, saved and offline | [#13 to #17](https://github.com/Olti1947/techtok/issues?q=is%3Aissue+%22W2%22) |
| Weekend 3 | Ranking, daily limit, interests and settings, "more like this" | [#18 to #21](https://github.com/Olti1947/techtok/issues?q=is%3Aissue+%22W3%22) |
| Weeks 4 to 6 | Private beta with friends: deployment, source health admin, EAS builds | [#22](https://github.com/Olti1947/techtok/issues/22), [#23](https://github.com/Olti1947/techtok/issues/23), [#25](https://github.com/Olti1947/techtok/issues/25) |
| Before public launch | Copyright, AI labelling, privacy and account deletion | [#24](https://github.com/Olti1947/techtok/issues/24) |

The beta succeeds if testers open this app instead of a social feed.

## Content and copyright

The app shows other people's writing, so these rules hold before any public release:

- Cards show the summary and an excerpt by default, and every article links to the original site. Full text is shown only for blogs with an open license or that have opted in.
- Every AI-written summary is labelled "AI summary".
- The crawler respects robots.txt, sends an honest User-Agent, and fetches each site at most once per poll.
- Anonymous accounts can delete their data, and the app ships with a privacy policy.

## Not planned for v1

User-generated content, comments, social following, a web version and monetization.

## Getting started

There is nothing to run yet. Setup instructions arrive with [#1](https://github.com/Olti1947/techtok/issues/1), which scaffolds the monorepo and a local Postgres.
