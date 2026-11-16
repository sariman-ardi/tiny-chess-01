# TinyChess v2 — engine catur 4 KB untuk bounty make.fun `tiny-chess-01`

Engine catur original (bukan sunfish/GPL, bukan salinan engine lain), ditulis
dari nol untuk bounty "Build the strongest chess engine in 4 KB of Python".
Board 0x88, move generator lengkap dan legal (termasuk rokade, en passant,
promosi), alpha-beta + quiescence (hanya capture), iterative deepening dengan
node budget tetap 20.000 node (deterministik, bukan berbasis jam), transposition
table, move ordering (TT move, MVV-LVA, promosi), evaluasi material +
piece-square tables + bonus bishop pair.

## Anti-repetisi

Kelemahan engine sebelumnya adalah banyak remis repetisi vs lawan setara.
Engine ini mengatasi itu sejak awal:

- Di search, posisi yang sudah muncul 2x di jalur pencarian dinilai remis (0).
- Di root, langkah yang langsung mengulang posisi untuk ketiga kalinya
  mendapat penalti −80 cp, jadi hanya dipilih kalau alternatifnya jelas
  lebih buruk.
- Histori posisi game dibawa lewat UCI `position ... moves ...`.

## File

| File | Keterangan |
| --- | --- |
| `main.py` | Engine siap submit, **3.579 byte** (batas 4.096). Versi terkompresi (zlib+base85, `exec` saat start) dari `main_src.py`; fungsinya identik. |
| `main_src.py` | Versi golf dari engine yang sama, terbaca sebagian (6.948 byte). |
| `src.py` | Versi lengkap yang mudah dibaca dari engine yang sama (17.079 byte), dengan komentar dan perft. |
| `compress.py` | Script build: `main_src.py` -> `main.py`. Jalankan `python3 compress.py` setelah mengubah `main_src.py`. |
| `test.py` | Harness tes lokal (lawan greedy dan depth-2 buatan sendiri). |
| `test_v2_depth34.py` | Harness tes `main.py` via UCI vs depth-3/depth-4 mate-aware; log `test_log_v2_depth3.txt` / `test_log_v2_depth4.txt`. |
| `LICENSE` | MIT. |
| `README.md` | File ini. |

## Cara menjalankan

Engine berbicara UCI di stdin/stdout, dan juga menerima satu baris FEN:

```sh
python3 main.py
uci
isready
position startpos moves e2e4 e7e5
go
# -> bestmove b1c3

# atau langsung FEN:
echo "rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1" | python3 main.py
# -> b1c3
```

Jalankan tes:

```sh
python3 test.py
```

## Hasil tes (lokal, 2026-11-16, lawan proxy buatan sendiri — bukan house engine)

Perft (legalitas move generator, `src.py`):

- startpos d1=20, d2=400, d3=8902 ✓
- Kiwipete d1=48, d2=2039 ✓
- Posisi 3 (8/2p5/...) d1=14, d2=191 ✓
- Posisi 4 (r3k2r/Pppp1ppp/...) d1=6, d2=264 ✓
- Posisi 5 (rnbq1k1r/pp1Pbppp/...) d1=44, d2=1486 ✓

Hasil game (6 game, opening berbeda, engine gantian putih/hitam):

- vs greedy: **W6 D0 L0, skor 6,0/6**, rata-rata 1,49 detik/langkah, maks 2,81 detik.
- vs depth-2 (proxy pertama, lihat catatan): **W6 D0 L0, skor 6,0/6**, rata-rata 1,72 detik/langkah, maks 2,53 detik.
- vs depth-2 benar (mate-aware, seri ulang setelah proxy diperbaiki): **W6 D0 L0, skor 6,0/6**, rata-rata 1,58 detik/langkah, maks 2,60 detik. Game lebih panjang (26–75 langkah) daripada seri pertama, jadi hasil ini lebih bisa dipercaya.

Catatan jujur: proxy depth-2 pertama ternyata cacat — evaluasi daunnya tidak
mendeteksi skakmat, jadi ia kalah cepat di 2 game (contoh: 1.d4 d5 2.Nc3 Nc6
3.Nf3 Nf6 4.e4 dxe4 5.Nd2 Qd4 6.Nxe4 Nxe4 7.Bb5 Qxf2#, skakmat sungguhan,
sudah diverifikasi langkah demi langkah). Angka 6/6 vs depth-2 pertama itu
melebih-lebihkan kekuatan vs depth-2 yang benar, karena itu seri diulang vs
depth-2 yang benar (mate-aware) dan hasilnya tetap 6/6 — lihat
`test_results_depth2_true.txt` / `test_log_depth2_true.txt`.

