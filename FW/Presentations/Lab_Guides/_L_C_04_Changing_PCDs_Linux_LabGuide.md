## Slide 1
# UEFI & EDK II Training

## Changing PCDs with Linux Labs
<br>
<a href='http://www.tianocore.org'>tianocore.org</a>

<!---
 Lab_Guide.md for How to Write a Changing PCDs with Linux Labs

  Copyright (c) 2020-2022, Intel Corporation. All rights reserved.<BR>

  Redistribution and use in source (original document form) and 'compiled'
  forms (converted to PDF, epub, HTML and other formats) with or without
  modification, are permitted provided that the following conditions are met:

  1) Redistributions of source code (original document form) must retain the
     above copyright notice, this list of conditions and the following
     disclaimer as the first lines of this file unmodified.

  2) Redistributions in compiled form (transformed to other DTDs, converted to
     PDF, epub, HTML and other formats) must reproduce the above copyright
     notice, this list of conditions and the following disclaimer in the
     documentation and/or other materials provided with the distribution.

  THIS DOCUMENTATION IS PROVIDED BY TIANOCORE PROJECT "AS IS" AND ANY EXPRESS OR
  IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
  MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO
  EVENT SHALL TIANOCORE PROJECT  BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
  SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
  PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS;
  OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
  WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
  OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS DOCUMENTATION, EVEN IF
  ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

-->

---  
## Slide 2 @title[Lesson Objective]

### Lesson Objective 

- UEFI Application with PCDs



---
## Slide 3 @title[UEFI Application w/ PCDs Section]

### UEFI Application w/ PCDs 

---
## Slide 4 @title[EDK II PCD’s Purpose and Goals]
<br>
<p align="center"><b>EDK II PCD’s Purpose and Goals</b> - REVIEW</p>
Documentaton :  <a href="https://github.com/tianocore/edk2/blob/master/MdeModulePkg/Universal/PCD/Dxe/Pcd.inf"> MdeModulePkg/Universal/PCD/Dxe/Pcd.inf  </a> 

Purpose

- Establishes platform common definitions 
- Build-time/Run-time aspects 
- Binary Editing Capabilities 

Goals

- Simplify porting 
- Easy to associate with a module or platform 

Note:

---



## Slide 5 @title[PCD Syntax review]
### <p align="center">PCD Syntax  - REVIEW 
PCDs can be located anywhere within the Workspace even though a different package will use those PCDs for a given project</span>

Note:

- The Platform configuration database is generated by the build process parsing the build description files that define and specify PCD entries.

- What we see on this slide is how the PCD data is being used in various levels of the build description files

- First we have the DEC file – this Defines a list of PCD tokens that modules can use. 
 	It Defines the PCD entries that will exist under the GUID for that  package, the PCD restriction, valid types for the PCD, and a default value for the PCD. There is a whole syntax and how to define a PCD in the DEC file.

- Next we have PCB entries in the INF file- and this Defines the usage of PCD tokens by the module.
	It Defines what PCD entries are being used within the module, the PCD restriction (or DYNAMIC for none), and a Optional default value for the PCD within this module only.

- Next is the DSC file – This is at the Platform level and describes the contents of the build for a specific platform. 
	PCD entries are assigned values and types for the platform build. You would define a value here to be used by that platform. The value could be different when it is defined in the DEC file but the value in the DSC would be the final value . And They Cannot conflict with established restrictions.

- Not on this slide but also there is the FDF build description File – and this file would have flash layout related values 
---
## Slide 6 @title[Lab 1: Writing UEFI Applications with PCDs]

### Lab 1: Writing UEFI Applications with PCDs

In this lab, you’ll  learn how to write UEFI applications with PCDs.

Note:




---
## Slide 7 @title[EDK II HelloWorld  App  Lab ]
<b>EDK II HelloWorld  App  Lab  </b></p>

First Setup for Building EDK II for Emulator, See <a href="https://github.com/tianocore-training/Presentation_FW/blob/main/FW/Presentations/Lab_Guides/_L_C_01_Platform_Build_OVMF-QEMU_Lab_guide.md">Lab Setup for Emulator </a>

Locate and Open <br>
`edk2/MdeModulePkg/Application/HelloWorld/HelloWorld.c`

Notice the PCD values
<br>
<br>
<br>
Build Emulation <br>
Then Run HelloWorld at the Shell command interface 


Edit and add the following line (at the end of the file)


Edit OvmfPkg/OvmfPkgX64.dsc  add HelloWorld.inf - Save

```
[Components]
. . . 
# Add new modules here
 MdeModulePkg/Application/HelloWorld/HelloWorld.inf
```
Build the OvmfPkgX64 from Terminal Prompt (Cnt-Alt-T)
```
 bash$ cd ~/src/edk2-ws/edk2
 bash$ build  –D ADD_SHELL_STRING 
```


Note:

---



 

## Slide 8 @title[EDK II HelloWorld  App  Lab steps]
<p align="left"><b>EDK II HelloWorld  App  Lab  </b></p>



Copy the HelloWorld.efi to the ~run-ovmf/hda-contents directory
```
  bash$ cd ~/run-ovmf/hda-contents
  bash$ cp ~/src/edk2-ws/Build/OvmfX64/DEBUG_GCC5/X64/HelloWorld.efi
```
CD to the run-ovmf directory and run Qemu with the RunQemu.sh shell
 ```
  bash$ cd ~/run-ovmf
  bash$ . RunQemu.sh
```
At the UEFI Shell prompt
```
Shell> Helloworld
UEFI Hello World!
Shell> 
```


