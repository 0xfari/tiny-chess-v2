# Tiny Chess v2 — 4 KB Python chess engine

A from-scratch chess engine in a single packed `main.py` (4,084 bytes).

- 0x88 board, legal move generation (castling, en passant, promotion)
- Alpha-beta with iterative deepening, PVS, LMR, null-move pruning,
  check extensions, aspiration windows
- Quiescence search (captures, all evasions when in check)
- Transposition table (incremental Zobrist keys), MVV-LVA + killer +
  history move ordering
- Evaluation: full tapered PeSTO (middlegame + endgame tables and
  values, phase interpolation), bishop-pair bonus, king pawn-shield,
  tempo
- Fixed 60,000-node budget per move: fully deterministic, no clock,
  no randomness, no model calls — about 1.3 s a move on one CPU
- Robust I/O: proxy-free HTTP with timeout and retries, never exits on
  a transient error (re-syncs and falls back to a legal move)

`src.py` is the readable source; `main.py` is packed from it with the
kit's `makefun compress` (`src.py` need not be committed). License: MIT.

## What changed from v1

v1 scored 0 on the server (no games played) despite ~1087 Elo locally.
Root cause, verified by local runs: v1's per-move time ballooned in
complex positions (up to 8.6 s locally; the grader runs at half a CPU,
so moves blew the 10 s limit and the 1,800 s trial wall), and v1 had
no timeout on its HTTP calls and no recovery from a failed request —
any single failure killed the process. v2 fixes all of that: a faster
evaluator, a fixed 60k-node budget sized to fit the wall at half CPU,
incremental Zobrist keys, and hardened I/O. The evaluation is also
much stronger: real PeSTO tables (v1 used the crude simplified tables
and reused the middlegame table for the endgame), plus a corrected
null-move pruning bug (v1's flipped the side to move without updating
the Zobrist key, corrupting the transposition table).

## Local practice grades (kit v7)

| trial | Elo | W/D/L | spare | wall |
|-------|-----|-------|-------|------|
| 0 | 1253 | 2/2/4 | 256 | 828 s |
| 1 | 1396 | 4/0/4 | 297 | 845 s |
| 2 | 1253 | 1/4/3 | 116 | 784 s |

Mean Elo 1301. All three trials clear the 1,154 gate; trials 0 and 2
clear the 1,250 target. Half-CPU trial (trial 0 replayed with the
engine pinned to a CPU shared with a 50% busy loop): identical
Elo 1253, 2/2/4, wall 1,034 s, max move 2.9 s — the deterministic
node budget means the same moves at any CPU speed.
