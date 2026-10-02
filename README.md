# portal-affordable

a Hermes Agent profile for Nous Portal users who want to keep spend low.

v0.1.0. the numbers below were measured on 2026-10-02. Portal prices change, so check https://portal.nousresearch.com/models before relying on any of them.

## why this profile

Hermes runs side tasks on "auxiliary" models: describing images, compressing long sessions, naming sessions, and classifying risky shell commands. with the default `provider: auto`, each of these runs on your main model ([docs](https://hermes-agent.nousresearch.com/docs/user-guide/configuration#auxiliary-models)). on an expensive main model, every screenshot and every compression is billed at that model's rate.

this profile pins each side task to a low-cost Portal model that passed a test for that task. the main model is low-cost too. if you switch to a stronger model with `/model` for a hard turn, the side tasks stay on their own models and their cost does not change.

## install

```bash
hermes profile install github.com/notwitcheer/portal-affordable --alias
portal-affordable setup      # sign in to Nous Portal
portal-affordable chat
```

## what is set

- main model: `deepseek/deepseek-v4-flash-0731`. low cost, and its price is not a promotion that can end.
- vision: `inclusionai/ling-3.0-flash-vl`, falling back to `google/gemma-4-26b-a4b-it`, then `openai/gpt-4.1-nano`. an explicit vision model also sends images through this describer when the main model has native vision ([docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/vision)).
- compression: `deepseek/deepseek-v4-flash-0731` at low reasoning. the compression summary is what the session remembers of itself, so this slot uses the model that kept every fact in testing rather than the cheapest one.
- approval classifier: `openai/gpt-5-nano` at low reasoning. its one miss in testing marked a safe command as dangerous.
- titles, skills hub, MCP dispatch, profile descriptions, kanban triage, mail scoring, TTS audio tags and memory query rewrite: `inclusionai/ling-3.0-flash`.
- `display.show_cost: true`, so spend shows in the status bar.

these stay on your main model on purpose: `background_review` (it writes your memory and skills), `curator`, `kanban_decomposer` and `goal_judge`. they shape what the agent keeps and plans, so they get the stronger model.

## keep your own profile instead

installing a distribution replaces that profile's config. to keep your current setup and only move the side tasks, run these against your own profile:

```bash
hermes config set auxiliary.vision.provider nous
hermes config set auxiliary.vision.model inclusionai/ling-3.0-flash-vl
hermes config set auxiliary.compression.provider nous
hermes config set auxiliary.compression.model deepseek/deepseek-v4-flash-0731
hermes config set auxiliary.approval.provider nous
hermes config set auxiliary.approval.model openai/gpt-5-nano
hermes config set auxiliary.title_generation.provider nous
hermes config set auxiliary.title_generation.model inclusionai/ling-3.0-flash
```

or run `hermes model` and pick "Configure auxiliary models" ([docs](https://hermes-agent.nousresearch.com/docs/user-guide/configuration#configuring-auxiliary-models-interactively)).

## how the models were picked

every call went through the Nous Portal API on 2026-10-02. costs are the per-call `cost` that Portal returned.

- vision: two generated test images, an invoice with 12 facts to read and a browser form with 10, three runs each. ling-3.0-flash-vl read 12 of 12 on every invoice run and 9 or 10 of 10 on the form, at $0.00003 to $0.00007 a call. gemma-4-26b, mistral-small-3.2, gpt-4.1-nano and gpt-5-nano also read 12 of 12 on the invoice and 8 of 10 on the form, at up to ten times the cost.
- compression: a session of about 6,000 tokens with 10 facts planted in it (a path, a port, a branch, a decision, a bug number, a to-do, a client name, a version, a deadline and a failing test). deepseek-v4-flash-0731 kept all 10. ling-3.0-flash kept 9 or 10 across four runs at about a twentieth of the cost, so it handles the lower-stakes slots.
- approval: 20 shell commands, 12 of them dangerous. gpt-5-nano got 19 right; its one miss flagged `rm -rf ./build` as dangerous. ling-3.0-flash got 17 and returned empty answers on two dangerous commands, so it is not used for this slot.
- main model: a short multi-step tool task (find the live config file, read a value, write a file), three runs per model. all eight candidates passed, so this test did not separate them. the main model was chosen on price stability and how widely the community already runs it.

the profile was installed with `hermes profile install` into a clean Hermes home (v0.21.5), and each side task was checked to resolve to its intended model through Hermes's own config resolver.

## licence

MIT. see LICENSE.
