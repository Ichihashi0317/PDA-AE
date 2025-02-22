# PDA-AE
PDA
- MIMO通信
- 検出器にPDA
- 変調器は従来のQAMかニューラルネットワーク (NN) で構成か選べる
- 検出器は従来のPDAか適応スケールビリーフ (ASB) か深層展開 (DU)か選べる

![Image](https://github.com/user-attachments/assets/f3a9b51e-3ac4-4d4f-9371-2bf58fbbfd84)

## シミュレーションパラメータの説明
### 学習・テスト 共通設定
#### モード選択
- use_nnmod (bool) - FalseならQAM変調器，TrueならNN変調器になる
- ASB_mode (str) - 検出器の選択
  - "PDA" - 従来のPDA．
  - "ASB" - ASBを用いたPDA．
  - "DU" - DUを適用したPDA．初期値はASBのスケーリング係数の設定値と同じになる．
  <!-- - "ASB" - ASBを用いたPDA．繰り返し $i$回目におけるスケーリング係数 $\mu^{(i)}$は，最大繰り返し回数 $I$，調整パラメータ $d_1, d_2$を用いて次式で与えられる．
  <br>
  $$\mu^{(i)} = d_1 \left( \dfrac{i}{I} \right) ^ {d_2}, \quad i \in \\{1,2,\cdots,I\\}$$
  - "DU" &thinsp; - DUを適用したPDA．初期値は上式で与えられる． -->
#### 基本設定
- M (int) - 送信アンテナ本数
- N (int) - 受信アンテナ本数
- Q_ant (int) - <br> 
1アンテナあたりの多値数．use_nnmod = False の場合， $2$または $4^n$ ($n$は任意の自然数) で指定．use_nnmod = True の場合は2以上の任意の自然数で指定．
- Kd (int) - データフレーム長
- num_joint_ant (int, float) - NN変調時，信号検出時のアンテナ結合本数．現在0.5または1のみ指定可能．
  - 0.5 - シンボルの実部（同相成分）と虚部（直行成分）を別々に検出する．NN変調器の場合，格子状の信号点配置になる．
  - 1 &nbsp;&thinsp; - シンボルの実部と虚部を合わせて検出する．NN変調器の場合，非格子状の信号点配置になる．
- niter_PDA (int) - PDAの繰り返し回数
- d1 (float) - ASBのスケーリング係数の調整パラメータ[^1][^2]．ASB_modeが"ASB"または"DU"のときのみ指定可能．
- d2 (float) - ASBのスケーリング係数の調整パラメータ[^1][^2]．ASB_modeが"ASB"または"DU"のときのみ指定可能．
[^1]: https://ieeexplore.ieee.org/abstract/document/8543847
[^2]: https://www.ieice.org/publications/ken/summary.php?contribution_id=128665&society_cd=CS&ken_id=CS&year=2024&presen_date=2024-01-18&schedule_id=8121&lang=jp&expandable=1

### 学習設定
#### 変調器・検出器 共通設定
- epochs (int) - エポック数
- mbs (int) - ミニバッチサイズ
- EsN0_train (float) - 学習時の $E_\mathrm{s} / N_0 \ [\mathrm{dB}]$
#### 変調器
- hidden_depth (int) - 中間層（隠れ層）の深さ
- hidden_dim (int) - 中間層（隠れ層）の次元
- ind_cons (bool) - Falseなら各アンテナの信号点配置は同じになり，Trueなら独立した信号点配置になる
<!--  -->
- lr_mod (float) - 変調器の学習率
- drop_start_mod (int) - 変調器の学習率スケジューラの減衰開始エポック
- drop_factor_mod (float) - 変調器の学習率スケジューラの減衰率
#### 検出器
- lr_det (float) - 検出器の学習率
- drop_start_det (int) - 検出器の学習率スケジューラの減衰開始エポック
- drop_factor_det (float) - 検出器の学習率スケジューラの減衰率

### テスト設定
- EsN0_test (list, ndarray) - テスト時の $E_\mathrm{s} / N_0 \ [\mathrm{dB}]$を並べた一次元配列
- nloop_max (int, float) - $E_\mathrm{s}/N_0$毎のシミュレーション回数 <br>
十分なエラー数が得られるまで計算をしたい場合はfloat('inf')に設定する[^3]．
後述する，シンボルエラー数がSE_maxに達するまで計算を続けたい場合はfloat('inf')に設定する[^3]．
- SE_max (int, float) - 早期終了条件 <br>
シンボルエラー数がこの設定値に達すると，その $E_\mathrm{s} / N_0$でのシミュレーションを早期終了する．早期終了しない場合はfloat('inf')に設定する[^3]．
[^3]: nloop_maxとSE_maxを両方ともfloat('inf')に設定すると計算が終わらないので，片方は有限値に設定すること．

### 並列処理設定
- nworker (int) - 並列ワーカー数（並列処理は計算機サーバーでの実行時のみ行う）
