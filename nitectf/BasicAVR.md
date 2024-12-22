I'd like to credit my teammate _invokedynamic_ for the massive help in patching the elf file.

Find out which board is being used:
```bash
(base) achu@air Downloads % strings BasicAVR.elf | grep atmega
atmega2560
(base) achu@air Downloads % 
```
We'll use [simavr](https://github.com/buserror/simavr) to simulate `atmega2560`.
```bash
(base) achu@air Downloads % avr-objcopy -O ihex BasicAVR.elf BasicAVR.hex
(base) achu@air Downloads % simavr -m atmega2560 -f 16000000 BasicAVR.hex
Loaded 1 section(s) of ihex
Load HEX flash 00000000, 3628 at 00000000
nite2024 Initialized. Patch ELF and Configure port pins correctly. iykyk :))..
Unpatched ELF :< NO Run..
Unpatched ELF :< NO Run..
Unpatched ELF :< NO Run..
Unpatched ELF :< NO Run..
Unpatched ELF :< NO Run..
Unpatched ELF :< NO Run..
Unpatched ELF :< NO Run..
Unpatched ELF :< NO Run..
Unpatched ELF :< NO Run..
Unpatched ELF :< NO Run..
(last line continues to repeat ...)
```
This clearly states that we need to patch elf and configure ports in it to get the flag.

![Patching video](../assets/contents/BasicAVR/1.mp4)

After patching, attempting again
```bash
(base) achu@air Downloads % avr-objcopy -O ihex BasicAVR.elf BasicAVR.hex 
(base) achu@air simavr -m atmega2560 -f 16000000 BasicAVR.hex
Loaded 1 section of ihex
Load HEX flash 00000000, 3628
nite2024 Initialized. Patch ELF and Configure port pins correctly. iykyk :))
FLAG: nite{h4RDW4Re_rEv_I5_gOA7Ed}..
FLAG: nite{h4RDW4Re_rEv_I5_gOA7Ed}..
FLAG: nite{h4RDW4Re_rEv_I5_gOA7Ed}..
(last line continues to repeat ...)
```
We get our flag `nite{h4RDW4Re_rEv_I5_gOA7Ed}`
