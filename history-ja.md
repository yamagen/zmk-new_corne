# 設定

- 左親指
   - 左: shift/option tap dance (左シフト/オプション)これはまだ成功していない。 
   - 中: number layer
   - 右: alt

- 右親指
   - 左: space
   - 中: symbol layer
   - 右: enter


        td1: td1 {
           compatible = "zmk,behavior-hold-tap";
           label = "LGUI when tap, Shift when hold";
           #binding-cells = <0>;
           tapping-term-ms = <200>;
           flavor = "hold-preferred";
           bindings = <&kp LGUI>, <&kp LEFT_SHIFT>;
        };
