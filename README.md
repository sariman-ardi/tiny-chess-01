[![make.fun - tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/sariman-ardi/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/sariman-ardi/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# TinyChess v2 — engine catur 4 KB

Entry untuk bounty [tiny-chess-01 di make.fun](https://make.fun/bounty/tiny-chess-01) — "Build the strongest chess engine in 4 KB of Python", hadiah total $500.

Engine original (bukan sunfish, bukan salinan engine lain), ditulis dari nol.
Board 0x88, move generator legal lengkap (rokade, en passant, promosi),
alpha-beta + quiescence, iterative deepening, node budget tetap 20.000 node
(deterministik), transposition table, evaluasi material + piece-square tables
+ bonus bishop pair.

## Anti-repetisi

Posisi yang sudah muncul 2x di jalur search dinilai remis (0). Di root,
langkah yang mengulang posisi untuk ketiga kalinya kena penalti -80 cp.

## File

- `main.py` — engine siap submit, **3.579 byte** (batas 4.096).
  Terkompresi (zlib+base85), mandiri, tidak butuh file lain.
- `LICENSE` — MIT.

## Cara menjalankan

```sh
python3 main.py
uci
isready
position startpos moves e2e4 e7e5
go
```

## Hasil tes lokal (2026-11-16, lawan proxy buatan sendiri)

- Perft: startpos d3=8902, Kiwipete d2=2039 (legalitas OK).
- vs depth-3: 6,0/6, semua skakmat, 0 remis repetisi.
- vs depth-4: 5,5/6, Elo perkiraan ~1.616, 0 remis repetisi.
- Gabungan 12 game: 11,5/12, rata-rata 2,03 detik/langkah, maks 3,43 detik
  (batas 10 detik/langkah). Elo gabungan ~1.691 (sangat tidak pasti);
  patokan yang lebih bisa dipegang: ~1.616 vs depth-4.
- Catatan: lawan adalah proxy minimax buatan sendiri, bukan house engine,
  jadi Elo ada ketidakpastian ±100 dan 12 game sampel kecil. Ambang
  latihan 1.154 terlihat terlampaui dengan margin besar, tapi bukan jaminan
  lolos vs house engine asli.
