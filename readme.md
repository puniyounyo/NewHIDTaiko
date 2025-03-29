# 太鼓の達人コントローラー仕様書

**1. はじめに**

本仕様書は、Arduino Leonardo互換のPro Microマイコン、圧電素子、およびOLEDディスプレイ（SSD1306）を使用した、PCおよびNintendo Switchに対応した太鼓の達人コントローラーの仕様を定めるものです。振動センサー（圧電素子）による入力検出、モード切り替え機能、感度調整機能、誤入力防止機能（入力強度学習型）、同時押し判定機能、およびOLEDディスプレイによるステータス表示機能を備えることを目的とします。

**2. システム概要**

本コントローラーは、Pro Microマイコンを中核とし、4つの圧電素子からのアナログ入力を処理し、PCまたはNintendo Switchに対して太鼓の入力信号を送信します。起動時のジャンパピンの状態によって動作モード（PCモード/SWモード）を切り替えます。ユーザーはOLEDディスプレイと操作によって設定モードを通じて、圧電素子の反応閾値、同時押し判定の時間、誤入力防止の感度を調整できます。また、各センサーの入力強度を学習させる機能も備えています。設定値はEEPROMに保存され、電源を切っても保持されます。動作モードや設定状態はOLEDディスプレイに表示されます。

**3. ハードウェア構成**

* **マイコン:** Arduino Leonardo互換 Pro Micro
* **圧電素子:** 4個 (左右カッ、左ドン、右ドンに対応)
* **ジャンパピン:** モード切り替え用
* **OLEDディスプレイ:** SSD1306 (または互換性のあるもの、I2C接続を想定)
* **その他:** 配線材、筐体 (別途検討)

**4. ソフトウェア仕様 (ファームウェア)**

**4.1. 動作モード**

起動時にジャンパピンの状態を読み取り、以下のいずれかのモードで動作します。

* **PCモード:** USB-HIDデバイスとしてPCに認識され、特定のキーボード入力を送信します。
* **SWモード:** Nintendo Switchに対応したコントローラーとして認識され、特定のボタン入力を送信します。

**4.2. 入力検出 (圧電素子)**

* 4つの圧電素子をPro Microのアナログ入力ピンに接続します。
* 各圧電素子からのアナログ信号を読み取り、デジタルフィルタ（ローパス、移動平均など）を適用してノイズを除去します。
* フィルタリング後の信号値が、設定された閾値を超えた場合に「入力あり」と判定します。
* 各センサーと入力の対応は以下の通りとします。
    * アナログ入力 1: 左カッ
    * アナログ入力 2: 左ドン
    * アナログ入力 3: 右ドン
    * アナログ入力 4: 右カッ

**4.3. 各入力時の動作**

| 入力   | PCモード出力 (キーボード) | SWモード出力 (ボタン) |
| ------ | ----------------------- | --------------------- |
| 左カッ | `d`                     | `ZL`                  |
| 左ドン | `f`                     | `L`                   |
| 右ドン | `j`                     | `R`                   |
| 右カッ | `k`                     | `ZR`                  |

**4.4. 同時押し判定 (センサー追加なし)**

* 左右の同じ種類のセンサー（左カッと右カッ、左ドンと右ドン）で、極めて短い設定された時間差以内に閾値を超える入力が検出された場合に、同時押しと判定します。
* 同時押しと判定された場合の出力は以下の通りです。
    * 左カッ + 右カッ: 大カッ (PCモード: `d` + `k` 同時押し, SWモード: `ZR` + `ZL` 同時押し)
    * 左ドン + 右ドン: 大ドン (PCモード: `f` + `j` 同時押し, SWモード: `R` + `L` 同時押し)

**4.5. 誤入力防止 (入力強度学習型)**

* **セットアップモードでの学習:** 設定モード中に、各センサーに対してユーザーに数回叩いてもらい、その際の入力信号のピーク値の平均値または最大値を「通常の入力強度」として学習し、EEPROMに保存します。
* **誤入力の判定:** 通常動作時、センサーからの入力信号のピーク値が、学習した基準値よりも設定された割合（誤入力防止感度）よりも低い場合に、誤入力と判定し、出力を無視します。
* 誤入力防止感度は、設定モードで調整可能です。

**4.6. 設定モード**

* 特定の操作（例: 特定のジャンパピンの状態での起動、特定のボタン同時押し）で設定モードに移行します。
* OLEDディスプレイに設定メニューを表示し、ユーザーは操作によって以下の項目を調整できます。
    * 各圧電素子の反応閾値
    * 同時押し判定の時間差
    * 誤入力防止の感度
    * 入力強度学習の実行 (各センサーごとの再学習)
* 設定値はOLEDディスプレイにリアルタイムに表示され、調整後にEEPROMに保存されます。

**4.7. ステータス表示 (OLEDディスプレイ)**

* **通常動作時:** 現在の動作モード (`PC`/`SW`) を表示します。
* **設定モード時:** 設定メニュー、選択中の項目、現在の設定値を表示します。
* **入力強度学習モード時:** 学習対象のセンサーと進捗状況を表示します。
* **起動時:** 起動メッセージなどを表示します。

**4.8. EEPROM保存データ**

* 各圧電素子ごとの入力判定閾値 (整数値)
* 同時押し判定時間差 (マイクロ秒単位)
* 各センサーごとの通常の入力強度の基準値 (整数値)
* 誤入力防止感度 (パーセンテージ)

**5. 画面設計 (概要)**

* **OLEDディスプレイ表示:**
    * 起動画面
    * 通常動作画面 (例: `MODE: PC` or `MODE: SW`)
    * 設定メニュー画面 (例: `1. 閾値`, `2. 同時押し`, `3. 誤入力`, `4. 学習`)
    * 設定項目表示画面 (例: `閾値(左カッ): 100`)
    * 設定値編集画面 (例: `閾値(左カッ): 100 <+->`)
    * 学習モード画面 (例: `学習: 左ドン`, `叩いてください...`)
* 視認性の高いフォントと情報配置を考慮します。

**6. 非機能要件**

* **応答性:** 入力から出力までの遅延を最小限に抑えること。
* **安定性:** 安定して動作し、誤動作が少ないこと。
* **省電力性:** (特にSWモードでのバッテリー消費を考慮)
* **保守性:** 設定変更やファームウェアのアップデートが比較的容易であること。

**7. その他**

* PCとの接続はUSBケーブルを使用します。
* Nintendo Switchとの接続はUSBケーブルを使用します。
* ファームウェアのアップデート方法を検討します。
* 筐体の設計は別途行います。
