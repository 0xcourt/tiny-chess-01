# tiny-chess-01 — 4 KB Python chess engine

Original engine by @courtizen (0xcourt), MIT licensed.

## Method

- 0x88 board, packed-int moves (from | to<<7 | promo<<14 | flag<<17).
- Negamax alpha-beta, iterative deepening on a fixed 80,000-node
  budget per move (never a clock, so fully deterministic); root moves
  re-ordered by the previous iteration's scores.
- Transposition table keyed by board tuple, 2 killer moves per ply,
  MVV-LVA capture ordering, quiescence over captures/promotions, and
  full evasion search when in check (a quiescence that used stand-pat
  in check lost 0-8; fixing it was worth ~+1,000 Elo).
- Evaluation: material + formulaic piece-square tables (own values),
  tapered king midgame/endgame by non-pawn material phase, bishop-pair
  bonus, and king safety: friendly pawns shielding the king and
  flight squares next to it, phase-scaled so it fades in the endgame.
- A check extension was tried in v2 and REMOVED in v3: with the fixed
  80k-node budget it cost more breadth than the extra ply was worth.
  Ablation, trials 0/2: king-safety-only 1,530/1,464; safety+extension
  1,253/1,326; extension-only 1,326/1,173; neither 1,326/989.
- Move legality: pseudo-legal generation + own-king-attacked filter;
  repetition inside a search line scores 0. Root falls back to the
  first legal move if the env disagrees about legality.
- Source minified, lzma-compressed (FORMAT_ALONE, preset 9) and
  base85-packed into `main.py` (3,906 bytes) with a two-line unpacker
  (line 1 the unpacker prefix, line 2 the packed blob);
  the uncompressed source is kept in the author's dev directory. No
  network, no files, no randomness — stdlib only.

## Run

```
python3 main.py          # needs MAKEFUN_ENV_URL set by the harness
```

## Local results

- Perft exact: startpos 20/400/8902/197281; Kiwipete 48/2039/97862.
- Make/unmake consistency: board + incremental eval restore exactly
  over 300 random plies and for every special move (castling both
  sides, en passant, promotion to all 4 pieces, double push).
- Speed: ~35k nodes/s (single CPU); 80k-node root search ~2.1 s at
  startpos; trial walls 972–1,090 s vs the 1,800 s budget — well
  inside the 10 s/move limit.
- Practice trials (kit env server + sealed scorer, run manually):

  | Version | Trial 0 | Trial 1 | Trial 2 | Mean |
  |---------|---------|---------|---------|------|
  | v1 | 1,326 (3W/1D/4L) | 1,396 (3W/2D/3L) | 989 (0W/3D/5L) | 1,237 |
  | v2 | 1,253 (2W/2D/4L) | 1,396 (3W/2D/3L) | 1,326 (3W/1D/4L) | 1,325 |
  | **v3** | **1,530 (4W/2D/2L)** | **1,396 (3W/2D/3L)** | **1,464 (4W/1D/3L)** | **1,463** |
  | sunfish | 1,326 | 1,396 | 1,253 | 1,325 |

  - v3 mean 1,463 clears the 1,154 final floor by 309 and sits 138
    above the sunfish practice mean (1,325). Local scores are
    directional only, not a prediction of the live score.
- Determinism: two runs of trial 0 produce byte-identical games
  (verified: trial-0-A vs trial-0-A2, IDENTICAL moves, same Elo
  1,530, same spare plies 450).
