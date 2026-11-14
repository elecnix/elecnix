# Nicolas Marchildon

## Coding agents

### [your-attention](https://github.com/elecnix/your-attention)

An agent's question waits in whichever terminal asked it, so with ten sessions open it is found late or never. Your Attention collects every question into one queue of cards, one at a time, each showing whether its agent is still listening, busy, or offline. An answer routes back to that session within a second, and waits there if the session went offline rather than being lost. Every answer carries a delivery state — stored, delivered, read by the model — because "answered" and "seen" are different facts.

Cards can carry a Mermaid diagram, parsed before the card is shown, so a broken one returns to the agent with the parser's error instead of appearing as an error box. A card can also request a secret: the browser encrypts it for that session, and the model is given a file path rather than a value.

![One card at a time: a priority queue on the left, the open card on the right](https://raw.githubusercontent.com/elecnix/your-attention/main/docs/screenshot.png)

### [pi-prospector](https://github.com/elecnix/pi-prospector)

Reading your own agent transcripts to find the prompts and skills that waste turns is work nobody does. pi-prospector does it instead: it indexes sessions from five harnesses into a local database, measures every turn, and produces ranked proposals about what to change. Nothing it proposes is applied.

Indexing is incremental and deterministic — only new lines are parsed, forked sessions share their message tree — and a model is consulted only where the signal justifies the spend. The output is an append-only graph in which each verdict keeps its evidence and is recomputed only when something it depends on changes.

### [bellwether](https://github.com/elecnix/bellwether)

Starting a child agent takes one call. Knowing whether it is working, stalled, waiting for input, or already dead is the harder half, and most setups answer it by watching a terminal. Bellwether subscribes to [herdr](https://github.com/herdrdev/herdr) pane state and gives the parent a new turn whenever a child reports, goes idle, blocks, stalls, or exits — five signals with their own settle times, so three seconds of "blocked" is distinguished from ten minutes of silence.

Most of the design handles failure. A child's record reaches disk before herdr is asked to spawn anything, so a crash halfway through still says what was requested. The subscription opens before the snapshot is read, since herdr does not replay events, so a reconnect reconciles instead of assuming continuity. One outbox entry per child lets newer state replace older, marked read only once the parent's context contains it.

## Code review

### [cite](https://github.com/elecnix/cite)

AI reviewers get muted because they comment on style, on files nobody touched, and on lines they cannot demonstrate. Cite is built around one rule: a finding that cannot quote the line it is about is dropped, not reworded. It reviews with the model you choose, anchors comments to real lines, and publishes a check run a repository may require before merge.

Noise is treated as a contract rather than a hope: ten comments per review, two per file, three on a small pull request, no style comments, and a push re-reviews only the files whose content changed while earlier findings elsewhere stand untouched. Nothing to say means nothing posted. Cost is printed per run, and a run the provider does not price reads `unknown` rather than `$0.0000`. Adoption needs no new file, because it reads the instruction files a team already has, and its compatibility tiers are dated with a test fixture behind each guarantee.

### [dupdelta](https://github.com/elecnix/dupdelta)

A codebase that is twelve percent duplicated was twelve percent duplicated before the branch under review and will still be next week. A job reporting that constant says nothing about the change in front of it, which is why duplication reports get muted. dupdelta scans a branch and its merge-base and reports only what appeared between them: no baseline to regenerate, no allowlist to triage, no threshold to keep tuning.

Adding a language is a table and a registry entry rather than code, and a test asserts that every node kind a language declares exists in the grammar linked against it, so a typo cannot quietly turn a language into one that finds nothing.

## Watching pull requests

`gh` shows a pull request body and a check list. Inline review threads are invisible, and an agent working on the pull request has no idea that a reviewer replied, CI went red, or the branch now conflicts. [gh-monitor](https://github.com/elecnix/gh-monitor) turns a pull request into a stream of events, one line per genuinely new change and nothing when nothing changed, with adapters for the terminal, for Claude Code ([claude-gh-monitor](https://github.com/elecnix/claude-gh-monitor)) and for Pi ([pi-ghpr-monitor](https://github.com/elecnix/pi-ghpr-monitor)).

Most of the design shows under failure. A failing API surface is reported once per episode instead of once per poll, and the recovery notice states which events were missed and will not be replayed, because the cursor only advances on a successful fetch; a consumer that knows it has a hole can backfill. When GitHub rate-limits the expensive query, the watch continues over a cheaper surface with its own budget. It began as a fork of an extension whose useful pull requests had gone unmerged upstream, and the fork carries them.

## Infrastructure

### [terraform-permcheck](https://github.com/elecnix/terraform-permcheck)

An `aws_backup_vault` needs `kms:CreateGrant`, and no Terraform plan says so. The gap surfaces as a failed apply three deploys later, or not at all when the plan happens to be a no-op. PermCheck reads the plan, maps each resource type to its cloud schema, and diffs what those resources require against the policy of the role that will apply them — including what a plan cannot show, such as the secret an application reads at startup or the image a workflow verifies before deploying.

For AWS it does not trust CloudFormation. It parses the terraform-provider-aws source, follows each resource's create, read, update and delete into helpers, retry closures and paginators, and embeds the resulting table in the binary, so a run never clones the provider and CloudFormation covers only what the parse misses.

### [infra-cost-model](https://github.com/elecnix/infra-cost-model)

A frontend taking ten thousand requests a second implies a load on everything behind it, and that derived load is what a cost model needs. Tools that ask for a static figure per resource cannot produce it. Here demand propagates through the service dependency graph and the formulas stay symbolic, so a what-if sweep re-evaluates parameters instead of re-deriving the model, and comparing two architectures is a sweep rather than a rewrite. Each metric maps to the line of the cloud bill it lands on, and each concept cites the paper it came from.

## Financial planning

### [lifedraft](https://github.com/elecnix/lifedraft)

Planning tools hide their assumptions and return a confident number. lifedraft takes a household — people, accounts, debts, properties, beliefs about returns — plus the options being weighed, simulates each through decades of tax and account rules, and ranks them by what survives after tax. The ranking is the whole output; the choice stays with the reader.

Two questions do more work than the ranking. One puts a dollar value on resolving each unknown in the file and ranks them, which turns a first run from a two-hundred-field form into the short list of lookups that pay. Another labels every value measured, stated, derived, assumed, or unknown. Mortgages and retirement ship as data packages, so the engine stays generic while the rules stay reviewable.

## Smaller tools

- **[visual-reviewer](https://github.com/elecnix/visual-reviewer)** — a green Playwright test means its assertions passed. visual-reviewer asks separately whether the software did what the test was meant to verify, and shows the request and the screen text when it did not. Advisory only, so it never fails a build.
- **[gh-pr-banner](https://github.com/elecnix/gh-pr-banner)** — sets, updates and clears a banner in a pull request body without regenerating the body, so the text outside the markers is preserved byte for byte and a model is never asked to re-emit prose it might quietly change.
- **[usage-tracker](https://github.com/elecnix/usage-tracker)** — anonymous daily usage counts for an open-source tool, on your own Cloudflare free plan. Your config lists the dimensions and the Worker stores nothing else, so a report has nowhere to put a user's text or a file path.

---

Everything else public here is a fork, an experiment, or archived code.
