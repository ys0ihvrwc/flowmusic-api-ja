# Flow Music API (`flowmusic`)

<!-- conv-kit:v1 -->

<p align="center">
  <img src="assets/badges/price.svg" alt="observed unit price"> <img src="assets/badges/billing.svg" alt="billing model"> <img src="assets/badges/compat.svg" alt="OpenAI-compatible endpoint">
</p>

<p align="center">
  <img src="assets/audio-waveform.svg" width="820" alt="Music generation request path through the APIMart audio endpoint">
</p>

> **from $0.01 per generation** on the APIMart route — `https://api.apimart.ai/v1` の OpenAI 互換エンドポイント1つだけ、月額プランは不要です。*(2026-10-12 時点)*

**[APIキー](https://go.apimart.ai/k-0f4205)** · **[料金](https://go.apimart.ai/k-f5974d)** · **[モデルページ](https://go.apimart.ai/k-840eca)** · [⚡ 60秒クイックスタート](#quickstart)

**チームが APIMart 経由で Flow Music（`flowmusic`）を使う理由**

- **キー1つでカタログ全体。**同じ `https://api.apimart.ai/v1` の base URL と `Authorization` ヘッダーで Flow Music（`flowmusic`）と画像・動画・言語モデルなど 300+ のモデルにアクセスできます。切り替えるのは `model` フィールドだけで、クライアント側の変更は不要です。
- **最低 $1 から、従量課金。**サブスクリプションも、事前に規模を見積もるプリペイドプランもありません。$1 からチャージして、呼び出した分だけ使えます。
- **課金内容はレスポンスで返ります。**完了したタスクはすべて請求額（`cost` / `credits_cost`）を報告し、`failed` になったタスクは返金されます。
- **非同期前提の設計。**送信して `task_id` を受け取り、`GET /v1/tasks/{id}` をポーリングするだけ。バッチ処理もリトライも特別な統合ではなく、普通のキュー処理です。

<!-- /conv-kit:v1 -->



Flow Music API on APIMart provides a production-ready AI music generation workflow for creating full songs from lyrics and style prompts, with vocals or instrumental output and asynchronous task polling.

## Model id and routes

| Route | `model` value | Billing | Notes |
| --- | --- | --- | --- |
| Music generation (async task) | `flowmusic` | per generated track, by operation | see the [model page](https://go.apimart.ai/k-840eca) for the exact tiers |

Endpoint: `POST https://api.apimart.ai/v1/music/generations`, then poll `GET /v1/tasks/{task_id}`.

## Pricing

<!-- pricing:model:start -->
| Resolution | List price | Effective price |
| --- | --- | --- |
| default | 0.075 | 0.06 |


<!-- pricing:model:end -->

Prices are a snapshot; the [pricing page](https://go.apimart.ai/k-f5974d) and [`data/model.json`](data/model.json) are refreshed by CI, and a completed task reports the exact amount in its `cost` field.

## Request parameters

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| model | string | — | the music model id |
| title | string | — | track title |
| sound_prompt | string | — | style / instrumentation description (one of sound_prompt or lyrics is required) |
| lyrics | string | — | lyrics text with section tags; leave empty for instrumental |
| bpm | string | number | — | tempo of the generated track |
| length | integer | — | target duration in seconds |
| audio_url | string | — | source audio for extend / cover / stem operations |
| callback_url | string | — | webhook called when the task completes |

The table lists the fields accepted by the APIMart `music/generations` endpoint. Check the [model page](https://go.apimart.ai/k-840eca) for anything this model adds on top.

## Quickstart

```bash
curl --request POST --url https://api.apimart.ai/v1/music/generations \
  --header "Authorization: Bearer $APIMART_API_KEY" --header 'Content-Type: application/json' \
  -d '{"model":"flowmusic","title":"My Song","sound_prompt":"upbeat pop music with piano","bpm":"120","length":60}'
```

```python
import os, time, requests

BASE = "https://api.apimart.ai/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['APIMART_API_KEY']}", "Content-Type": "application/json"}

created = requests.post(f"{BASE}/music/generations", headers=HEADERS, timeout=60, json={
    "model": "flowmusic", "title": "My Song", "sound_prompt": "upbeat pop music with piano", "bpm": "120", "length": 60,
}).json()
task_id = created["data"]["id"]

while True:
    task = requests.get(f"{BASE}/tasks/{task_id}", headers=HEADERS, timeout=60).json()["data"]
    if task["status"] in ("completed", "failed"):
        break
    time.sleep(5)
print(task.get("cost"), task["result"]["audio"][0]["url"])
```

Runnable versions: [`examples/`](examples). The task lifecycle is `pending → processing → completed | failed`.

## FAQ

**Flow Music のモデル id は何ですか？**

`flowmusic` です。上のルート表を参照してください。

**課金方法は？**

**from $0.01 per generation** on the APIMart route。完了したタスクは `cost` フィールドに正確な金額を報告し、失敗したタスクは返金されます。

**How do I get a key?**

Use the [Get an API key](https://go.apimart.ai/k-0f4205) link — $1 minimum top-up, no plan required.

<!-- related-repos:auto -->
## Related repositories

- **Same model, this language** — the other half of this pair: [flowmusic-prompts-ja](https://github.com/ys0ihvrwc/flowmusic-prompts-ja)
- **Same model, other languages** — 10 more language builds: English `flowmusic-api` · 中文 [flowmusic-api-cn](https://github.com/jj4e6nt/flowmusic-api-cn) · Deutsch `flowmusic-api-de` · Español `flowmusic-api-es` · Français `flowmusic-api-fr` · Relay/gateway `flowmusic-api-gateway` · Bahasa `flowmusic-api-id` · 한국어 [flowmusic-api-ko](https://github.com/x4g008/flowmusic-api-ko) · Português `flowmusic-api-pt` · Русский `flowmusic-api-ru`
- **More models in this language** — [midjourney-api-ja](https://github.com/apimartt1q/midjourney-api-ja) · [gemini-2-5-flash-image-preview-api-ja](https://github.com/ic7skkh/gemini-2-5-flash-image-preview-api-ja) · [nano-banana-api-ja](https://github.com/e4ylndl3g-v/nano-banana-api-ja) · [gemini-3-1-flash-image-preview-api-ja](https://github.com/gv4ju/gemini-3-1-flash-image-preview-api-ja) · [nano-banana-2-api-ja](https://github.com/lpbxkrzbfel/nano-banana-2-api-ja) · [gemini-3-pro-image-preview-api-ja](https://github.com/zzsxs8r/gemini-3-pro-image-preview-api-ja)
- **Topic hubs** — every build of this model, all languages: [flowmusic](https://github.com/topics/flowmusic) · [flowmusic-api](https://github.com/topics/flowmusic-api)

This repository is one build in a cross-linked set of API guides for the same model family; each language build links back to the others.
<!-- /related-repos:auto -->

## Attributed links (how this repository is measured)

| Purpose | Attributed link | Target |
| --- | --- | --- |
| Model page | <https://go.apimart.ai/k-840eca> | `apimart.ai/ja/model/flow-music` |
| Live pricing | <https://go.apimart.ai/k-f5974d> | `apimart.ai/ja/pricing` |
| Get an API key | <https://go.apimart.ai/k-0f4205> | `apimart.ai/ja/keys` |

Outbound APIMart links are minted through the promo link API; hand-made tracking parameters are rejected by
`tools/check_links.py` in CI.

## Disclosure

Flow Music is a third-party model served through APIMart; this repository documents how to call it and publishes
model ids and prices, and does not claim official status. Model names, prices and documentation belong to their
respective owners.

## License

MIT — see [LICENSE](LICENSE).
