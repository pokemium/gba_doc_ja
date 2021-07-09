# メモリアクセス時間

## システムクロック

```
  Bus clock  = 33MHz (33.513982 MHz)
  NDS7 clock = 33MHz (busと同じクロック)
  NDS9 clock = 66MHz (busの2倍のクロック)
```

このドキュメントに記載されているほとんどのタイミングは、33MHzのクロックで指定されています。（66MHzのクロックではありません）

また、NDS9のタイミングは「ハーフ」サイクルでカウントされます。

## メモリアクセスにかかる時間

以下の表は、ARM7/ARM9 CPUにおけるコードフェッチ・データフェッチのアクセス時間の違いを、シーケンシャル/ノンシーケンシャルの32bit/16bitアクセスで測定したものです。

すべてのタイミングは33MHz単位でカウントされます。つまり 1サイクル = 1/33M秒 です。

注：8bitデータのアクセスにかかる時間は、16bitデータと同じ時間です。

### コードフェッチ

**NDS7**

これらのメモリ領域に対してのアクセス時間は、可能性が高い程度のものです。つまり実際にはアクセス時間が異なる可能性もあります。

Region | N32 | S32 | N16 | S16 | Bus幅 
-- | -- | -- | -- | -- | -- 
Main RAM (read) (cache off) | 9 | 2 | 8 | 1 | 16 
WRAM,BIOS,I/O,OAM | 1 | 1 | 1 | 1 | 32 
VRAM,Palette RAM | 2 | 2 | 1 | 1 | 16 
GBA ROM (example 10,6 access) | 16 | 12 | 10 | 6 | 16 

**NDS9**

Region | N32 | S32 | N16 | S16 | Bus幅 
-- | -- | -- | -- | -- | -- 
Main RAM (read) (cache off) | 9 | 9 | 4.5 | 4.5 | 16 
WRAM,BIOS,I/O,OAM | 4 | 4 | 2 | 2 | 32 
VRAM,Palette RAM | 5 | 5 | 2.5 | 2.5 | 16 
GBA ROM (example 10,6 access) | 19 | 19 | 9.5 | 9.5 | 16 
TCM, Cache_Hit | 0.5 | 0.5 | 0.5 | 0.5 | 32 

これは最も厄介なタイミングです。

全てのノンシーケンシャルアクセス（キャッシュ、Tcm、Main RAMを除く）に3サイクルの悪名高いPENALTYが加算されます。

また、すべてのオペコードフェッチは強制的に32ビットのノンシーケンシャルアクセスにされます。(NDS9は単に高速なシーケンシャルオペコードフェッチをサポートしていません)

これはTHUMBコードにも当てはまります。つまり2つの16bitオペコードを1回のノンシーケンシャルな32bitアクセスでフェッチします。そのため、16bitオペコードあたりの時間は32bitフェッチの半分になります。

### データフェッチ

**NDS7**

NDS7のコードフェッチと全く同じです。ただし、メインRAMのノンシーケンシャルアクセスは1サイクル遅く、GBAスロットのノンシーケンシャルアクセスは1サイクル速いという不思議なことになっています。

Region | N32 | S32 | N16 | S16 | Bus幅 
-- | -- | -- | -- | -- | -- 
Main RAM (read) (cache off) | 10 | 2 | 9 | 1 | 16 
WRAM,BIOS,I/O,OAM | 1 | 1 | 1 | 1 | 32 
VRAM,Palette RAM | 1 | 2 | 1 | 1 | 16 
GBA ROM (example 10,6 access) | 15 | 12 | 9 | 6 | 16 
GBA ROM (example 10 access) | 9 | 10 | 9 | 10 | 8 

**NDS9**

シーケンシャルとノンシーケンシャル、16bitと32bitの両方のアクセスが可能なので、NDS9のコードフェッチよりも高速になっています。

ただし、ノンシーケンシャルなアクセスには3サイクルのペナルティがあります。また、NDS7のデータフェッチと同様に、ノンシーケンシャルなMain RAMアクセスには1サイクルだけ余計に時間がかかります。

Region | N32 | S32 | N16 | S16 | Bus幅 
-- | -- | -- | -- | -- | -- 
Main RAM (read) (cache off) | 10 | 2 | 9 | 1 | 16 
WRAM,BIOS,I/O,OAM | 4 | 1 | 4 | 1 | 32 
VRAM,Palette RAM | 5 | 2 | 4 | 1 | 16 
GBA ROM (example 10,6 access) | 19 | 12 | 13 | 6 | 16 
GBA ROM (example 10 access) | 13 | 10 | 13 | 10 | 8 
TCM, Cache_Hit | 0.5 | 0.5 | 0.5 | -- | 32 
Cache_Miss (BIOS) | 11 | 11 | 11 | -- | 32 
Cache_Miss (Main RAM) | 23 | 23 | 23 | -- | 16 

## Actual CPU Performance

33MHzで動作するNDS7は、多少なりともきれいに動いています。

しかし、66MHzで動作するはずのNDS9では、待ち時間が非常に多く、バスのクロックは実際のところ8～16MHz程度がやっとです。

唯一の例外は、キャッシュまたはtcm内のコードアクセスとデータアクセスで、最終的には本当の66MHzに到達しています。これは、キャッシュがヒットすることを前提としていますが、そうでなければ、キャッシュがミスした場合、キャッシュメモリのタイミングは1.4MHz程度まで下がるかもしれません。

