# tiny-chess-01: a 4 KB Python chess engine

An original 0x88 chess engine for the make.fun tiny-chess-01 bounty.
`main.py` is the whole engine (3,941 bytes as committed): the readable
source packed with the kit's own `makefun compress` (makefun-compress/v1
stub around an LZMA stream of the source). The compact readable source
is `source.py` in this repo (uncommitted; the kit's bundle sends it
beside the session transcripts, never published).

## Engine

- 0x88 board, incremental material + piece-square eval (Michniewski
  tables), bishop-pair bonus, king pawn shield, endgame king table.
- Negamax with PVS, iterative deepening under a fixed node budget
  (100k/80k/50k/30k by game phase), quiescence on captures, promotions
  and check evasions.
- Null-move pruning, LMR on late quiet moves, futility pruning,
  check extensions, aspiration windows at the root.
- Transposition table (300k entries), killer moves, history heuristic,
  repetition awareness (game history + search path score 0, TT entries
  shaped by a repetition are not stored).
- Deterministic: fixed node budgets, no clock, no randomness.
  Plays through the env at MAKEFUN_ENV_URL (protocol in the kit).

## Local practice results (kit grader, 3 trials) — v7 (compact source)

- Trial 0: Elo 1396 (4-0-4)
- Trial 1: Elo 1464 (4-1-3)
- Trial 2: Elo 1396 (3-2-3)
- Mean 1418.7, vs the sunfish 4 KB baseline mean of 1325
  (1326 / 1396 / 1253). Practice floor to enter the final is 1154.
- Identical to v6 on every trial (same W/D/L, same Elo): the compact
  rewrite changed no logic. The kit-packed entry (3,941 bytes) was
  graded on trial 0 and gave the same result as the readable source
  (Elo 1396, 4-0-4, same plies every game).

## Build (v7)

The readable source is `source.py`: a compact rewrite of the v6 engine
(same search, eval, move generation and node budgets — only style
changed: short identifiers, dense formatting, piece-square tables
encoded as one letter per square, since every table value is a multiple
of 5). v6 (`engine/main.py` in the build workspace, 25,075 bytes) is
kept as the reference. `source.py` (9,679 bytes) is packed with the
kit's `makefun compress` — run as its own command after the source is
written, and again after every change to the source:

    makefun compress source.py --out main.py

The packed entry (3,941 bytes) was verified to grade identically to the
readable source. Earlier packaging (custom minifier + lzma + latin-1
payload, 4,081 bytes) graded identically too, but its bytes are not
UTF-8 text, so the transcript check cannot count any authored line in
it — hence the switch to the kit's packer. See BUILD_LOG.md in the
build workspace for the full history (v1–v6, packaging notes).