Kecepatan memenuhi target: rata-rata < 2 detik/langkah, maks < 10 detik/langkah
(batas harness 10 detik/langkah).

### vs depth-3 / depth-4 (2026-11-16, `main.py` asli via UCI, lawan proxy mate-aware)

Seri 12 game, 6 opening berbeda, engine gantian putih/hitam
(`test_v2_depth34.py`, log `test_log_v2_depth3.txt` / `test_log_v2_depth4.txt`):

- vs depth-3 (anchor Elo 1073): **W6 D0 L0, skor 6,0/6** — semua menang
  skakmat (51–99 langkah), **0 remis repetisi**, rata-rata 2,07 detik/langkah,
  maks 3,43 detik. Sapu bersih, jadi tidak ada Elo berhingga; sebagai
  perbandingan, skor 5,5/6 vs anchor ini setara Elo ~1.490.
- vs depth-4 (anchor Elo 1199): **W5 D1 L0, skor 5,5/6, Elo perkiraan ~1.616**
  — 5 menang skakmat (38–128 langkah), 1 remis, **0 remis repetisi**,
  rata-rata 2,00 detik/langkah, maks 3,30 detik. Satu-satunya remis (game 3,
  Sicilian, Putih) adalah batas 160 langkah harness, bukan repetisi:
  game endgame panjang yang tidak selesai, bukan posisi berulang.
- Gabungan 12 game: **W11 D1 L0, skor 11,5/12, 0 remis repetisi**.
  Elo gabungan (method-of-moments dari 11,5/12 vs kedua anchor): **~1.691**,
  dengan catatan angka ini sangat tidak pasti karena seri depth-3 sapu bersih.
  Patokan yang lebih bisa dipegang: Elo vs depth-4 saja, ~1.616.
  Rata-rata gabungan 2,03 detik/langkah (541 langkah engine), maks 3,43 detik.

Banding vs engine lama (referensi saja): engine lama 4,0/6 vs depth-3
(Elo 1.193) dan 2,5/6 vs depth-4 (Elo 1.141), dengan 8 dari 12 game berakhir
remis repetisi. Engine-v2 jelas lebih baik di kedua level — terutama soal
repetisi: 0 remis repetisi dari 12 game, berbanding 8 dari 12 pada engine lama.
Anti-repetisi (penalti −80 cp di root + posisi berulang dinilai remis di
search) tampaknya bekerja: game-game v2 berakhir skakmat atau berjalan
panjang, bukan berputar di posisi yang sama.

Catatan jujur: lawan di sini proxy minimax buatan sendiri, bukan house engine
asli, jadi perkiraan Elo ada ketidakpastian sekitar ±100, dan 12 game adalah
sampel kecil. Proxy depth-3 memakai minimax tanpa pengurutan langkah;
proxy depth-4 memakai minimax yang sama (kedalaman dan evaluasi sama)
ditambah pengurutan MVV-LVA + alpha root, karena versi tanpa pengurutan
terlalu lambat (game 1 tidak selesai dalam 9 menit). Pengurutan tidak
mengubah nilai minimax — hanya tie-break antar langkah bernilai sama yang
bisa berbeda — jadi kekuatan lawan tidak berubah, hanya kecepatannya
(0,9–1,5 detik/langkah depth-4 setelah diurutkan, dicek menemukan skakmat
dalam 1 langkah pada posisi Scholar's mate).

Belum dites: vs nodes-100k/250k/500k (house gauntlet). Ambang latihan 1.154
terlihat terlampaui dengan margin besar vs proxy ini, tapi itu bukan jaminan
lolos vs house engine asli.

## Status

- `main.py` final: 3.579 byte, lolos tes jalan dasar (UCI + FEN), legalitas
  dan evaluasi terverifikasi sama dengan `src.py` pada posisi uji.
- Tes vs depth-3/depth-4 selesai 2026-11-16: W11 D1 L0 dari 12 game,
  0 remis repetisi, Elo gabungan perkiraan ~1.691 (vs proxy buatan sendiri,
  ketidakpastian ~±100). Layak lanjut ke repo + submit; belum dites vs
  nodes-100k/250k/500k.
- Belum submit ke make.fun, belum ada repo GitHub untuk engine ini.
