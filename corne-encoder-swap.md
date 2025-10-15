
# Corne Wireless ロータリーエンコーダ交換記録

## EC11 → EVQWGD001（静音・低トルクタイプ）

### 概要

Corne Wireless の標準 EC11 エンコーダを、Alps Alpine 製 EVQWGD001 に交換する。
目的は**低トルク化と静音化**による操作性の向上。
押し込みスイッチも PCB 側で生きているため、交換後もそのまま使用可能。

---

## 1. 使用部品・工具

### 部品

* **EVQWGD001** × 1（Alps Alpine、静音タイプ、押し込み付き）
* 必要に応じてエンコーダ用ノブ

### 工具

* 温調式はんだごて（350℃前後推奨）
* はんだ吸い取り線（wick）
* フラックス（吸い取り効率向上）
* ピンセット
* 精密ドライバー（ケース分解用）
* マルチメータ（導通確認用）

---

## 2. 事前準備

1. **ファームウェア設定をバックアップ**（ZMK）
2. **電池を外す／電源オフ**
3. 作業環境を静電気対策（リストストラップなど）

---

## 3. 分解手順

1. ケースのネジを外す（写真1: ケース全体俯瞰）
2. PCBをそっと持ち上げる（写真2: エンコーダ裏面の半田面アップ）
3. エンコーダの位置とピン配置を記録

   * 上3ピン＝回転（A/B/GND）
   * 下2ピン＝固定脚（兼GND）
   * 押し込み2ピンは内部に組み込み

---

## 4. 取り外し

1. フラックスを全ピンに塗布
2. **固定脚（下2ピン）から吸い取り**
3. 回転信号ピン（上3ピン）を順に吸い取る
4. 全ピンが動く状態になったら、エンコーダを真上に抜く（写真5）

---

## 5. 取り付け（EVQWGD001）

1. 新品EVQWGD001のピンを軽く整形
2. 基板に垂直に挿入（写真6）
3. 固定脚を軽く仮止め→垂直確認→回転3ピン→固定脚本締め
4. はんだは少なめに、ランドとピンがしっかり濡れ色になるまで

---

## 6. 動作確認

1. 基板をPC接続し、ZMKの`encoder-debug`や`zmk-console`で回転・押し込みの信号を確認
2. 回転方向が逆なら、`.overlay` に `direction = "inverted";` を追加するか、`encoder-bindings` のキー順を入れ替える

---

## 7. ZMK 設定例

### overlay（`overlay/corne_left.overlay`）

```dts
left_enc0: encoder_left_0 {
    compatible = "alps,ec11";
    a-gpios = <&gpio0 6 (GPIO_PULL_UP | GPIO_ACTIVE_LOW)>;
    b-gpios = <&gpio0 8 (GPIO_PULL_UP | GPIO_ACTIVE_LOW)>;
    steps = <2>; /* EVQWGD001は軽トルクなので2〜4で調整 */
    /* direction = "inverted"; */ /* 必要なら有効化 */
};
```

### keymap（`keymap/corne.keymap`）

```dts
encoder-bindings = <
    &inc_dec_kp KC_VOLU KC_VOLD /* 時計回り / 反時計回り */
>;
```

押し込み（行列配線されている場合）：

```dts
&kp KC_MUTE
```

---

## 10. Ctrl/Shift 最適化（オプション）

Corne をプログラマ向けに最適化する小改造です。Caps を排し、A 左に Ctrl、Z 左に Shift を配置。誤って Caps が入った場合のみ、Shift キーのダブルタップで解除できるようにします。

### 10.1 追加する behaviors（Shift/Caps のタップダンス）

```dts
/ {
    behaviors {
        td_shift_caps: td_shift_caps {
            compatible = "zmk,behavior-tap-dance";
            display-name = "Shift/CapsLock";
            #binding-cells = <0>;
            bindings = <&kp LSFT>, <&kp CAPS>;
            // 必要ならダブルタップ判定時間を調整:
            // tapping-term-ms = <200>;
        };
    };
};
```

### 10.2 default\_layer の差し替え（2カ所のみ）

現状（抜粋）:

```dts
&kp TAB    &kp Q  &kp W  &kp E     &kp R  &kp T                               &kp UP                &kp Y        &kp U  &kp I      &kp O    &kp P     &kp BSPC
&td0       &kp A  &kp S  &kp D     &kp F  &kp G                     &kp LEFT  &kp ENTER  &kp RIGHT  &kp H        &kp J  &kp K      &kp L    &kp SEMI  &kp SQT
&kp LCTRL  &kp Z  &kp X  &kp C     &kp V  &kp B        &kp SPACE              &kp DOWN              &kp N        &kp M  &kp COMMA  &kp DOT  &kp FSLH  &kp ESC
```

変更後（2行だけ置換）:

```dts
&kp TAB    &kp Q  &kp W  &kp E     &kp R  &kp T                               &kp UP                &kp Y        &kp U  &kp I      &kp O    &kp P     &kp BSPC
&kp LCTRL  &kp A  &kp S  &kp D     &kp F  &kp G                     &kp LEFT  &kp ENTER  &kp RIGHT  &kp H        &kp J  &kp K      &kp L    &kp SEMI  &kp SQT
&td_shift_caps  &kp Z  &kp X  &kp C     &kp V  &kp B        &kp SPACE          &kp DOWN              &kp N        &kp M  &kp COMMA  &kp DOT  &kp FSLH  &kp ESC
```

* A 左が Ctrl（\&kp LCTRL）
* Z 左が Shift（押しっぱなしで使用）、ダブルタップで Caps トグル
* 既存レイヤーやエンコーダ設定には非依存

### 10.3 調整のヒント

* ダブルタップが暴発する場合は `tapping-term-ms` を 180–250 に調整
* Caps を完全無効にしたい場合は、`bindings = <&kp LSFT>, <&kp NO>` にして二度押し動作を無効化

---

## 8. 完成・効果

* 回転操作が軽く静かになり、連続操作がスムーズに
* 押し込み動作も従来通り使用可能
* steps調整で意図しない回転入力も抑制

---

## 9. 撮影推奨カット

1. ケース全景（交換前）
2. 基板裏面・エンコーダ部クローズアップ
3. 吸い取り線で固定脚を除去中
4. 回転3ピン除去中
5. 取り外したEC11と基板
6. EVQWGD001を仮差し込みした状態
7. 交換後の基板全景
8. 組み立て後の完成写真
