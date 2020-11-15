* Terminology
  * **short/half word** word 2-bytes
  * **double word** DWORD 4-bytes
* 0x250 Getting your hands dirty
  * 0x251 The bigger Picture   
    `objdump -M intel -D a.out|grep -A20 main.:`  
    * disassembly of a.out
    * `-A20` 20 lines after regex
    * `-M intel` set flavor intel
  * 0x252 The x86 Processor
    * `gdb -q a.out` switch -q for quite
      * `(gdb) break main`
      * `(gdb) run`
      * `(gdb) info registers` 32 Bit
        * **EAX** Accumulator
        * **ECX** Counter
        * **EDX** Data
        * **EBX** Base
        * **ESP** Stack Pointer, 32 Bit address
        * **EBP** Base Pointer, 32 Bit address
        * **ESI** Source Index
        * **EDI** Destination Index
        * **EIP** Instruction Pointer
      * `(gdb) info registers` 64 Bit
        * RAX, RBX, RCX, ... //64 Bit address
        * `print $ebp - 4` print command for simple math
        * `nexti` next instruction
      * `(gdb) quit`
  * 0x253 Assembly Language
    * `echo "set disassembly-flavor intel" > ~/.gdbinit` set permantly disassembly flavor for gdb
    * intel flavor: `operation <destination>, <source>`
    * `gcc -g <programm>` compile with debug symbols
    * **GDB** examine the memory `x/3i $eip`
      * **o** Display in octal
      * **x** Display in hexadecimal
      * **u** Display in unsigned, standard base-10 decimal
      * **t** Display in binary
      * **s** Display as string
      * **i** Display as instruction
      * **b,h,w,g** behind the format to change unit size
    * little-endian byte order = least significant byte first
* 0x260 Back to Basics
  * 0x261 Strings
    * libs: stdio.h string.h
    * use safe libaries ~strcpy~ => strncpy
  * 0x263 Pointers
    * `&` address of operator
    * `*` dereference operator
    ```
    reader@hacking:~/booksrc $ ./a.out  
    int_ptr = 0xbffff834 
    &int_ptr = 0xbffff830 
    *int_ptr = 0x00000005 
    int_var is located at 0xbffff834 and contains 5 
    int_ptr is located at 0xbffff830, contains 0xbffff834, and points to 5
    ```
  * 0x264 Format Strings
    * `%d` Decimal
    * `%u` Unsigned decimal
    * `%x` Hexadecimal
    * `%s` String
    * `%n` number of written bytes so far
    * `%p` Pointer short for 0x%08x
  * 0x265 Typecasting
    * `(typecast_data_type) variable`
    ```C
    int main(){
        int a, b;
        float c, d;
        
        a = 13;
        b = 5;
        
        c = a / b;                  // c = 2.0, divide using integers
        d = (float) a / (float) b;  // d = 2.6, divide using floats with typecast
    }
    ```
    *  `(typecast_data_type *) pointer`
  * 0x266 Command-Line Arguments
    ```C
    #include <stdio.h>
    
    int main(int argc, char *argv[]){
        int i;
        printf("There were %d arguments provided:\n", argc);
        for(i=0; i<argc; i++){
            printf("argument #%d\t-\t%s\n", i ,argv[i]);
        }
    }
    ```
  * 0x267 Variable Scoping
    * `static variable` will only be initialised ones
* 0x270 Memory Segmentation
    * five segments: text, data, bss, heap, stack
