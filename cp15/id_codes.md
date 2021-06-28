# ID Codesレジスタ

## C0,C0,0 - Main IDレジスタ (R)

Main IDレジスタはARMプロセッサ(≠コプロセッサ)の情報を提供するレジスタです。

 bit  |  内容
---- | ---- 
12-15 | ARMプロセッサの世代 (0=Pre-ARM7, 7=ARM7, other=Post-ARM7)

他のbitの内容はARMの世代によって変わってきます。

### Post-ARM7

 bit  |  内容
---- | ---- 
0-3   | Revision Number
4-15  | Primary Part Number (Bit12-15 must be other than 0 or 7) (eg. 946h for ARM946)
16-19 | Architecture (1=v4, 2=v4T, 3=v5, 4=v5T, 5=v5TE, 6=v6, ?=v7)
20-23 | Variant Number
24-31 | Implementor (41h=ARM, 44h=Digital Equipment Corp, 69h=Intel)

### ARM7

 bit  |  内容
---- | ---- 
0-3   | Revision Number
4-15  | Primary Part Number (Bit12-15 must be 7)
16-22 | Variant Number
23    | Architecture        (0=v3, 1=v4T)
24-31 | Implementor         (41h=ARM, 44h=Digital Equipment Corp, 69h=Intel)

### Pre-ARM7

 bit  |  内容
---- | ---- 
0-3   | Revision Number
4-11  | Processor ID LSBs (30h=ARM3/v2, 60h,61h,62=ARM600,610,620/v3)
12-31 | Processor ID MSBs (fixed, 41560h)

Note: On the NDS9, this register is 41059461h (ARMv5TE, ARM946, rev1). NDS7 and GBA don't have CP15s.

## C0,C0,1 - Cache Type Register (R)

TODO

## C0,C0,2 - Tightly Coupled Memory (TCM) Size Register (R)

TODO

## C0,C0,3..7 - Reserved (R)

使われないレジスタです。内容は`C0,C0,0`のミラーです。
