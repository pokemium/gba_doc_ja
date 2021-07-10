# Cache と TCM

キャッシュとTCMはSystem Control Coprocessor(ARM CP15 System Control Coprocessor)によって制御されています。

これらはNDS9のためのものです。NDS7はTCMやCache、CP15を持ちません。

## TCM

TCM = Tightly Coupled Memory

メインメモリへのアクセスはバス速度により制限され、CPUのクロックより遅くなります。 

TCMはARM9のコアに直接入ってるメモリ(らしい)で、CPUクロックと同じ速度で動作します。

キャッシュと違いプログラマが内容をコントロールできるらしいです。命令コード用TCM(ITCM)とデータ用TCM(DTCM)があります。

TCM内に完結した状態でCPUが動作している間はバスが空くので、DMAコントローラを同時に動作させることができます。

```
  ITCM 32K, base=00000000h (固定)
  DTCM 16K, base=moveable  (デフォルトでは base=27C0000h)
```

ITCMは移動できませんが、NDSのファームウェアではITCMのサイズを32MBに設定しているため、`0x0-0x1FF_FFFF`のITCMミラーが生成されます。

さらに、PU(後述)を使ってその領域のメモリをロック/アンロックすることができます。このトリックにより、下位32MBのメモリ内のどこにでもITCMを移動させることができます。

## キャッシュ

- Data Cache 4KB, Instruction Cache 8KB
- 4-way set associative method
- Cache line 8 words (32 bytes)
- Read-allocate method (ie. writes are not allocating cache lines)
- Round-robin and Pseudo-random replacement algorithms selectable
- Cache Lockdown, Instruction Prefetch, Data Preload
- Data write-through and write-back modes selectable

## Protection Unit(PU)

デフォルトでは次のように設定されています。

Region Name | Address | Size | Cache | WBuf | Code | Data 
-- | -- | -- | -- | -- | -- | -- 
Background     | 0x0000_0000 | 4GB   | -  | -  | -   | -
I/O and VRAM   | 0x0400_0000 | 64MB  | -  | -  | R/W | R/W
Main Memory    | 0x0200_0000 | 4MB   | On | On | R/W | R/W
ARM7-dedicated | 0x027C_0000 | 256KB | -  | -  | -   | -
GBA Slot       | 0x0800_0000 | 128MB | -  | -  | -   | R/W
DTCM           | 0x027C_0000 | 16KB  | -  | -  | -   | R/W
ITCM           | 0x0100_0000 | 32KB  | -  | -  | R/W | R/W
BIOS           | 0xFFFF_0000 | 32KB  | On | -  | R   | R
Shared Work    | 0x027F_F000 | 4KB   | -  | -  | -   | R/W

Notes: In Nintendo's hardware-debugger, Main Memory is expanded to 8MB (for that reason, some addresses are at 27NN000h instead 23NN000h) (some of the extra memory is reserved for the debugger, some can be used for game development). 

Region 2 and 7 are not understood? GBA Slot should be max 32MB+64KB, rounded up to 64MB, no idea why it is 128MB? DTCM and ITCM do not use Cache and Write-Buffer because TCM is fast. 

Above settings do not allow to access Shared Memory at 37F8000h? Do not use cache/wbuf for I/O, doing so might suppress writes, and/or might read outdated values.

The main purpose of the Protection Unit is debugging, a major problem with GBA programs have been faulty accesses to memory address 00000000h and up (due to \[base+offset\] addressing with uninitialized (zero) base values). 

This problem has been fixed in the NDS, for the ARM9 processor at least, still there are various leaks: For example, the 64MB I/O and VRAM area contains only ca. 660KB valid addresses, and the ARM7 probably doesn't have a Protection Unit at all. Alltogether, the protection is better than in GBA, but it's still pretty crude compared with software debugging tools.

Region address/size are unified (same for code and data), however, cachabilty and access rights are non-unified (and may be separately defined for code and data).
