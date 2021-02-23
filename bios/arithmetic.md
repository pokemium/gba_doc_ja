# 算術処理

Div, DivArm, Sqrt, ArcTan, ArcTan2

## Div: swi 0x06

```
(r0, r1, r3) = r0/r1
```

引数:

- r0: 分子
- r1: 分母

戻り値:

- r0: 商
- r1: 余り
- r3: 商の絶対値

例:

(r0, r1) = (-1234, 10) のとき

(r0, r1, r3) = (-123, -4, +123)

0で割った時は無限ループに陥ります。

## DivArm: swi 0x07

```
(r0, r1, r3) = r1/r0
```

引数の分母分子が逆以外はDivと同じです。 

ARMライブラリとの互換を保つための関数でDivより3クロックだけ遅いです。

## Sqrt: swi 0x08

```
r0: uint16 = sqrt(r0: uint32)
```

返り値は整数になります。例えばsqrt(2)は1になります。

## ArcTan: swi 0x09

TODO

## ArcTan2: swi 0x0a

TODO
