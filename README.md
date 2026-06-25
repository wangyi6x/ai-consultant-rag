# ai-consultant-rag

n8n workflow template: an AI sales-consultant bot for Telegram that answers from a **knowledge base (RAG)** — not from the prompt. It understands voice and text, holds a real conversation, qualifies the lead, and hands hot ones to a human manager.

One engine, any niche — only the knowledge base changes. Swap Telegram for WhatsApp or a website widget without touching the logic.

> 📖 **Full write-up with n8n node screenshots and a live chat demo (Russian):**
> [delai.agency/avtomatizacii/ai-konsultant-telegram-n8n](https://delai.agency/avtomatizacii/ai-konsultant-telegram-n8n/)
>
> Step by step: how both workflows were built, what broke and how it was fixed, the cost at scale.

## What's included

- `workflow.json` — the bot: Telegram trigger → voice/text split → AI agent with memory and knowledge-base search → reply parsing → manager handoff.
- `kb-ingestion.json` — a second workflow that loads your documents into the vector store. Run it once, and after every knowledge-base change.
- `.env.example` — the values you need (OpenAI key, Telegram bot token, manager chat id).

## Architecture

```
[Telegram Trigger] -> [Typing...] -> [Switch: voice / text]
     voice -> [Download file] -> [Whisper transcribe] -+
     text  -------------------------------------------+-> [Merge] -> [AI Agent]
                                                                       |  +- Chat model (gpt-4o-mini)
                                                                       |  +- Window memory (per chat id)
                                                                       |  +- Tool: RAG search over `delai_kb`
                                                                       v
                                            [Parse reply + lead tag] -> [Hot lead?]
                                                 true  -> [Notify manager] -> [Reply to client]
                                                 false -> [Reply to client]
```

The agent always calls the RAG tool before answering, so it speaks from your documents. When a lead is hot **and** left a contact, the agent appends a hidden `[[LEAD|hot|niche|task|contact]]` tag; a Code node parses it, the IF routes a summary to the manager, and the tag is stripped from the client's reply.

## How it answers from a knowledge base (RAG)

`kb-ingestion.json` turns your documents into vectors and stores them under the key `delai_kb`. The bot's "Knowledge base search" tool retrieves from the same key. Change a service, price or policy → edit the docs in the Code node, re-run ingestion, and the bot is up to date — no re-coding.

> ⚠️ The template uses an **In-Memory Vector Store** — simple for testing, but it resets when the n8n process restarts, and both workflows must run in the same n8n instance to share the `delai_kb` key. For production, swap it for a persistent store (Pinecone, Qdrant, Supabase) on both workflows.

## How to use

1. **Import both workflows** into n8n: Workflows → Import from File → `workflow.json`, then `kb-ingestion.json`.
2. **Set up credentials:**
   - OpenAI (`openAiApi`) — used by the chat model, Whisper and embeddings.
   - Telegram (`telegramApi`) — your bot token from [@BotFather](https://t.me/BotFather).
3. **Replace placeholders:**
   - `REPLACE_WITH_MANAGER_CHAT_ID` in the "Notify manager" node → your Telegram chat id (ask [@userinfobot](https://t.me/userinfobot)).
4. **Put your own knowledge base** into the "Knowledge base documents" Code node in `kb-ingestion.json` (replace the example DelAI docs with yours).
5. **Run `kb-ingestion.json` once** to fill the vector store, then **activate `workflow.json`**.
6. **Write `/start`** to your bot. The manager must message the bot at least once, or Telegram won't deliver the summary.

## Background

Full write-up (Russian — how it's built step by step, with screenshots and a live demo): https://delai.agency/avtomatizacii/ai-konsultant-telegram-n8n/

## License

MIT — see [LICENSE](LICENSE).

Built by [DelAI Agency](https://delai.agency).
