# WordPress Website AI Site Agent

An approval-first AI agent plugin for WordPress sites using the Block Editor. It turns a natural-language request into a detailed change plan, then applies that plan only when a WordPress administrator explicitly approves it.

Built as a portfolio-ready demo for [kannaujiBoli.in](https://kannaujiboli.in), a WordPress cultural and literary website.

## What it does

- Accepts plain-language requests in Hindi, English, or Kannauji.
- Reads relevant WordPress pages/posts and proposes block-editor changes.
- Plans page/post creation, updates, trashing, attachment changes, and HTTPS media sideloading.
- Shows every individual action, its reason, and deletion warnings before execution.
- Requires an administrator to approve or reject every plan.
- Records the prompt, generated plan, status, and execution log in a custom WordPress table.
- Never permanently deletes a post or media item; removal moves it to Trash.

## Installation

1. Download this repository or create `kannauji-ai-site-agent.zip` from the plugin folder.
2. In WordPress, go to **Plugins → Add New → Upload Plugin**, upload the zip, and activate it.
3. Go to **AI Site Agent → Settings**.
4. Enter an OpenAI API key and an available text model, then save. For production, prefer adding this line to `wp-config.php` instead of storing a key in the database:

```php
define('KBA_OPENAI_API_KEY', 'your-api-key');
```

5. Open **AI Site Agent**, enter a request, review the generated plan, and choose **Approve and apply** or **Reject**.

## Demo flow

Use this request during a live demo:

> Refresh the introduction on the home page in Hindi. Keep the Kannauji cultural tone, make the paragraphs shorter, and create a three-column block section for भाषा, साहित्य, and संस्कृति. Do not publish any new pages.

The plugin shows a plan first. Clicking **Reject** proves that no content is modified. Clicking **Approve and apply** executes the recorded plan and displays its log.

## Architecture

```text
WordPress administrator
        │ prompt
        ▼
AI Site Agent admin UI ──► WordPress REST API ──► OpenAI Responses API
        ▲                          │                       │
        │                          ▼                       ▼
        └── approval/rejection ─ pending job + JSON plan ◄─ structured plan
                                           │
                                           ▼ (only after approval)
                                WordPress pages/posts/media
```

The browser never receives the OpenAI API key. Calls to the plugin REST API require a valid WordPress nonce and the `manage_options` capability.

## Security choices

- The agent is **plan-only** until a human administrator approves the plan.
- API keys stay server-side; use `KBA_OPENAI_API_KEY` in `wp-config.php` for production.
- REST endpoints require WordPress administrator capability.
- Posts and attachments are moved to Trash, not permanently deleted.
- Remote media imports accept HTTPS URLs only.
- All model-supplied post content is passed through WordPress `wp_kses_post()`.
- The OpenAI request sets `store: false`.

## Limits and next improvements

This is a strong live-demo MVP, not a replacement for staging and backups. Before deploying to production, add backups, a staging environment, token/cost budgets, request throttling, and an image copyright review process. The model may need better site-specific instructions for a custom theme. For a richer visual diff, add a block-level comparison view before approval.

## Repository layout

```text
kannauji-ai-site-agent/
├── kannauji-ai-site-agent.php  # Plugin, REST API, planning and execution
├── uninstall.php
├── assets/
│   ├── agent.js                # Admin dashboard behavior
│   └── agent.css               # Admin dashboard styling
└── README.md
```

## License

GPL-2.0-or-later.
