# ロード/ストア命令 その2

ロード/ストア命令のうち、LDRH, LDRSB, LDRSH, STRHについて解説します

LDRとSTRは転送単位がWord(32bit)かByte(8bit)でしたが、こちらは

```
LDRH = LDR + Halfword
LDRSB = LDR + Signed Byte
LDRSH = LDR + Signed Halfword
STRH = STR + Halfword
```

のとおり転送単位や符号が異なります

## 📜 命令のフォーマット

### I=0

LDR, STRのときと違って、**I=0のときはオフセットにレジスタ**を用いることに注意

 bit  |  内容
---- | ----
31-28 | 条件
27-25 | 0b000
24 | P - Post/Pre (後述 0=post 1=pre)
23 | U - マイナス(-offset) か プラス(+offset) か (0=マイナス, 1=プラス)
22 | 0b0 (I - オフセットはレジスタ)
21 | [後述](#-bit21について)
20 | L - 読み出しか書き出しか (0=書き出し, 1=読み出し)
19-16 | Rn - ベースレジスタ (R0..R15) (このとき R15はPC+8)
15-12 | Rd - ターゲットレジスタ (R0..R15) (このとき R15はPC+12)
11-8 | 0b0000
7 | 0b1
6-5 | [オペコード](#-オペコード)
4 | 0b1
3-0 | Rm - オフセットを格納したレジスタ (R0..R14)

### I=1

LDR, STRのときと違って、**I=1のときはオフセットに即値**を用いることに注意

 bit  |  内容
---- | ----
31-28 | 条件
27-25 | 0b000
24 | P - Post/Pre (後述 0=post 1=pre)
23 | U - マイナス(-offset) か プラス(+offset) か (0=マイナス, 1=プラス)
22 | 0b1 (I - オフセットは即値)
21 | [後述](#-bit21について)
20 | L - 読み出しか書き出しか (0=書き出し, 1=読み出し)
19-16 | Rn - ベースレジスタ (R0..R15) (このとき R15はPC+8)
15-12 | Rd - ターゲットレジスタ (R0..R15) (このとき R15はPC+12)
11-8 | オフセット(上位4bit)
7 | 0b1
6-5 | [オペコード](#-オペコード)
4 | 0b1
3-0 | オフセット(下位4bit)

## 🔴 オペコード

L(bit20)は

- 0: メモリへのストア
- 1: メモリからのロード

 値 | L | フォーマット | 処理内容 | 備考
---- | ---- | ---- | ---- | ----
0 | 0 | -- | 未使用 | --
1 | 0 | STR{cond}H  Rd, ${Addr} | \[a\]=Rd | ハーフワードの格納
2 | 0 | LDR{cond}D  Rd, ${Addr} | R(d)=\[a\], R(d+1)=\[a+4\] | ダブルワードの読み込み
3 | 0 | STR{cond}D  Rd, ${Addr} | \[a\]=R(d), \[a+4\]=R(d+1) | ダブルワードの格納
0 | 1 | -- | 未使用 | --
1 | 1 | LDR{cond}H  Rd, ${Addr} | Rd=\[a\] | ハーフワードの読み込み
2 | 1 | LDR{cond}SB  Rd, ${Addr} | -- | 符号付きバイトの読み込み
3 | 1 | LDR{cond}SH  Rd, ${Addr} | Rd=\[a\] | 符号付きハーフワードの読み込み

32bitに満たないロードの場合、レジスタの上位部分は0埋めされます。

## 🔴 bit21について

bit21は P(bit24)の値によって意味が変わってきます。

### P=0(Post)のとき

bit21は使われず、必ず0となります。

### P=1(Pre)のとき

bit21はライトバックの有無を表すフラグ(W)となります。

- 0: ライトバックしない
- 1: Rnにアドレスをライトバックする

## 🔎 補足

フラグ: 不変

実行時間:

- LDR系: 1S+1N+1I
- LDR系でRdがPC: 2S+2N+1I
- STRH: 2N
