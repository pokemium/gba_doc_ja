# BIOS

BIOS には、SWI命令を通して利用可能ないくつかのシステムコール機能が含まれています。

システムコールの引数は通常、レジスタR0,R1,R2,R3に渡されます。

システムコール終了後、R0,R1,R3には、ゴミか戻り値のいずれかが含まれます。他のレジスタ(R2,R4-R14)は全て変更されません。

## 注意

SWIをARMステートから呼び出す場合、THUMBステートでは`swi nn`でしたが、ARMステートでは`swi nn*0x10000`とする必要があります。

## BIOS関数の一覧

### 標準機能

nn | 関数
-- | -- 
0x00 | SoftReset
0x01 | RegisterRamReset
0x02 | Halt
0x03 | Stop/Sleep
0x04 | IntrWait
0x05 | VBlankIntrWait
0x06 | Div
0x07 | DivArm
0x08 | Sqrt
0x09 | ArcTan
0x0a | ArcTan2
0x0b | CpuSet
0x0c | CpuFastSet
0x0d | GetBiosChecksum
0x0e | BgAffineSet
0x0f | ObjAffineSet

### 解凍

nn | 関数
-- | -- 
0x10 | BitUnPack
0x11 | LZ77UnCompReadNormalWrite8bit
0x12 | LZ77UnCompReadNormalWrite16bit
0x13 | HuffUnCompReadNormal
0x14 | RLUnCompReadNormalWrite8bit
0x15 | RLUnCompReadNormalWrite16bit
0x16 | Diff8bitUnFilterWrite8bit
0x17 | Diff8bitUnFilterWrite16bit
0x18 | Diff16bitUnFilter

### サウンド・その他

nn | 関数
-- | -- 
0x19 | SoundBias
0x1a | SoundDriverInit
0x1b | SoundDriverMode
0x1c | SoundDriverMain
0x1d | SoundDriverVSync
0x1e | SoundChannelClear
0x1f | MidiKey2Freq
0x20 | SoundWhatever0
0x21 | SoundWhatever1
0x22 | SoundWhatever2
0x23 | SoundWhatever3
0x24 | SoundWhatever4
0x25 | MultiBoot
0x26 | HardReset
0x27 | CustomHalt
0x28 | SoundDriverVSyncOff
0x29 | SoundDriverVSyncOn
0x2a | SoundGetJumpList