# シミュレーション特論 第1回レポート課題 (26AK01) 測定結果

## システム情報

| 項目 | 値 |
|------|-----|
| 計算ノード名 | ysnd00.edu.tut.ac.jp |
| CPU | AMD EPYC 9254 24-Core Processor（2ソケット、48コア） |
| メモリ量 | 250 GiB |
| コンパイルコマンド（OpenMP版） | `icx -fopenmp -std=gnu89 -O2 -o bin/himeno_omp src/himenoBMTxpa.c` |
| コンパイルコマンド（MPI版） | `mpicc -cc=icx -O3 -std=gnu89`（Makefile.sample を基にビルド） |

測定日時: 2026-06-16  
ジョブ投入: `qsub jobs/run_all.pbs`（課題1〜3）、`qsub jobs/mpi_extension.pbs`（発展課題）

---

## 【1】グリッドサイズと性能（OMP_NUM_THREADS=1, -O2）

| Grid size | CPU time (sec.) | Loop executed | MFLOPS |
|-----------|-----------------|---------------|--------|
| S | 0.943088 | 115 | 2008.07 |
| M | 59.229043 | 858 | 1986.12 |
| L | 60.139006 | 101 | 1878.80 |

ログ: `results/raw/task1_S_t1.log`, `task1_M_t1.log`, `task1_L_t1.log`

**所見:** グリッドが大きくなるほど1ループあたりの計算量が増えるため、リハーサルで決まるループ回数は減少する。Sサイズは約1分に満たない短時間で終了するが、M/Lは約60秒の測定時間に調整される。

---

## 【2】最適化オプション（Grid=S, OMP_NUM_THREADS=1）

| Option | CPU time (sec.) | Loop executed | MFLOPS |
|--------|-----------------|---------------|--------|
| -O0 | 57.237897 | 1461 | 420.34 |
| -O1 | 48.274247 | 6630 | 2261.68 |
| -O2 | 49.703904 | 6045 | 2002.80 |
| -O3 | 49.476148 | 6031 | 2007.36 |

ログ: `results/raw/task2_O0_t1.log` 〜 `task2_O3_t1.log`

**所見:** -O0（最適化なし）はMFLOPSが大幅に低下。-O1以上で大きく改善し、-O2/-O3はほぼ同等の性能となる。

---

## 【3】OpenMP スレッド数（Grid=S, -O2）

| Number of OpenMP threads | CPU time (sec.) | Loop executed | MFLOPS |
|--------------------------|-----------------|---------------|--------|
| 1 | 49.915502 | 6075 | 2004.21 |
| 2 | 40.914857 | 10099 | 4064.72 |
| 4 | 31.634604 | 14898 | 7755.30 |
| 8 | 17.128982 | 14860 | 14286.31 |

ログ: `results/raw/task3_t1.log` 〜 `task3_t8.log`

**所見:** スレッド数を増やすとCPU時間は短縮され、MFLOPSは増加する。姫野ベンチのOpenMP版はヤコビ反復の内側ループに並列化が入っており、本環境ではスレッド並列が有効に機能している。

---

## 【発展課題】MPI プロセス数（Grid=S）

| Number of MPI processes | CPU time (sec.) | Loop executed | MFLOPS |
|-------------------------|-----------------|---------------|--------|
| 1 | 58.945497 | 77996 | 21789.86 |
| 2 | 53.178672 | 138258 | 42813.98 |
| 4 | 47.673682 | 250549 | 86545.96 |
| 8 | 37.696982 | 337259 | 147329.53 |

プロセス分割: 1→(1,1,1), 2→(2,1,1), 4→(2,2,1), 8→(2,2,2)

ログ: `results/raw/mpi_S_p1.log` 〜 `mpi_S_p8.log`

---

## 実施した作業の概要

1. RIKEN 姫野ベンチマークのソースコードを取得（OpenMP版・MPI版）
2. Intel oneAPI (`module load intel/2025`) で OpenMP 版をコンパイル
3. PBS ジョブ（Eduq キュー、8 CPU、16 GB）で計算ノード上にて課題1〜3を一括実行
4. Intel MPI (`module load intelmpi/2025`) で MPI 版をビルド・実行（発展課題）
5. `scripts/parse_output.sh` でログから CPU 時間・ループ回数・MFLOPS を抽出

## 結果の再確認コマンド

```bash
bash scripts/parse_output.sh results/raw/task1_*.log
bash scripts/parse_output.sh results/raw/task2_*.log
bash scripts/parse_output.sh results/raw/task3_*.log
bash scripts/parse_output.sh results/raw/mpi_*.log
```
