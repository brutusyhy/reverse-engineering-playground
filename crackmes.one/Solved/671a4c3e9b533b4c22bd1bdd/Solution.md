1. Use Exeinfo PE to test the .exe:
```
x64 Microsoft Visual C++ v14.41 - 202x - ExE Debug ( jmp/jmp/jmp/int3 ) - microsoft.com [ Win Vista ]
```
We are good to go with x64dbg

2. In x64dbg, search for string references in the current module:
```
Address=00007FF6EEC81A15
Disassembly=lea rcx,qword ptr ds:[7FF6EEC8ACA8]
String Address=00007FF6EEC8ACA8
String="Enter the password: "
Address=00007FF6EEC81A88
Disassembly=lea rcx,qword ptr ds:[7FF6EEC8ACC8]
String Address=00007FF6EEC8ACC8
String="Correct"
Address=00007FF6EEC81A97
Disassembly=lea rcx,qword ptr ds:[7FF6EEC8ACD4]
String Address=00007FF6EEC8ACD4
String="Wrong"

```
We follow the 00007FF6EEC81A15 to see the password logic

3. Let's have a look at the password logic
```
00007FF6EEC81A15 | lea rcx,qword ptr ds:[7FF6EEC8ACA8]                                  | 00007FF6EEC8ACA8:"Enter the password: "
00007FF6EEC81A1C | call simple crackme #2.7FF6EEC8119F                                  |
00007FF6EEC81A21 | nop                                                                  |
00007FF6EEC81A22 | xor ecx,ecx                                                          |
00007FF6EEC81A24 | call qword ptr ds:[<__acrt_iob_func>]                                |
00007FF6EEC81A2A | mov r8,rax                                                           | r8:&"ALLUSERSPROFILE=C:\\ProgramData"
00007FF6EEC81A2D | mov edx,C                                                            | C:'\f'
00007FF6EEC81A32 | lea rcx,qword ptr ss:[rbp + 8]                                       |
00007FF6EEC81A36 | call qword ptr ds:[<fgets>]                                          |
00007FF6EEC81A3C | nop                                                                  |
00007FF6EEC81A3D | lea rdx,qword ptr ds:[7FF6EEC8B2B8]                                  | rdx:&"C:\\git\\reverse-engineering-playground\\crackmes.one\\671a4c3e9b533b4c22bd1bdd\\simple crackme #2.exe"
00007FF6EEC81A44 | lea rcx,qword ptr ss:[rbp + 8]                                       |
00007FF6EEC81A48 | call qword ptr ds:[<strcspn>]                                        |
00007FF6EEC81A4E | mov qword ptr ss:[rbp + 138],rax                                     | [rbp+138]:_get_initial_narrow_environment+9
00007FF6EEC81A55 | cmp qword ptr ss:[rbp + 138],C                                       | [rbp+138]:_get_initial_narrow_environment+9, C:'\f'
00007FF6EEC81A5D | jae simple crackme #2.7FF6EEC81A61                                   |
00007FF6EEC81A5F | jmp simple crackme #2.7FF6EEC81A67                                   |
00007FF6EEC81A61 | call simple crackme #2.7FF6EEC812B2                                  |
00007FF6EEC81A66 | nop                                                                  |
00007FF6EEC81A67 | mov rax,qword ptr ss:[rbp + 138]                                     | [rbp+138]:_get_initial_narrow_environment+9
00007FF6EEC81A6E | mov byte ptr ss:[rbp + rax + 8],0                                    |
00007FF6EEC81A73 | lea rcx,qword ptr ss:[rbp + 8]                                       |
00007FF6EEC81A77 | call simple crackme #2.7FF6EEC813ED                                  |
00007FF6EEC81A7C | mov dword ptr ss:[rbp + 54],eax                                      |
00007FF6EEC81A7F | cmp dword ptr ss:[rbp + 54],D1E7F089                                 |
00007FF6EEC81A86 | jne simple crackme #2.7FF6EEC81A97                                   |
00007FF6EEC81A88 | lea rcx,qword ptr ds:[7FF6EEC8ACC8]                                  | 00007FF6EEC8ACC8:"Correct"
00007FF6EEC81A8F | call simple crackme #2.7FF6EEC8119F                                  |
00007FF6EEC81A94 | nop                                                                  |
00007FF6EEC81A95 | jmp simple crackme #2.7FF6EEC81AA4                                   |
00007FF6EEC81A97 | lea rcx,qword ptr ds:[7FF6EEC8ACD4]                                  | 00007FF6EEC8ACD4:"Wrong"
00007FF6EEC81A9E | call simple crackme #2.7FF6EEC8119F                                  |
```
What's important seems to be the jne jump on 00007FF6EEC81A86, as it will divert us into the wrong branch when it holds. Let's replace this jne with nop, and patch it. Voila! Now entering any password will pass this check.