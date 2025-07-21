---
title: Unpacking Malware (PMA Lab 18) 
date: 2025-05-31
categories: [malware analysis, writeups]
tags: [malware analysis, windows, reverse engineering]      # TAG names should always be lowercase
---




# Packing and Unpacking:

Malware is often obfuscated and or "packed" to make it harder to reverse and examine. Packing can modify various data sections. When dealing with packed malware you will often need to reconstruct the import section.


In genreal we will want to find the "unpacking stub" which will lead to the `OEP` Original Entry Point of the executable. From there we can reconstruct the program. The unpacked program will not be identical to the original as the PE header is reconstructed.


For more info on packing see this eariler post: <https://saadams.github.io/posts/mal-antirev/>




## Steps for Unpacking:

#### Automatic:

    Use a tool like PE explorer, PEID


#### Manual:

* Use a debugger to find the OEP then dump the program.
    * OllyDbg has a built in OEP finder in OllyDump
    * Otherwise step through the program to find OEP.
    * Search for the `tail jump` which is a jump that typically goes far away.


#### Repairing Imports Manually:

* the imports table is 2 tables in memory:

   * list of names or ordinals used by the loader or unpacking stub to determine which functions are needed.
   * list of addresses of all functions imported.
* Repair each import function as encountered...
    * find unknown import func in IDA look it up in the debugger and then label it accordingly in IDA.


-----

## Sample 1


DIE (UPX detected)

![alt text](image-19.png)

Here we can see that the file is detected to have used UPX to be packed.

![alt text](image-20.png)

We can also observe that the entropy is very high which is a good indicator that it is packed.



Due to the fact that this file is using a modified version of `UPX` we will not be able to simply unpack it with `UPX.`

![alt text](image-21.png)



#### Manual Unpacking:

![DIE,unpack](../assets/imgs/antidebug/man_unpack1.png)
* When the file is opened in IDA it only contains 1 function `start` and a very small import table.

![alt text](../assets/imgs/antidebug/man_unpack2.png)

* Many unpackers use `jmp` as a means to transfer control to the unpacked code. A search for `jmp` instructions leaves a few results.

![alt text](../assets/imgs/antidebug/man_unpack3.png)


![alt text](../assets/imgs/antidebug/man_unpack4.png)

* The one that stood out was a jump at the address `0x409F43` as it jumps far away and to invalid instructions.

![sus jump](../assets/imgs/antidebug/sus_jmp.png)


* Setting a breakpoint on the address in x32 debugger

![alt text](../assets/imgs/antidebug/man_unpack6.png)
![alt text](../assets/imgs/antidebug/man_unpack7.png)

* When the program is run again `eip` is stopped at the breakpoint.

![alt text](../assets/imgs/antidebug/man_unpack8.png)

* Single step/step into and the eip moves to the unpacked code. From here the process memory can be dumped with `scylla`.

![alt text](../assets/imgs/antidebug/man_unpack9.png)

* Open scylla and auto search.

![alt text](../assets/imgs/antidebug/man_unpack10.png)

* Get imports and then dump

![alt text](../assets/imgs/antidebug/man_unpack12.png)

* After the program is dumpped, fix the dump which will make the program analyzable in IDA.

 ![alt text](../assets/imgs/antidebug/man_unpack13.png)

* Re-opening the sample in IDA all the imports and functions now appear and the sample is ready for analysis.

 ![alt text](../assets/imgs/antidebug/man_unpack14.png)


------

## Sample 2


DIE results:

![alt text](image-24.png)

Here we can see that DIE believes the file is packed due to repeating section names.

Lets check out the binary sections to see what is going on.


![alt text](image-28.png)

We can also observe that the entropy is very high which is a good indicator that it is packed.



We can also see that the `DOS stub` has been modifyed.

![alt text](image-26.png)

This differs from what would we expect to see `“This program cannot be run in DOS mode.”`


 Within `PEStudio` we can see that the packer used is `FSG 1.0`.

![alt text](image-27.png)


#### Unpacking:

