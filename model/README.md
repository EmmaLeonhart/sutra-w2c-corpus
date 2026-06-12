# Baseline weight→code model (official, 2026-06-01 run)

The official baseline seq2seq decompiler trained on this corpus: a small
encoder–decoder Transformer that reads a program's weight matrices + substrate
input→output pairs and **generates its Sutra source**. Published from the
2026-06-01 training run (Emma greenlit the official push 2026-06-12).

## Files

- `model.pt` — the checkpoint, a dict `{model, args, vocab}`
  (`torch.load(..., weights_only=False)`); self-contained.
- `vocab.json` — the token vocabulary (also embedded in the checkpoint).
- `eval_result.json` — the full substrate evaluation this card's numbers come
  from, produced by `experiments/w2c_seq2seq/eval_substrate.py` in the Sutra
  repo on 2026-06-01: per-structure rates, the win/fail lists, coefficient
  substitution probes.

## Architecture / training

d_model 128, 3 layers, 1,483,064 parameters; 40 epochs, batch 32, lr 3e-4,
coefficient auxiliary loss weight 0.5 (detached). Trained on the 7200-program
corpus (90/10 split: 6480 train / 720 held-out val) by
`experiments/w2c_seq2seq/model.py`.

## Measured results (held-out 720 programs, substrate-verified)

Every generated program is recompiled and **run on the Sutra substrate**;
"IO reproduction" means the generated source reproduces the held-out
input→output behavior (tol 1e-3), not just string match.

| metric | value |
|---|---|
| exact source match | 584/720 = **0.811** |
| exact (canonical, `1.0 *` stripped) | 594/720 = **0.825** |
| substrate IO reproduction | 595/720 = **0.826** |
| compile failures | **0** |
| run failures | **0** |

Structure transfers; scalar coefficients are the wall: the plain families sit
at 0.94–1.0 IO reproduction (`chain4` 1.0, `affine` 1.0, `bundle3` 1.0), while
the coefficient families drag the mean (`gen_affine` 0.44, `scaled_diff` 0.44,
`two_mat_affine` 0.17 IO). The coefficient-substitution probe
(`eval_result.json` → `coeff_subst`) measures the same wall directly. Detail
and the negative architecture levers (aux loss, post-hoc substitution, input
features — all null/negative) are in the Sutra repo's
`planning/findings/2026-05-30-w2c-coeff-head-diagnostic.md`.

## Reproduction

From the Sutra repo (model + data are regenerated, not unpickled, end-to-end):

```bash
git submodule update --init corpus
py experiments/w2c_seq2seq/prepare.py
py experiments/w2c_seq2seq/model.py
py experiments/w2c_seq2seq/eval_substrate.py
```
