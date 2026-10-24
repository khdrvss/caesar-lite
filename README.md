# Caesar Lite

Small standalone Caesar: a fixed personality, Telegram chat and one web-search
service. Optional incoming photos and voice notes. One Node.js file, zero package
dependencies, no conversation memory, learned personality or session-usage tracking.

## Run

1. Install Node.js 22 or newer. No `npm install` is needed.
2. Copy `.env.example` to `.env` and fill in the settings.
3. Start with `node bot.mjs` (or `npm start`). Stop with Ctrl+C.

Required settings: a **separate Telegram bot token**, numeric `ALLOWED_CHAT_IDS`,
your chosen `LLM_BASE_URL`, `LLM_API_KEY`, `LLM_MODEL`, and `TAVILY_API_KEY`.
The model provider must support the chat-completions HTTP interface with function
tools, including a forced `web_search` call, and the `max_tokens` output limit.
The base URL includes the provider's
API prefix, typically `/v1`. No provider/model is selected or key provisioned by
this project. There is no automatic fallback or connection to the original Caesar.

Get a separate bot from BotFather, add it to an allowed group and enable whatever
message access the group needs. If Telegram privacy mode is enabled, Telegram may
not deliver ordinary name mentions; replies to the bot and `/search@bot_username`
are the dependable ways to address it. Add a private chat's numeric ID to the
allowlist to enable private messages. A configured webhook must be removed before
polling; the application reports that condition rather than changing it for you.

## Use

- In a group, mention `@your_bot`, say `caesar ...`, or reply to the bot.
- `/search question` explicitly searches the web.
- Current-information wording also requests search; the model can choose search
  for other questions. This uses at most one search and one final generation.
- `/help` and `/start` describe enabled features.
- Every request is independent. Only the current message and its direct Telegram
  reply are included; nothing is remembered for the next request.
- Replies are plain text. Source links from the current search appear underneath.
  Model-authored web links absent from search evidence are removed.

Edit `persona.txt` to change Caesar's voice. Its default is Uzbek Latin with a
direct, dry, informal tone. An explicitly requested other language is allowed.

## Optional media

- Photos: set `ENABLE_PHOTOS=true` only when the configured model accepts image
  input. Ask in the photo caption or reply to a photo while addressing Caesar.
- Voice notes: set `TRANSCRIPTION_MODEL` only when the same provider exposes
  compatible `/audio/transcriptions` and supports OGG. In a group, a voice note
  must be a reply to Caesar, or referenced by an addressed text reply.
- Media stays in memory. One incoming photo or voice note per request; maximum
  8 MB; voice maximum 120 seconds. Replies remain text. These flags do not certify
  that a provider actually supports media; failures produce a short notice.

## Limits and operational state

The application runs one request at a time, caps work at 60 seconds and allows at
most 5 more seconds for sending. No unbounded retries or recursive research.
Search uses Tavily basic search with at most five excerpts. There are no browser,
vector, database or queue services. Model/service accounts can still charge for
requests; the bot does not collect, display or persist usage/cost totals.

The only persistent operational files are `.state.json` (Telegram update cursor)
and `.bot.lock` (process ID while running). Neither contains messages. The cursor
is saved **before** handling each update to avoid duplicate processing; a crash
can lose an in-flight request. Telegram sends are never blindly retried because
a timeout can occur after delivery. This is not an exactly-once delivery system.
Do not run multiple processes or share one token with the original Caesar.
If you change to another bot token, stop Lite and remove its old `.state.json`
before restarting; polling cursors belong to the previous bot.
If a lock is left behind by a crashed process, startup refuses to proceed.
Inspect the recorded PID and confirm the old bot is stopped before manually
removing `.bot.lock`. The bot never tries to steal an existing lock.

Logs contain fixed event names, not prompts, messages, API errors or credentials.
Configuration and state files are ignored by git. Nothing from `groupbot/.env`,
its databases or its services is copied or imported.

## Verification status

Built on 2026-10-24. JavaScript syntax checks and read-only source review were
completed. No behavioral tests or live API probes were run. Actual provider
compatibility, web-search results, media and Telegram delivery remain unverified.
Configure your own services before running; this repository is not a deployment.

`npm run check` checks JavaScript syntax only; it does not run the bot or tests.

## API references

- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Tavily search](https://docs.tavily.com/documentation/api-reference/endpoint/search)
- [Chat HTTP schema reference](https://developers.openai.com/api/reference/resources/chat/subresources/completions/methods/create)
- [Transcription HTTP schema reference](https://developers.openai.com/api/reference/resources/audio/subresources/transcriptions/methods/create)