* When the file is opened in IDA it only contains very few functions and a very small import table.

![alt text](image-29.png)


* Combing through the binary in IDA a specfic `jmp` stood out.

![alt text](image-30.png)

This jump while at first it stands out it ends up just being part of a looping call for the function.

However if we follow this call we can see that there is another jump instruction 

![alt text](image-31.png)

This jump appears to go to the end of a code section with invalid instructions and it jumps far away from the current addresses of the code being executed. This could likely be a tail jump.

Below we can observe the section the code jumps to.

![alt text](image-32.png)


Lets pick this apart in a debugger and see what is going on. I am going to set a breakpoint on the first address we found as well as the second.

![alt text](image-33.png)

As the program runs you can watch the addresses increase in `esi` and `edi` this is likely used with the packing algorithim.

![alt text](image-34.png)

![alt text](image-35.png)

![alt text](image-36.png)


Now im going to jump to the next breakpoint that we marked.

![alt text](image-37.png)

This time the jump is not taken so lets run the program until we see the jump is taken.

![alt text](image-38.png)

Each time we run it we can see a new string stored in `edi`

![alt text](image-39.png)

Here we can see the jump will be taken.

![alt text](image-40.png)

Lets take it...

![alt text](image-41.png)

Now lets run an analysis in x32dbg using `anlaysis` > `"analyze module"`

Then we can pick apart the disasembly and make sure it looks like unpacked code...


Here we can see the functions:

![alt text](image-42.png)


Clicking into `sub_401090` which is where our jump took us we can open up a disassembly page and see what it is doing.

After picking around a little bit I found the following:

![alt text](image-43.png)

That looks pretty unpacked to me. Lets dump it and open it up with `IDA.`

I will be dumping with `scylla`

![alt text](image-44.png)

> Remmeber to use the IAT autosearch first.

![alt text](image-45.png)

> You can additionally use `fix dump` on the dumped file if something doesnt look right.

Now we can work in `IDA`

![alt text](image-46.png)

------

## Sample 3


DIE results:

![alt text](image-47.png)

Here we can see that DIE believes the file is packed using PE Compact.


pestuido:

![alt text](image-48.png)

We get the same conclusion in pestudio.



Looking in `IDA` we can see a small ammount of imports, functions and no readable strings.

![alt text](image-50.png)

![alt text](image-51.png)


#### Auto Unpacking:

Do to this sample using PE Compact we can attempt to unpack it with peid and the generic unpacker plugin.

![alt text](image-52.png)

After clicking the arrow we can see it found a `OEP`

![alt text](image-53.png)

![alt text](image-54.png)

![alt text](image-55.png)


After unpacking with peid we can open the unpacked binary with `IDA` and see all sorts of unpacked strings.

![alt text](image-49.png)

We could now continue analysis in `IDA` or another dissasembler with the unpacked binary.


#### Manual Unpacking:








------

## Sample 5 (Lab18_05.exe): 

LoadLibraryA often used with packed malware.
![alt text](image-16.png)

DIE (Upack like)
![alt text](image-17.png)


High entropy
![alt text](image-18.png)










## References/Resources:


* Mastering Malware Analysis by Alexey Kleymenov and Amr Thabet
  * <https://www.packtpub.com/en-us/product/mastering-malware-analysis-9781803240244>

* Practical Malware Analysis by Michael Sikorski and Andrew Honig
  * <https://nostarch.com/malware>

* <https://learn.microsoft.com/en-us/windows/win32/seccng/cng-portal>

* <https://learn.microsoft.com/en-us/windows/win32/seccrypto/cryptography-portal>

* <https://www.aldeid.com/wiki/PEB-Process-Environment-Block/NtGlobalFlag>

* <https://www.aldeid.com/wiki/PEB-Process-Environment-Block/ProcessHeap>

* <https://anti-debug.checkpoint.com/>

* <https://www.jaiminton.com/Tutorials/PracticalMalwareAnalysis/Chapter18/#>

* <https://www.infosecinstitute.com/resources/malware-analysis/analyzing-packed-malware/>