<font color="green">How can we force the HelloWorld application to print out 3 times ?</font>


Note:

---
## Slide 9 @title[EDK II HelloWorld  App  Lab location]
### <b>EDK II HelloWorld  App  Lab  </b>

<a href="https://github.com/tianocore/edk2/tree/master/MdeModulePkg/Application/HelloWorld"> MdeModulePkg/Application/HelloWorld </a>


Note:

First let's look at the source code for the HellowWorld Application

---
## Slide 10 @title[EDK II HelloWorld  App  Lab code]
### <b>EDK II HelloWorld  App  Lab  </b> 
<span style="font-size:01.0em" >Source: <font color="green">Helloworld.c</font></span>

```C++
EFI_STATUS
EFIAPI
UefiMain (
  IN EFI_HANDLE        ImageHandle,
  IN EFI_SYSTEM_TABLE  *SystemTable
  )
{
	UINT32 Index;
	Index = 0;
  // Three PCD type (FeatureFlag, UINT32 
  // and String) are used as the sample.
  if (FeaturePcdGet (PcdHelloWorldPrintEnable)) {
  	for (Index = 0; Index < PcdGet32   (PcdHelloWorldPrintTimes); Index ++) {
  	  
  	  // Use UefiLib Print API to print
      // string to UEFI console
  	  
    	Print ((CHAR16*)PcdGetPtr (PcdHelloWorldPrintString));
    }
  }

  return EFI_SUCCESS;
}
```

Note:


Source from Helloworld.c 

---

## Slide 11 @title[EDK II HelloWorld  App  Lab solution]
### <b>EDK II HelloWorld  App  Solution </b> 


Note:

- look in file: `C:\fw\edk2\MdeModulePkg\MdeModulePkg.Dec` 
 
- This PCD defines the print string.
-  This PCD is a sample to explain String typed PCD usage.
-  `gEfiMdeModulePkgTokenSpaceGuid.PcdHelloWorldPrintString|L"UEFI Hello World!\n"|VOID*|0x40000004`

Solution:
1. Edit the file ~src/edk2-ws/edk2/OvmfPkg/OvmfPkgX64.dsc
  - After the section [PcdsFixedAtBuild], add the new line :  
  - `[PcdsFixedAtBuild]`
  - `gEfiMdeModulePkgTokenSpaceGuid.PcdHelloWorldPrintTimes|3`


2. Re-Build – Cd to ~/src/edk2-ws/edk2~ dir 
```
bash$ build –D ADD_SHELL_STRING
```
3. Copy  Helloworld.efi 	 
```
bash$ cd ~/run-ovmf/hda-contents
bash$ cp ~/src/Build/OvmfX64/DEBUG_GCC5/X64/HelloWorld.efi .
```

---
## SLide 12 @title[EDK II HelloWorld  App  Lab solution 02]
### <b>EDK II HelloWorld  App  Solution </b> 


Note:

4. Run QEMU
```
 bash$ cd ~/run-ovmf
 bash$ . RunQemu.sh
```

5. At the Shell prompt
```
 Shell> Helloworld
 UEFI Hello World!
 UEFI Hello World!
 UEFI Hello World!
 Shell> 
```
6. Exit QEMU

- How can we change the string of the HelloWorld application?
- Also see  ~src/edk2-ws/edk2/MdeModulePkg/MdeModulePkg.Dec




---  
## Slide 13 @title[Summary]
<BR>
<p align="left"><span class="gold"   >Summary  <br>

- UEFI Application with PCDs
- Simple UEFI Application
- Add functionality to UEFI Application
- Using EADK with UEFI Application

 

---
## SLide 14 @title[Questions]
<br>


---
## Slide 15


Return to Training Table of contents for next presentation link: https://github.com/tianocore-training/Tianocore_Training_Contents/wiki



---
## Slide 16 @title[Logo Slide]
<br><br><br>


---
## SLide 17 @title[Acknowledgements]
<p align="left"><span class="gold"   >Acknowledgements 

```c++
/**
Redistribution and use in source (original document form) and 'compiled' forms (converted
to PDF, epub, HTML and other formats) with or without modification, are permitted provided
that the following conditions are met:

Redistributions of source code (original document form) must retain the above copyright 
notice, this list of conditions and the following disclaimer as the first lines of this 
file unmodified.

Redistributions in compiled form (transformed to other DTDs, converted to PDF, epub, HTML
and other formats) must reproduce the above copyright notice, this list of conditions and 
the following disclaimer in the documentation and/or other materials provided with the 
distribution.

THIS DOCUMENTATION IS PROVIDED BY TIANOCORE PROJECT "AS IS" AND ANY EXPRESS OR IMPLIED 
WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND 
FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL TIANOCORE PROJECT BE 
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES 
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, 
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, 
WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) 
ARISING IN ANY WAY OUT OF THE USE OF THIS DOCUMENTATION, EVEN IF ADVISED OF THE POSSIBILITY 
OF SUCH DAMAGE.

Copyright (c) 2022, Intel Corporation. All rights reserved.
**/

```