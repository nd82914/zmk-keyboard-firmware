# MOTTABLEKEY ZMK ファーム ビルドガイド

## 構成状態

✅ **完成した項目:**

- ✅ roBa と同等の構成に移行完了
- ✅ `build.yaml` (CI ビルド定義)
- ✅ `config/west.yml` (ZMK マニフェスト)
- ✅ `config/mottablekey.keymap` (キーマップ定義)
- ✅ `config/boards/shields/mottablekey/` (ハードウェア定義)
  - `Kconfig.shield`, `Kconfig.defconfig`
  - `mottablekey.dtsi` (基本定義)
  - `mottablekey_L.overlay`, `mottablekey_L.conf` (左半)
  - `mottablekey_R.overlay`, `mottablekey_R.conf` (右半)
- ✅ `.github/workflows/build.yml` (自動ビルド)
- ✅ `zephyr/module.yml`

## 主要ファイル説明

| ファイル | 目的 | 関係 |
| --- | --- | --- |
| **build.yaml** | GitHub Actions や west build の対象 (board + shield 組み合わせ) | CI・ローカルビルド両対応 |
| **config/west.yml** | west マニフェスト (ZMK・Zephyr 依存関係) | 初期化・更新時に参照 |
| **config/mottablekey.keymap** | キーレイアウト・レイヤー定義 | キー配置・レイヤー決定 |
| **config/boards/shields/mottablekey/mottablekey.dtsi** | GPIO・matrix-transform の基本定義 | ハードウェア固有設定 |
| **mottablekey_L/R.overlay** | 左右別の GPIO 割当 | 分割キーボード用 |
| **mottablekey_L/R.conf** | 左右別の CONFIG 設定 (マウスセンサなど) | PMW3610 設定等 |
| **.github/workflows/build.yml** | GitHub Actions トリガー設定 | push/PR 時に自動ビルド |

## ローカルビルド手順

### 1. west 環境構築

```bash
cd zmk-keyboard-firmware

# west 初期化（config/west.yml を参照）
python -m west init -l config

# 依存関係ダウンロード
python -m west update

# Zephyr 設定エクスポート
python -m west zephyr-export
```

### 2. ビルドコマンド

```bash
# 右半（トラックボール付き）
west build -s zmk/app -d build/right -b seeeduino_xiao_ble -- -DSHIELD=mottablekey_R

# 左半
west build -p -s zmk/app -d build/left -b seeeduino_xiao_ble -- -DSHIELD=mottablekey_L
```

成功時のアーティファクト:

- `build/right/zephyr/zmk.uf2`
- `build/left/zephyr/zmk.uf2`

### 3. フラッシュ

XIAO nRF52840 をブートローダモードで接続し、上記 UF2 をドラッグ&ドロップ。

## GitHub Actions での自動ビルド

push / PR 時に `.github/workflows/build.yml` が実行され、
右左両方のファームが自動ビルド・リリースアーティファクトに上がります。

## トラブルシューティング

### west update 時に git エラーが出る場合

```text
FATAL ERROR: command exited with status 128: rev-parse 'FETCH_HEAD^{commit}^{commit}'
```

以下のいずれかを試す:

- `git config --global core.longpaths true`
- Windows Defender 除外設定
- Git Bash ではなく WSL2 を使用

### キーマップが反映されない

- `config/mottablekey.keymap` の bindings 行数がキー数に一致しているか確認
- `default_transform` の map と col-offset を確認
- PMW3610 (トラックボール) 関連は `mottablekey_R.conf` / `mottablekey_R.overlay` に集約

## 今後の調整項目

1. **キー数が異なる場合**: `mottablekey.dtsi` の `default_transform` rows/columns と map を修正
2. **GPIOピン変更**: `mottablekey_L/R.overlay` の col-gpios / row-gpios を修正
3. **encoder 有効化**: `mottablekey.dtsi` の `left_encoder` / `right_encoder` status を "okay" に
4. **PMW3610 設定**: `mottablekey_R.conf` / `mottablekey_R.overlay` の各パラメータ微調整

---

**参考**: [zmk-config-roBa](https://github.com/kumamuk-git/zmk-config-roBa)

