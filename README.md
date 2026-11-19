[![make.fun - tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/sariman-ardi/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/sariman-ardi/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# TinyChess v4 — engine catur 4 KB (klien HTTP)

Entry untuk bounty [tiny-chess-01 di make.fun](https://make.fun/bounty/tiny-chess-01) — "Build the strongest chess engine in 4 KB of Python", hadiah total $500.

Engine original (bukan sunfish, bukan salinan engine lain), ditulis dari nol.
Board 0x88, move generator legal lengkap (rokade, en passant, promosi),
alpha-beta + quiescence (termasuk semua balasan saat skak), iterative
deepening depth 1–5, node budget tetap 30.000 node (deterministik),
transposition table, killer moves + history heuristic, evaluasi material +
piece-square tables + bonus bishop pair, anti-repetisi.

## Kenapa v4 (bukan v2/v3)

Round 3 gagal total (practice grade 0) bukan karena engine lemah, tapi karena
engine bicara UCI lewat stdin/stdout sementara env situs HANYA bicara HTTP.
v4 mengganti loop utama menjadi klien HTTP seperti `starter/main.py` di kit:
`POST $MAKEFUN_ENV_URL/game {}` lalu `POST /move {game, move}` sampai
status `done`. Inti search/evaluasi tetap dari v2.

## File

- `main.py` — engine siap submit, **3.299 byte** (batas 4.096), hasil pack
  resmi `makefun compress` (stub LZMA) dari `main_src.py`.
- `main_src.py` — source yang bisa dibaca (8.178 byte).
- `LICENSE` — MIT.

## Hasil uji trial-0 penuh (env asli kit, lokal)

- Perft: startpos d3=8902, Kiwipete d1=48, d2=2039 (legalitas OK).
- Trial-0 (6 game vs depth-2, depth-3, depth-4, nodes-100k, nodes-250k,
  nodes-500k): **2 menang, 1 remis, 3 kalah = 2,5 poin, Elo ~1.190** —
  di atas ambang practice 1.154. Menang vs depth-2 dan nodes-250k, remis vs
  depth-4. Semua 6 game dimainkan (working=1), tanpa timeout; total trial
  ~18,5 menit (batas 30 menit).
- Riwayat versi: v4 awal (12 ribu node) 1,0 poin (game 1 kalah karena waktu
  akibat kesalahan harness tes lokal, bukan engine); v4.1 (20 ribu node +
  killer + balasan-skak) 2,0 poin; v4.2 (bonus bidak bebas) 1,0 poin — bonus
  itu dibuang lagi; v4.3 (30 ribu node + history) 2,5 poin.
