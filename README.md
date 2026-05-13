# evennia-mob-spawner-test-yaml

Tiny synthetic rule-set repo used by [`evennia-mob-spawner`](https://github.com/FullCircleMUD/evennia-mob-spawner) for live integration smokes against `ms_load`, `ms-validate`, and the `GitHubReader` / `LocalReader` plumbing. Held outside the library's own test suite (which uses pure in-memory fixtures) so the discovery + loading + validation pipeline gets exercised end-to-end against a real backing store.

This repo is **fixture data**, not example usage. The shape and contents are deliberately minimal — just enough to walk the manifest tree and feed a few representative rules through the pipeline.

## Layout

```
definitions.yaml         # level vocabulary + gating assertion
index.yaml               # root-level entries
shard0/                  # shard (folder kind)
  index.yaml             # shard0's children
  millholm.yaml          # zone (file kind) — 4 rules covering the
                         # main schema patterns
  wilderness.yaml        # zone (file kind) — 1 rule, second-file
                         # coverage for shard0
```

The manifest declares `levels: [shard, zone]` — two hierarchical levels. One shard, two zones beneath it. Add more shards / zones as integration tests need them.

## Rule patterns demonstrated in millholm.yaml

- **Base + variant** — `Wolf` (`rule_id: 1`) and `WolfFat` (`rule_id: 2`) share `key` and `area_tag`; different typeclasses and (would be) different loot. Demonstrates the indistinguishable-variant pattern from the library's architecture doc.
- **Death-cooldown semantics** — `KoboldChieftain` (`rule_id: 3`) uses `death_cooldown_seconds` (cooldown from kill time) rather than `respawn_seconds` (from last-spawn-attempt time).
- **Den / lair targeting** — `KoboldChieftain` uses `den_room_tag` to pin spawn to one specific room rather than a random area pick.
- **`post_spawn_hook`** — `KoboldChieftain` invokes a hook after creation for per-instance state reset.
- **Pack spawning** — `Kobold` (`rule_id: 4`) uses `spawn_with_typeclass` so members spawn into rooms that already contain a living chieftain.

Typeclass paths in this repo (`typeclasses.test_mobs.*` and `typeclasses.test_hooks.*`) resolve against the demo gamedir bundled with the `evennia-mob-spawner` library at `examples/demo_game/`. Tier 3 validation (engine-resolvability) only runs at `ms_load` time inside a gamedir that has those typeclasses importable; the CLI runs only Tier 1+2 and treats these paths as opaque strings.

## Conventions exercised

- **Index-driven discovery.** `index.yaml` files are the source of truth for what's part of the tree.
- **Rule identity.** Every rule declares `rule_id` (a non-negative integer, unique within its file). Same pattern as world-builder's `deployment_id`.
- **Validation gating assertion.** `definitions.yaml` carries `repo-ci-pre-validation: false`. The library defaults this to `false` so consumers explicitly opt in to "trust the gate" only after standing up CI on their rule-set repo.

## Using it

Against a local clone (no auth):

```bash
git clone https://github.com/FullCircleMUD/evennia-mob-spawner-test-yaml.git
ms-validate --reader=local --root=./evennia-mob-spawner-test-yaml
```

Against the live repo via `GitHubReader` (when the CLI gets a `github` reader variant): TBD.

## Editing

Treat changes the same as any rule-set repo: open a PR, let CI's `ms-validate` job gate the merge, only land green commits on `main`. Branch protection on `main` is the reason `ms_load` consumers can later flip `repo-ci-pre-validation: true` and skip pre-validation at deploy time without losing safety.
