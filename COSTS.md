# Running Costs

What this bot costs to operate: fixed infrastructure plus pay-per-use API calls. All API prices below are the providers' published rates as of July 2026 — re-check them periodically ([fal.ai/pricing](https://fal.ai), [openrouter.ai/models](https://openrouter.ai/models), [Claude pricing](https://platform.claude.com/docs/en/pricing)).

## Fixed monthly costs

| Item | Cost | Notes |
|---|---|---|
| DigitalOcean droplet (1 vCPU / 2 GB) | ~$12/mo | Runs the bot, DB, media hosting. Check your plan for the exact figure. |
| Zernio subscription | per your plan | The publishing platform — see your Zernio dashboard. |
| Domain + HTTPS (optional) | ~$10–15/yr | Only needed for webhook mode; polling mode (current setup) needs none. |

## Per-use API prices

### Images (fal.ai)

| Model | Used for | Price |
|---|---|---|
| `fal-ai/flux-2/turbo` | Generated images with no text on them | $0.008/megapixel → **~$0.008** per 1:1 image |
| `fal-ai/nano-banana-2` | Generated images with text on them (accurate typography) | **$0.08** per image (1K resolution) |
| `fal-ai/nano-banana-2/edit` | Adding styled text to an uploaded photo | **$0.08** per edit |
| `fal-ai/nano-banana-pro` / `/edit` (optional upgrade) | Premium typography quality | $0.15 per image (1K), $0.30 (4K) |
| `fal-ai/flux-2/flex` (optional alternative) | Typography-strong FLUX variant | $0.06/megapixel |

The three models in use are runtime settings (admin console → Image engine), so switching tiers is a settings change, not a deploy.

### Text (LLM calls)

Every post makes 1–3 small LLM calls (intent detection, image-prompt enhancement, per-platform copy). Each call is roughly 1–2K tokens total.

| Provider | Price | Cost per call (approx.) |
|---|---|---|
| OpenRouter → `deepseek/deepseek-chat` (default) | ~$0.3/M input, ~$1/M output | **~$0.001** |
| Anthropic Claude Haiku 4.5 (fallback) | $1/M input, $5/M output | **~$0.005** |

Text is effectively free relative to images — under a cent per post even on the fallback.

## Cost per action

| Action | API calls | Approx. cost |
|---|---|---|
| `/dream` — no text on image | 3 LLM + flux-2/turbo | **~$0.01** |
| `/dream` — with promo text on image | 3 LLM + nano-banana-2 | **~$0.09** |
| Photo post — no text overlay | 2 LLM | **< $0.01** |
| Photo post — with text overlay | 2 LLM + nano-banana-2/edit | **~$0.09** |
| 🔄 Imagen (regenerate) | same as the matching /dream row | $0.01 / $0.09 |
| 🔄 Texto or ✏️ Ajustar (copy only) | 1 LLM | **< $0.01** |
| Publishing to Zernio | 0 (covered by subscription) | $0 |

**Rule of thumb: a post with AI text on the image costs about 9¢; anything else costs about a penny.**

## Monthly scenarios

Assuming the ~$12 droplet, excluding the Zernio subscription:

| Usage | API spend | Total |
|---|---|---|
| Light — 2 posts/day, ~half with on-image text | ~$3/mo | **~$15/mo** |
| Active — 5 posts/day, ~half with on-image text | ~$8/mo | **~$20/mo** |
| Heavy — 10 posts/day, most with on-image text + regenerations | ~$30/mo | **~$42/mo** |

## Cost controls already built in

- **Daily dream limit** (`DAILY_DREAM_LIMIT`, default 20/chat/day) caps the worst case at ~$1.80/day per chat even if every dream uses the expensive text model.
- **Model routing** — the cheap model ($0.008) is used automatically whenever no text is requested on the image; the $0.08 models only run when text is involved.
- **Every fal call is capped at 1K resolution** — the 2K/4K tiers cost 2–4× and aren't needed for social feeds.
- **Failed publishes cost nothing extra** — retry is a button press re-using the already-generated media; only regenerating the image spends money.

## If costs ever spike, check in this order

1. `journalctl -u social-bot | grep image_generated` — how many generations, and which model each used.
2. The fal.ai dashboard usage page — it itemizes per model.
3. Whether someone changed the models in the admin console to a premium tier (nano-banana-pro).
