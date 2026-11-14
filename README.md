# tiny-chess-01: a 4 KB Python chess engine

An original 0x88 chess engine for the make.fun tiny-chess-01 bounty.
`main.py` is the whole engine (3,974 bytes as committed): the readable
source packed with the kit's own `makefun compress` (makefun-compress/v1
stub around an LZMA stream of the source). The compact readable source
is `source.py` (uncommitted; the kit's bundle sends it beside the
session transcripts, never published).

## Engine

- 0x88 board, incremental material + piece-square eval (Michniewski
  tables), bishop-pair bonus, king pawn shield, endgame king table.
- Negamax with PVS, iterative deepening under a fixed node budget
  (100k/80k/50k/30k by game phase), quiescence on captures, promotions
  and check evasions, with delta pruning in quiescence (a capture is
  skipped when stand-pat + captured value + promotion value + 200
  cannot reach alpha; never applied to check evasions).
- Null-move pruning, LMR on late quiet moves, futility pruning,
  check extensions, aspiration windows at the root.
- Transposition table (300k entries), killer moves, history heuristic,
  repetition awareness (repeated positions score 0).
- Deterministic: fixed node budgets, no clock, no randomness.
  Plays through the env at MAKEFUN_ENV_URL (protocol in the kit).

## Local practice results (kit grader, 3 trials)

v7 (compact source, no delta pruning): 1396 / 1464 / 1396,
mean 1418.7, vs the sunfish baseline mean of 1325.

v8 (this entry: v7 + delta pruning in quiescence):
Trial 0: Elo 1530 (5-0-3; first trial-0 win vs nodes-250k, 53 plies).
Trial 1: Elo 1530 (5-0-3; includes a win vs nodes-2M, 81 plies).
Trial 2: Elo 1326 (3-1-4).
Mean 1462.0, W/D/L 13-1-10 — vs v7's 1418.7 (11-3-10), +43 Elo.
Delta pruning cuts fixed-depth node counts by 10–39% with identical
search values (middlegame depth 5: 50,362 → 45,231 nodes), buying
deeper search inside the same budget. Still far from the ~1750
top-20 line. The kit-packed entry (3,974 bytes) was verified against
the readable source: perft exact, fixed-depth search values and node
counts identical, same best move at the 100k budget; the packed file
was also graded on trial 0.

## Build

The readable source is `source.py`: a compact rewrite of the v6 engine
(same search, eval, move generation and node budgets — only style
changed: short identifiers, dense formatting, piece-square tables as
one letter per square), plus delta pruning in quiescence (v8).
v6 (`engine/main.py` in the build workspace, 25,075 bytes) is kept as
the reference. `source.py` (9,808 bytes) is packed with the kit's
`makefun compress` — run as its own command after the source is
written, and again after every change to the source:

    makefun compress source.py --out main.py

Earlier packaging (custom minifier + lzma + latin-1, 4,081 bytes)
graded identically too, but its bytes are not UTF-8 text, so the
transcript check cannot count any authored line in it. See
BUILD_LOG.md in the build workspace for the full history
(v1–v8, packaging notes, rejected candidates).
