# MOTTABLEKEY ZMK Config

`zmk-config-roBa` と同系統の構成で、GitHub Actions / ローカル `west build` の両方に対応しています。

## 構成

- `build.yaml`: CI ビルド対象（board + shield）
- `config/west.yml`: ZMK マニフェスト
- `config/mottablekey.keymap`: レイヤーとキー割当
- `config/boards/shields/mottablekey/*`: シールド定義（`Kconfig*`, `*.overlay`, `*.conf`, `*.dtsi`）
- `.github/workflows/build.yml`: push/PR 時の自動ビルド

## ローカルビルド

```bash
cd zmk-keyboard-firmware
west init -l config
west update
west zephyr-export

# Right
west build -s zmk/app -b seeeduino_xiao_ble -- -DSHIELD=mottablekey_R

# Left
west build -p -s zmk/app -b seeeduino_xiao_ble -- -DSHIELD=mottablekey_L
```
