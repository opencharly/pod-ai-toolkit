# AGENTS.md — pod-ai-toolkit

The org-wide rulebook is the umbrella's `AGENTS.md`; it applies here in full and is not repeated.
This file adds only what is specific to **this** repo.

## What this repo is

A **pod-class** repo (`pod-*`): it owns the `ai-toolkit` **candy**, the two skills that describe
it, AND the image that composes it — `box/ai-toolkit-cdi/charly.yml` (the `podcdi.pod-cdi` base
composing the LOCAL candy) plus its disposable R10 bed `check-cachyos-ai-toolkit-pod` in
`charly.yml`. The box and the bed live here with the service they test: a `distro-*` repo holds
the distro family, not one service's box (umbrella `AGENTS.md`, "Repo classes and naming").

The class rule cuts both ways: the box composes the LOCAL `ai-toolkit` candy, never a pinned
release of this repo, so a bed run proves the tree under change.

Class rule: a pod repo carries a pod's candy, its skills beside that candy, and nothing that
belongs to another class.

## Non-negotiables for this candy

- **The upstream pin is a COMMIT.** ai-toolkit publishes no tags. Do not "advance to latest":
  pick a commit, verify it, and let the build assert `git rev-parse HEAD` equals it.
- **The runtime venv is built from the pinned commit's own `requirements.txt`.** ai-toolkit pins a
  git revision of diffusers plus a solved set of torchao/transformers/peft versions. Do NOT
  re-express that set in `pixi.toml` — `pixi.toml` owns the Python MINOR VERSION only. Rationale
  and precedent: `.agents/skills/ai-toolkit-layer/SKILL.md`.
- **Never move the venv.** `ui/cron/pythonPath.ts` resolves `<root>/venv/bin/python`; the GUI's
  jobs and the CLI wrapper must both find the same interpreter.
- **The state tree stays spliced by symlink.** ai-toolkit has no `--base-directory`; the volume is
  linked in at runtime. Do not "fix" a missing state dir by building it into the image.
- **Two services.** The UI and its worker restart independently; do not collapse them into one
  `concurrently` wrapper.
- **`ZImagePipeline` is a contract, not a convenience.** If the `ai-toolkit-diffusers-zimage` check
  goes red, the pinned diffusers revision lost Z-Image support — that is an RCA, not a skip.

## Adding a new model family

Follow the upstream handler, not the model name: read
`extensions_built_in/diffusion_models/<family>/` at the pinned commit for that family's `arch`
value and for whether it needs `extras_name_or_path`. A family whose HF repo lacks
`text_encoder/`/`vae/` always needs an extras repo — the De-Turbo case is the worked example.

## Proving a change

There is no bed in this repo; the disposable bed lives with the image that composes the candy
(`distro-cachyos`: `check-cachyos-ai-toolkit-pod`). A candy change is only proved by that bed
building and running it.
