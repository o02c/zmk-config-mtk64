# MTK64 PMW3610-BJ-Dongle ファームウェアビルドと書き込みガイド

## 概要
このレポートは、`mentako-ya/zmk-config-mtk64`リポジトリの`pmw3610-bj-dongle`ブランチにおけるファームウェアのビルドと書き込み方法について説明します。

## GitHub Actions でのファームウェア生成

### ビルド設定
- **ワークフロー**: `.github/workflows/build.yml`
- **使用するワークフロー**: `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.2`
- **ボード**: Seeeduino XIAO BLE
- **シールド**: 
  - mtk64_R + rgbled_adapter
  - mtk64_L + rgbled_adapter  
  - mtk64_FOOT + rgbled_adapter (ZMK Studio対応)
  - settings_reset

### ファームウェアの生成場所
GitHub Actionsでビルドが成功すると、ファームウェアは以下の手順でダウンロードできます：

1. GitHubリポジトリの「Actions」タブをクリック
2. リストから最新のビルドを選択
3. ビルドページの右側パネルの「Artifacts」セクションから「firmware」をダウンロード
4. ダウンロードしたZIPファイルを解凍すると、以下のUF2ファイルが含まれています：
   - `seeeduino_xiao_ble-mtk64_R-rgbled_adapter-zmk.uf2` (右側キーボード用)
   - `seeeduino_xiao_ble-mtk64_L-rgbled_adapter-zmk.uf2` (左側キーボード用)
   - `seeeduino_xiao_ble-mtk64_FOOT-rgbled_adapter-zmk.uf2` (ドングル用)
   - `seeeduino_xiao_ble-settings_reset-zmk.uf2` (設定リセット用)

## ハードウェア構成

### 使用デバイス
- **マイコン**: Seeeduino XIAO BLE (nRF52840)
- **トラックボール**: PMW3610センサー搭載
- **接続方式**: SPI通信（SCK: P0.4, MOSI/MISO: P0.5, CS: P0.9）
- **割り込みピン**: P0.10

### 特徴
- ZMK公式サポート済みボード
- UF2形式でのファームウェア書き込み対応
- ワイヤレスドングルとして使用可能
- PMW3610ドライバー（`zmk-pmw3610-driver-bj`）を使用

## ファームウェアの書き込み方法

### 手順

1. **ブートローダーモードへの切り替え**
   - XIAO BLEのリセットボタンをダブルクリック
   - PCに新しいUSBストレージデバイスとして認識される

2. **ファームウェアの書き込み**
   - 対応するUF2ファイルをUSBストレージのルートディレクトリにコピー
   - 書き込み対象：
     - **右側キーボード**: `seeeduino_xiao_ble-mtk64_R-rgbled_adapter-zmk.uf2`
     - **左側キーボード**: `seeeduino_xiao_ble-mtk64_L-rgbled_adapter-zmk.uf2`
     - **ドングル**: `seeeduino_xiao_ble-mtk64_FOOT-rgbled_adapter-zmk.uf2`

3. **自動再起動**
   - ファームウェアの書き込みが完了すると、自動的にUSBストレージがアンマウントされ、新しいファームウェアで再起動

### 設定のリセット
接続に問題がある場合は、`settings_reset`ファームウェアを使用して設定をリセットできます：
1. `seeeduino_xiao_ble-settings_reset-zmk.uf2`を書き込む
2. 自動的に設定がリセットされる
3. 通常のファームウェアを再度書き込む

## 注意事項

1. **初回テスト**: ワイヤレス接続を試す前に、USB接続での動作確認を推奨

2. **バッテリー消費**: GD25Q16チップによる消費電力問題がある場合は、オーバーレイファイルに以下を追加：
   ```dts
   &qspi { 
     status = "disabled"; 
   }
   ```

3. **ブートローダーマクロ**: 将来のファームウェア更新のため、`&bootloader`アクションを参照するマクロを設定することを推奨

4. **PMW3610設定**:
   - CPI: 1000
   - Force-awakeモード有効
   - 4msサンプリングモード対応（USB直接接続時の250Hz用）

## 参考リンク
- [ZMK公式ドキュメント](https://zmk.dev/docs/user-setup)
- [GitHub Actions ページ](https://github.com/mentako-ya/zmk-config-mtk64/actions)
- [ZMKハードウェアサポート](https://zmk.dev/docs/hardware)