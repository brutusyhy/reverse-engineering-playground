1. Test the .exe with Exeinfo PE, with the following result:
```
 x64 - MinGW-w64 GCC: (GNU) compiler (exe) - [v.Mingw- - no  libgcj-1.x ]  - http://mingw-w64.sourceforge.net , 
Overlay : 2E6669... Nothing detected
```
Thus, we can safely analyze it using x64dbg

2. Open the .exe with x64dbg, search for all string references in the current module, we find the following ones:
```
Address=0000000000401631
Disassembly=lea rcx,qword ptr ds:[404000]
String Address=0000000000404000
String="Digite a senha: "

Address=00000000004016C4
Disassembly=lea rcx,qword ptr ds:[404013]
String Address=0000000000404013
String="Sua senha esta correta"

Address=00000000004016D2
Disassembly=lea rcx,qword ptr ds:[40402A]
String Address=000000000040402A
String="Sua senha esta errada"

```
Clearly, these are the places where the key is checked

3. If we set a breakpoint on 0000000000401631 and let the program run, however, it will exit! Why is that?

Take a look at the code surrounding that line:
```
000000000040162C | call veryeasycrackme.40159C                                          |
0000000000401631 | lea rcx,qword ptr ds:[404000]                                        | 0000000000404000:"Digite a senha: "
0000000000401638 | call <JMP.&printf>                                                   |
```

If we take a look at the function veryeasycrackme.40159C, the reason becomes obvious:
```
000000000040159C | push rbp                                                             |
000000000040159D | mov rbp,rsp                                                          |
00000000004015A0 | sub rsp,30                                                           |
00000000004015A4 | mov dword ptr ss:[rbp - 4],0                                         |
00000000004015AB | mov rax,qword ptr ds:[<IsDebuggerPresent>]                           |
00000000004015B2 | call rax                                                             |
00000000004015B4 | mov dword ptr ss:[rbp - 4],eax                                       |
00000000004015B7 | cmp dword ptr ss:[rbp - 4],1                                         |
00000000004015BB | jne veryeasycrackme.4015CB                                           |
00000000004015BD | mov ecx,1                                                            |
00000000004015C2 | mov rax,qword ptr ds:[<ExitProcess>]                                 |
...

```
The function will check if the program is being debugged. If it is, the jne on line will fail, leading to termination.

4. We change the line 00000000004015BB to jmp and run, the program will still terminate, since there's another function call testing the debugger's presence:
```
0000000000401550 | push rbp                                                             |
0000000000401551 | mov rbp,rsp                                                          |
0000000000401554 | sub rsp,30                                                           |
0000000000401558 | mov dword ptr ss:[rbp - 4],0                                         |
000000000040155F | mov rax,qword ptr ds:[<GetCurrentProcess>]                           |
0000000000401566 | call rax                                                             |
0000000000401568 | mov rcx,rax                                                          | rcx:NtQueryInformationProcess+14
000000000040156B | lea rax,qword ptr ss:[rbp - 4]                                       |
000000000040156F | mov rdx,rax                                                          |
0000000000401572 | mov rax,qword ptr ds:[<CheckRemoteDebuggerPresent>]                  |
0000000000401579 | call rax                                                             |
000000000040157B | test eax,eax                                                         |
000000000040157D | je veryeasycrackme.401595                                            |
000000000040157F | mov eax,dword ptr ss:[rbp - 4]                                       |
0000000000401582 | cmp eax,1                                                            |
0000000000401585 | jne veryeasycrackme.401595                                           |
0000000000401587 | mov ecx,1                                                            |
000000000040158C | mov rax,qword ptr ds:[<ExitProcess>]                                 |
```
The jne jump on 0000000000401585, again, will fail if a remote debugger is present, so we will patch it to jmp.

5. Let the debugging proceed and enter the password, we can easily identify the comparison code down the line:
```
00000000004016B6 | call <JMP.&memcmp>                                                   |
00000000004016BB | mov dword ptr ss:[rbp - 8],eax                                       |
00000000004016BE | cmp dword ptr ss:[rbp - 8],0                                         |
00000000004016C2 | jne veryeasycrackme.4016D2                                           |
00000000004016C4 | lea rcx,qword ptr ds:[404013]                                        | 0000000000404013:"Sua senha esta correta"
00000000004016CB | call <JMP.&puts>                                                     |
00000000004016D0 | jmp veryeasycrackme.4016DE                                           |
00000000004016D2 | lea rcx,qword ptr ds:[40402A]                                        | 000000000040402A:"Sua senha esta errada"
```
If our password is incorrect, the jne jump on 00000000004016C2 will succeed. Thus, we will simply remove this jump so that the correct branch will be entered.

5. Apply all the patches (although the first two are unnecessary outside of debugger), and run the file. Now we will pass the authentification no matter what password we entered!