```
ARM9 opcode fetches are always N32 + 3 waits.
- S16 and N16 do not exist (because thumb-double-fetching) (see there).
- S32 becomes N32 (ie. the ARM9 does NOT support fast sequential timing).

That N32 is having same timing as normal N32 access on NDS7, plus 3 waits.
- Eg. an ARM9 N32 or S32 to 16bit bus will take: N16 + S16 + 3 waits.
- Eg. an ARM9 N32 or S32 to 32bit bus will take: N32 + 3 waits.
```

メインメモリは、常に非連続なアクセスに対して3サイクルの待ち時間を必要とします。ARM7であっても同様です。

しかし、ARM9のデータフェッチは、シーケンシャルなアクセスでは高速にアクセスすることができ、生の16ビットアクセスも可能です。 これは、無理に低速の32ビットアクセスに拡張されることはありません。

ただし、ノンシーケンシャルなアクセスに対しては3回分の待ち時間ペナルティが追加されます。

```
Only exceptions are cache and tcm which do not have that penalty.
 Eg. LDRH on 16bit-data-bus is N16+3waits.
 Eg. LDR  on 16bit-data-bus is N16+S16+3waits.
 Eg. LDM  on 16bit-data-bus is N16+(n*2-1)*S16+3waits.

Eventually, data fetches can take place parallel with opcode fetches.
 That is NOT true for LDM (works only for LDR/LDRB/LDRH).
 That is NOT true for DATA in SAME memory region than CODE.
 That is NOT true for DATA in ITCM (no matter if CODE is in ITCM).
```

## NDS9 Busses

Unlike ARM7, the ARM9 has separate code and data busses, allowing it to perform code and data fetches simultaneously (provided that both are in different memory regions).
Normally, opcode execution times are calculated as "(codetime+datatime)", with the two busses, it can (ideally) be "MAX(codetime,datatime)", so the data access time may virtually take "NULL" clock cycles.
In practice, DTCM and Data Cache access can take NULL cycles (however, data access to ITCM can't).
When executing code in cache/itcm, data access to non-cache/tcm won't be any faster than with only one bus (as it's best, it could subtract 0.5 cycles from datatime, but, the access must be "aligned" to the bus-clock, so the "datatime-0.5" will be rounded back to the original "datatime").
When executing code in uncached main ram, and accessing data (elsewhere than in main memory, cache/tcm), then execution time is typically "codetime+datatime-2".

## NDS9 Internal Cycles

Additionally to codetime+datatime, some opcodes include one or more internal cycles. Compared with ARM7, the behaviour of that internal cycles is slightly different on ARM9. First of, on the NDS9, the internal cycles are of course "half" cycles (ie. counted in 66MHz units, not in 33MHz units) (although they may get rounded to "full" cycles upon next memory access outside tcm/cache). And, the ARM9 is in some cases "skipping" the internal cycles, that often depending on whether or not the next opcode is using the result of the current opcode.
Another big difference is that the ARM9 has lost the fast-multiply feature for small numbers; in some cases that may result in faster execution, but may also result in slower execution (one workaround would be to manually replace MUL opcodes by the new ARM9 halfword multiply opcodes); the slowest case are MUL opcodes that do update flags (eg. MULS, MLAS, SMULLS, etc. in ARM mode, and all ALL multiply opcodes in THUMB mode).

## NDS9 Thumb Code

In thumb mode, the NDS9 is fetching two 16bit opcodes by a single 32bit read. In case of 32bit bus, this reduces the amount of memory traffic and may result in faster execution time, of course that works only if the two opcodes are within a word-aligned region (eg. loops at word-aligned addresses will be faster than non-aligned loops). However, the double-opcode-fetching is also done on 16bit bus memory, including for unnecessary fetches, such like opcodes after branch commands, so the feature may cause heavy slowdowns.

## Main Memory

Reportedly, the main memory access times would be 5 cycles (nonsequential read), 4 cycles (nonsequential write), and 1 cycle (sequential read or write). Plus whatever termination cycles. Plus 3 cycles on nonsequential access to the last 2-bytes of a 32-byte block.
That's of course all wrong. Reads are much slower than 5 cycles. Not yet tested if writes are faster. And, I haven't been able to reproduce the 3 cycles on last 2-bytes effect, actually, it looks more as if that 3 cycles are accidently added to ALL nonsequential accesses, at ALL main memory addresses, and even to most OTHER memory regions... which might be the source of the PENALTY which occurs on VRAM/WRAM/OAM/Palette and I/O accesses.

## DMA

In some cases DMA main memory read cycles are reportedly performed simultaneously with DMA write cycles to other memory.

## NDS9

On the NDS9, all external memory access (and I/O) is delayed to bus clock (or actually MUCH slower due to the massive waitstates), so the full 66MHz can be used only internally in the NDS9 CPU core, ie. with cache and TCM.

## Bus Clock

The exact bus clock is specified as 33.513982 MHz (1FF61FEh Hertz). However, on my own NDS, measured in relation to the RTC seconds IRQ, it appears more like 1FF6231h, that inaccuary of 1 cycle per 657138 cycles (about one second per week) on either oscillator, isn't too significant though.

## GBA Slot

The access time for GBA slot can be configured via EXMEMCNT register.

## VRAM Waitstates

Additionally, on NDS9, a one cycle wait can be added to VRAM accesses (when the video controller simultaneously accesses it) (that can be disabled by Forced Blank, see DISPCNT.Bit7). Moreover, additional VRAM waitstates occur when using the video capture function.
Note: VRAM being mapped to NDS7 is always free of additional waits.
