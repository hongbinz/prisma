### Prisma Trace Guide

Table of Contents
=================

* [Prisma Trace Guide](#prisma-trace-guide)
  * [查看TA生效](#%E6%9F%A5%E7%9C%8Bta%E7%94%9F%E6%95%88)
  * [查看pusch功率](#%E6%9F%A5%E7%9C%8Bpusch%E5%8A%9F%E7%8E%87)
  * [怎么计算offset 高频](#%E6%80%8E%E4%B9%88%E8%AE%A1%E7%AE%97offset-%E9%AB%98%E9%A2%91)
  * [查看PUCCH的打印](#%E6%9F%A5%E7%9C%8Bpucch%E7%9A%84%E6%89%93%E5%8D%B0)
  * [查看PDCCH打印](#%E6%9F%A5%E7%9C%8Bpdcch%E6%89%93%E5%8D%B0)
  * [查看Late decoded data](#%E6%9F%A5%E7%9C%8Blate-decoded-data)
  * [怎么重启ftp](#%E6%80%8E%E4%B9%88%E9%87%8D%E5%90%AFftp)
  * [怎么抓symshark dump](#%E6%80%8E%E4%B9%88%E6%8A%93symshark-dump)
  * [怎么重启LSU](#%E6%80%8E%E4%B9%88%E9%87%8D%E5%90%AFlsu)
  * [查看MSG1功率](#%E6%9F%A5%E7%9C%8Bmsg1%E5%8A%9F%E7%8E%87)
  * [查看RACH过程](#%E6%9F%A5%E7%9C%8Brach%E8%BF%87%E7%A8%8B)


#### 查看TA生效

```
trcinfo -p 0-1 |grep apCtlTaSet
```

#### 查看pusch功率

```
10879776783016|bbPowerPuschEval: rntiType=6 minPw=14.000000 maxPw=31.855480 scale=31.855481 cfgPwBase=-39.444519 cfgPwDeltaRB=24.080000 cfgPwTot=-15.364519 antScale=0.000000 pl=74.555481 alpha=1.000000 deltaMcs=0 
cfgPwTot: real power tx,dBm
```

#### 怎么计算offset 高频

```
 SSB_SC_OFFSET_00 = FFTsize/2 - (C.F. - SSBfreq)/SCspacing - 120 - GUARDBAND
   其中： fftsixe and number of rb is 32 是 512，Guardband is 64
   也就等于： SSB_SC_OFFSET_00 = 72 - (C.F. -SSBFreq)/240
对于 30Khz fftsize: 4096
guardband = (fftsize - numRB*12)/2
at 100Mhz num of Rb is 273
(4096-273*12)/2 = 410
fftSize = sampleRate/num
samplerate = 122.88e6
122.88e6/240e3 = 512
```

#### 查看PUCCH的打印

```
看下面三个关键字的打印：
bbAddUciItem is the first, and is print when HARQ feedback is communicated to UL core by DL core after PDSCH decoding
then, bbPucchProc and _fillPucchCfg are printed when PUCCH is being sent
```

#### 查看PDCCH打印

```
PDCCH DCI 0_1: txSfn=914 txSlot=17 rnti=8192 sliv=55 mcsTab=0 tbLen=53288 nAddDmrs=0 nLayers=2 nRBUl=273 nBlk=7 RAtype=1 (1) nRB=273 startRE=0 antPorts=1
PDCCH FOUND: slotNum 16 rnti 8192 lenDci 73. bwpT=2 aggLev=4 common=0 nCand=1 decoded=0000811081718111 (DCI0_1: time 0 mcs 5 res 545)
```

#### 查看Late decoded data

```
Late decoded data (16/[0 9] sym 13) blkE 108, tbLen 1032192
--------------------------------------------------------------------------
的意思是“ slotnumber 16 has been decoded during subframe 9 / slot 0 (that means slotnumber 18)，this is a warning because latency is >= 2 slots
the issue would be to have a latency > 3 slots, because we don't handle more than 3 slots  in memory

```

#### 怎么重启ftp

```
pidin | grep inetd

the leftmost field is the pid

lsusuid kill -SIGHUP <pid>
```

#### 怎么抓symshark dump

```
on -f ppu0-1 lsusuid <symshark路径>/symsk2Dl20 symskdrv-ap-01-00 
其中01： "CellId", cell ID Prisma side
里面有9个文件分别是 
symsk2Dl20
symsk2Dl200
symsk2Fft30_100
symsk2Fft120_100
symsk2Ul20
symsk2Ul200
symsk2Ul200w (可以开跑测试之前运行，通过MSG1触发开始抓200ms的包)
symsk4Dl20
symsk4Ul20
其中含义： 比如symsk2Dl20
2 (first one in file name)：	it will dump 2 antenna ports
Dl：	direction, downlink in this case
20：	dump duration will last 20 milliseconds

```

#### 怎么重启LSU

```
通过命令: shutdown -f
或者通过IPMI/通过LSU的电源按钮下电
```

#### 查看MSG1功率

```
trcinfo -p 0-2 |grep bbMsgRaReq 
0-2: was bb ppu
bbMsgRaReq: frame=1021 slot=15 ueId=-1 preambleIdx=11 beam=0 power=-113 distance=-1
bbMsgRaReq: frame=1023 slot=9 ueId=-1 preambleIdx=11 beam=0 power=-112 distance=-1
bbMsgRaReq: frame=982 slot=14 ueId=1 preambleIdx=21 beam=0 power=-112 distance=-1
```

#### 查看RACH过程

在rlc/mac ppu , **trcinfo -p <rlc/mac ppu> |grep -i rar**

```bash
Dec 20 12:55:37:822171 80500031   19271701 rlcm        -     05   7021 RAR Recv CellId=0 DbeamId=0
Dec 20 12:55:37:822173 80500031   19271701 rlcm        -     05   7069 mac_RAR_Recv e=0 t=1 rapid=56
Dec 20 12:55:37:822177 80500031   19271701 rlcm        -     1B    294 ulRarGrantDecod: bwpidx=0 Hop=0 fdra_hi=34 Fdra=545 Tdra=6 Mcs=0 Tpc=3 CsiReq=0 ~~ config: nFrontDMRS=1 nAddDMRS=2 xOverhead=0 locationAndBandwidth=1099 mcsTable=0 nLayers=1 ~~ TDAlloc: mu=1 K2=7 deltaForK2=0 StartSymbAndLen=41 rar_tti->Slot=130.17 ex_rar_slot=2617 ex_slot=2624 tx_slot=131.4 nSymb=13 ~~ FDAlloc: maxRB=273 nRB=273 ~~ tbsize_calc: TbLen=960 SegNum=3 SegLenBit=2592
Dec 20 12:55:37:822179 80500031   19271701 rlcm        -     05   7182 Valid RAR Recv cellId=1 macRaRnti=269 rarRaRnti=269 ueId=1 pid=56 ta_hi=0 ta=4 t_rnti_hi=21 t_rnti=5612
Dec 20 12:55:37:822181 80500031   19271701 rlcm        -     05   4638 RA_SuccessCompletion UeId=1 CONTENTION=0
Dec 20 12:55:37:822183 80500031   19271701 rlcm        -     05   7206 RAR Recv SUCC RA Contention-free CellId=1 UeId=1 rnti=5612 pid=56
Dec 20 12:55:37:822185 80500031   19271701 rlcm        -     05   7295 RAR GRANT TX (IMM) UeId=1
Dec 20 12:55:37:822187 80500031   19271701 rlcm        -     05   6379 InsertHarqUeList UeId=1 on CellId=1 flag=1
Dec 20 12:55:37:822188 80500031   19271701 rlcm        -     05   6665 MAC RAR GRANT TX UeId=1
Dec 20 12:55:37:822190 80500031   19271701 rlcm        -     0F    453 _HARQ_: RarGrant(): cell=1 UeId=1 rnti=5612 hentId=0 hproId=0
Dec 20 12:55:37:822191 80500031   19271701 rlcm        -     0F    151 harq_ProSelect: RAR cell=1 UeId=1 hproId=0
Dec 20 12:55:37:822193 80500031   19271701 rlcm        -     0F    612 MAC SCHEDULE MSG3/DATA on RAR grant UeId=1

```

