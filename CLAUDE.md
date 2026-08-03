# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo overview

This is a collection of independent, self-contained ML-security tools and research write-ups (the `13o-bbr-bbq/machine_learning_security` repo), not a single application. Each top-level directory has its own README/LICENSE/`requirements.txt` and should be treated as its own project — don't assume shared dependencies or cross-module imports:

- `DeepExploit/` — automatic penetration-testing tool combining Metasploit + deep reinforcement learning (A3C). The flagship, most actively maintained module.
- `Recommender/` (PyRecommender) — recommends injection code for detecting reflective XSS. Still on an old TF1/Keras 2.2.4 stack — has NOT been migrated to TF2.
- `Generator/` (DeepGenerator) — generates injection payloads via Genetic Algorithm + GAN.
- `Analytics/` — k-means clustering on packet-capture (KDD Cup) data.
- `CNN_test/` — adversarial example generation against a CNN.
- `Security_and_MachineLearning/` — course material (chapters + scripts).
- `Vulnerabilities_of_ML/` — research write-ups, Markdown/images only, no runnable code.
- `SAIVS/` — stub only ("Coming Soon"), no code.

There is no test suite, no CI, and no linter/formatter configured anywhere in the repo.

## DeepExploit gotchas

- Target runtime is **Python 3.7 + TensorFlow 2.10.1**. The A3C reinforcement-learning code (`ParameterServer`, `LocalBrain` in `DeepExploit.py`) runs natively on TF2 eager execution: `tf.keras.optimizers.RMSprop`, `tf.GradientTape` for the policy/value/entropy loss, and a per-`LocalBrain` `tf.function`-wrapped train step (built with an explicit `input_signature` in `__init__`, not as a bare decorator, since `NUM_ACTIONS` is only known once `main()` sets it at runtime). Weight sync between the global `ParameterServer` and each thread's `LocalBrain` uses `model.get_weights()`/`set_weights()`, not session-based assign ops. There is no `tf.Session`/`SESS` anywhere in the file anymore — don't reintroduce `tensorflow.compat.v1`.
- Model checkpointing uses `tf.train.Checkpoint`/`tf.train.latest_checkpoint(env.save_path)`, not `tf.train.Saver`.
- `Recommender/` is still on the old TF1/Keras 2.2.4 stack and was NOT touched by the TF2 migration above — treat it as a separate, still-legacy module.
- `DeepExploit/config.ini` contains Metasploit RPC connection settings (`server_host`, `server_port`, `msgrpc_user`, `msgrpc_pass`) with real-looking credentials committed in plaintext. Treat as local-dev/test-only, not a secret to preserve.
- DeepExploit requires a running Metasploit RPC daemon (`msgrpc`) reachable per `config.ini`, plus `nmap` (and optionally `proxychains`) on `PATH`.
- `DeepExploit/.gitignore` excludes `*.xml` (nmap output), `data/`, `trained_data/`, `deepexploit-env/` — but only within `DeepExploit/`. Always run DeepExploit from inside `DeepExploit/` so generated scan/report files land where they're ignored; there is no root `.gitignore` covering stray output at the repo root.

## Code style

- Existing code uses module-level `UPPER_SNAKE_CASE` constants as enums (e.g. `ST_OS_TYPE = 0`) rather than Python's `enum` module, with trailing comments column-aligned via multiple spaces. Match this local convention when editing existing files rather than reformatting to PEP8.
- No type hints, minimal docstrings, `docopt` (not `argparse`) for CLI parsing — consistent with the existing style.

## Linting

`ruff` is configured via `pyproject.toml` (line-length 120, target py37). Run `ruff check .` before considering a change done.
