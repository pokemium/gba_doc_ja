# CP15

CP15は、メモリ管理用のコプロセッサです。 キャッシュのON/OFFなどをコントロールします。 

CP15自体に制御用レジスタを持っていて、CPUからこれに値を書き込むことでCP15の動作を制御します。

メモリ制御機能を持たないCPUでは、CP15が搭載されておらず、その場合、Main IDレジスタを読もうとしても、未定義命令の例外が発生します。

## CP15 Opcodes

CP15にはMCRとMRCオペコードを用いてアクセス可能です。その際に、`Pn=P15, <cpopc>=0`としてやる必要があります。

```
  MCR{cond} P15,<cpopc>,Rd,Cn,Cm,<cp>   ; ARMレジスタからCP15にデータを転送
  MRC{cond} P15,<cpopc>,Rd,Cn,Cm,<cp>   ; CP15からARMレジスタにデータを転送
```

RdはARMレジスタを指しており、R0-R14から選択可能です。R15はP15に対するMCR/MRCでは利用すべきではありません。

\<cpopc\>,Cn,Cm,\<cp\>はCP15のレジスタを指定するのに利用します。

例えば、`<cpopc>,Cn,Cm,<cp> = [0, C0, C0, 0]`とするとMain IDレジスタを指定します。

コプロセッサ用の他のオペコード(CDP, LDC, STC)はP15に対しては利用できません。

## CP15 Register List

レジスタ | 説明 
-- | -- 
0,C0,C0,0 | Main ID Register (R)
0,C0,C0,1 | Cache Type and Size (R)
0,C0,C0,2 | TCM Physical Size (R)
0,C0,C0,3 | ARM11: TLB Type Register
0,C0,C1,0 | ARM11: Processor Feature Register 0
0,C0,C1,1 | ARM11: Processor Feature Register 1
0,C0,C1,2 | ARM11: Debug Feature Register 0
0,C0,C1,3 | ARM11: Auxiliary Feature Register 0
0,C0,C1,4 | ARM11: Memory Model Feature Register 0
0,C0,C1,5 | ARM11: Memory Model Feature Register 1
0,C0,C1,6 | ARM11: Memory Model Feature Register 2
0,C0,C1,7 | ARM11: Memory Model Feature Register 3
0,C0,C2,0 | ARM11: Set Attributes Register 0
0,C0,C2,1 | ARM11: Set Attributes Register 1
0,C0,C2,2 | ARM11: Set Attributes Register 2
0,C0,C2,3 | ARM11: Set Attributes Register 3
0,C0,C2,4 | ARM11: Set Attributes Register 4
0,C0,C2,5 | ARM11: Set Attributes Register 5
-- | -- 
0,C1,C0,0     | Control Register (R/W, or R=Fixed)
0,C1,C0,1     | ARM11: Auxiliary Control Register
0,C1,C0,2     | ARM11: Coprocessor Access Control Register
0,C2,C0,0     | ARM11: Translation Table Base Register 0
0,C2,C0,1     | ARM11: Translation Table Base Register 1
0,C2,C0,2     | ARM11: Translation Table Base Control Register
0,C3,C0,0     | ARM11: Domain Access Control Register
0,C5,C0,0     | ARM11: Data Fault Status Register
0,C5,C0,1     | ARM11: Instruction Fault Status Register
0,C6,C0,0     | ARM11: Fault Address Register (FAR)
0,C6,C0,1     | ARM11: Watchpoint Fault Address Register (WFAR)
0,C2,C0,0     | PU Cachability Bits for Data/Unified Protection Region
0,C2,C0,1     | PU Cachability Bits for Instruction Protection Region
0,C3,C0,0     | PU Cache Write-Bufferability Bits for Data Protection Regions
0,C5,C0,0     | PU Access Permission Data/Unified Protection Region
0,C5,C0,1     | PU Access Permission Instruction Protection Region
0,C5,C0,2     | PU Extended Access Permission Data/Unified Protection Region
0,C5,C0,3     | PU Extended Access Permission Instruction Protection Region
0,C6,C0..C7,0 | PU Protection Unit Data/Unified Region 0..7
0,C6,C0..C7,1 | PU Protection Unit Instruction Region 0..7
0,C7,Cm,Op2   | Cache Commands and Halt Function (W)
0,C9,C0,0     | Cache Data Lockdown
0,C9,C0,1     | Cache Instruction Lockdown
0,C9,C1,0     | TCM Data TCM Base and Virtual Size
0,C9,C1,1     | TCM Instruction TCM Base and Virtual Size
0,C13,Cm,Op2  | Misc Process ID registers
0,C15,Cm,Op2  | Misc Implementation Defined and Test/Debug registers

## Data/Unified Registers

Some Cache/PU/TCM registers are declared as "Data/Unified".

That registers are used for Data accesses in case that the CPU contains separate Data and Instruction registers, otherwise the registers are used for both (unified) Data and Instruction accesses.
