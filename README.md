# api-sunset-rules

Machine-readable rules for API, SDK and AI-model deprecations that have a
published shutdown date. Each rule ties a pattern you can find in source code
to the vendor's own announcement of when that code stops working.

These are the rules behind [API Sunset Autopilot](https://otisz.hu/api-sunset).
If you received an issue from this account about a deprecated API in your
repository, the matching rule file here shows the official source, the date,
and exactly what was (and was not) checked.

## The idea: deadline-driven code repair

Most deprecations do not fail loudly in advance. The notice goes to an inbox,
the SDK keeps a default API version, a model ID sits in a config file as a
plain string, and the first visible symptom is an outage on the shutdown date.

Deadline-driven repair turns that around:

1. Start from a dated, official vendor announcement, not from a hunch.
2. Encode it as a rule: what to look for in code, what the deadline is, what
   to move to, and where the migration guide lives.
3. Find the exact files and lines that depend on the deprecated thing.
4. Report each finding with the deadline and source, and where the change is
   mechanical, propose a reviewable patch. Where it is not (for example a
   retired AI model, whose replacement behaves and costs differently), flag it
   and leave the choice to a human.

## Rules

| Rule | Provider | Deadline |
|---|---|---|
| [`google-ads-v22`](rules/google-ads-v22.yaml) | Google Ads API | 2026-10-07 |
| [`cloudflare-kv-legacy-routes`](rules/cloudflare-kv-legacy-routes.yaml) | Cloudflare Workers KV REST API | 2026-10-15 |
| [`shopify-admin-2025-10`](rules/shopify-2025-10.yaml) | Shopify Admin API | 2026-10-16 |
| [`openai-legacy-2026-10`](rules/openai-legacy-2026-10.yaml) | OpenAI API | 2026-10-23 (dated GPT-5 / o3 snapshots: 2026-12-11) |
| [`anthropic-sonnet-4-5`](rules/anthropic-sonnet-4-5.yaml) | Anthropic Claude API | 2026-11-30 |
| [`github-runner-min-version`](rules/github-runner-min-version.yaml) | GitHub Actions self-hosted runners | 2026-09-29 (enforcement already live) |
| [`apple-private-relay-domain`](rules/apple-private-relay-domain.yaml) | Sign in with Apple private relay | later in 2026, no exact date published |
| [`npm-trusted-publishing`](rules/npm-trusted-publishing.yaml) | npm publishing from GitHub Actions | around 2027-01, targeted, not firm |

Dates are copied from the rule files. A rule whose deadline has passed is kept,
because code that still matches it is now broken rather than at risk.

## Rule format

Every rule is one YAML file. The common fields:

| Field | Meaning |
|---|---|
| `id` | Stable rule identifier |
| `provider` | The vendor product the rule is about |
| `deprecated` | What is going away, in the vendor's terms |
| `deadline` | The date it stops working, as published. Free text when the vendor gives no exact date |
| `deadline_note` | Exact wording or caveats (tentative dates, moved dates, scope) |
| `deadline_source` | The official page the date was read from |
| `sources` | Further official pages, each with a short note of what it states |
| `verified_on` | When the source was last re-read (where recorded) |
| `replacement` / `target_version` | What to move to |
| `migration_guide` | The vendor's upgrade documentation |
| `patterns` | File globs and regexes that identify an affected line |
| `ignore_paths` | Paths never counted (vendored code, lockfiles, docs, fixtures) |
| `exclude_owners` | The vendor's own organisations, which are not reported |
| `tested_note` | What a finding has been checked for, and what it has not |

Some rules carry extra, rule-specific sections: the Google Ads rule lists
identifiers removed between v22 and v25 (`breaking_tokens`) and the mechanical
rewrites it allows; the OpenAI rule lists every retired model ID with its
official replacement. `engine` names a specialised matcher where plain regexes
are not enough (for example parsing workflow YAML for the npm rule).

## How rules are verified

* **Official sources only.** Every date comes from the vendor's own changelog,
  blog, deprecation page or documentation, linked in `deadline_source` and
  `sources`. Third-party summaries are not used as a source.
* **Read, not remembered.** The source page is re-read when a rule is written
  or changed, and the date is copied with its exact wording. Tentative or
  month-only dates stay tentative; nothing is rounded into a precise day.
* **Moved dates are tracked.** When a vendor moves or pauses a deadline (as
  GitHub did for runner enforcement), every announcement is listed in order.
* **Findings state their limits.** `tested_note` says precisely what was
  checked (for example: a complete string literal on a non-comment line of the
  default branch, patched files syntax-checked) and what was not (nothing was
  executed, runtime configuration may override a literal).
* **No guessed fixes.** Rules only generate patches for changes that are 1:1 by
  the vendor's own mapping. Model swaps and other behaviour-changing
  migrations are reported as advisory.

## Corrections

If a date, scope or pattern here is wrong, please open an issue with a link to
the official source. Wrong data is worse than no data, so corrections are
applied first.

## License

MIT, see [LICENSE](LICENSE).
