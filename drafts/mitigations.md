Off:

```
[quartz@hydrogen4 ~]$ 7z b

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_GB.UTF-8,Utf16=on,HugeFiles=on,64 bits,8 CPUs Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz (806EA),ASM,AES-NI)

Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz (806EA)
CPU Freq: - - - - - - - - -

RAM size:    7612 MB,  # CPU hardware threads:   8
RAM usage:   1765 MB,  # Benchmark threads:      8

                       Compressing  |                  Decompressing
Dict     Speed Usage    R/U Rating  |      Speed Usage    R/U Rating
         KiB/s     %   MIPS   MIPS  |      KiB/s     %   MIPS   MIPS

22:      15959   652   2381  15526  |     213934   774   2357  18248
23:      14439   664   2215  14712  |     202683   777   2258  17540
24:      13841   676   2203  14883  |     195642   779   2205  17171
25:      13264   690   2195  15145  |     189803   772   2189  16892
----------------------------------  | ------------------------------
Avr:             670   2249  15066  |              775   2252  17463
Tot:             723   2251  16264
[quartz@hydrogen4 ~]$ grep -H '' /sys/devices/system/cpu/vulnerabilities/*
/sys/devices/system/cpu/vulnerabilities/itlb_multihit:KVM: Mitigation: VMX disabled
/sys/devices/system/cpu/vulnerabilities/l1tf:Mitigation: PTE Inversion; VMX: vulnerable
/sys/devices/system/cpu/vulnerabilities/mds:Vulnerable; SMT vulnerable
/sys/devices/system/cpu/vulnerabilities/meltdown:Vulnerable
/sys/devices/system/cpu/vulnerabilities/mmio_stale_data:Vulnerable
/sys/devices/system/cpu/vulnerabilities/spec_store_bypass:Vulnerable
/sys/devices/system/cpu/vulnerabilities/spectre_v1:Vulnerable: __user pointer sanitization and usercopy barriers only; no swapgs barriers
/sys/devices/system/cpu/vulnerabilities/spectre_v2:Vulnerable, IBPB: disabled, STIBP: disabled
/sys/devices/system/cpu/vulnerabilities/srbds:Vulnerable
/sys/devices/system/cpu/vulnerabilities/tsx_async_abort:Not affected
```

On:

```
[quartz@hydrogen4 ~]$ grep -H '' /sys/devices/system/cpu/vulnerabilities/*
/sys/devices/system/cpu/vulnerabilities/itlb_multihit:KVM: Mitigation: VMX disabled
/sys/devices/system/cpu/vulnerabilities/l1tf:Mitigation: PTE Inversion; VMX: conditional cache flushes, SMT vulnerable
/sys/devices/system/cpu/vulnerabilities/mds:Mitigation: Clear CPU buffers; SMT vulnerable
/sys/devices/system/cpu/vulnerabilities/meltdown:Mitigation: PTI
/sys/devices/system/cpu/vulnerabilities/mmio_stale_data:Mitigation: Clear CPU buffers; SMT vulnerable
/sys/devices/system/cpu/vulnerabilities/spec_store_bypass:Mitigation: Speculative Store Bypass disabled via prctl
/sys/devices/system/cpu/vulnerabilities/spectre_v1:Mitigation: usercopy/swapgs barriers and __user pointer sanitization
/sys/devices/system/cpu/vulnerabilities/spectre_v2:Mitigation: Retpolines, IBPB: conditional, IBRS_FW, STIBP: conditional, RSB filling
/sys/devices/system/cpu/vulnerabilities/srbds:Mitigation: Microcode
/sys/devices/system/cpu/vulnerabilities/tsx_async_abort:Not affected
[quartz@hydrogen4 ~]$ 7z b

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_GB.UTF-8,Utf16=on,HugeFiles=on,64 bits,8 CPUs Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz (806EA),ASM,AES-NI)

Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz (806EA)
CPU Freq: - - - - 128000000 256000000 - - -

RAM size:    7612 MB,  # CPU hardware threads:   8
RAM usage:   1765 MB,  # Benchmark threads:      8

                       Compressing  |                  Decompressing
Dict     Speed Usage    R/U Rating  |      Speed Usage    R/U Rating
         KiB/s     %   MIPS   MIPS  |      KiB/s     %   MIPS   MIPS

22:      15485   661   2279  15064  |     213430   770   2364  18205
23:      14042   640   2235  14308  |     204061   776   2276  17659
24:      13921   682   2196  14969  |     196270   776   2220  17226
25:      13410   696   2201  15311  |     191557   774   2202  17048
----------------------------------  | ------------------------------
Avr:             670   2228  14913  |              774   2265  17534
Tot:             722   2247  16224

```

