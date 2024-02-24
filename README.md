# C Clean sheet


Compilation process

(f_ext -> File Extensions)

serial  |               process       | input F_ext  | output f_ext  |      gcc command          |  process  Description
--------|-----------------------------|--------------|---------------|---------------------------|---------------------------------
1       | Preprocessing               | .c           | .i            | gcc -E <file.c>           | All the statements starting with the # symbol in a C program are processed by the pre-processor (directives are processed)
2       | Compiling                   | .i           | .s            | gcc -S <file.c>           | convert the intermediate (.i) file into an Assembly file (.s) having assembly level instructions (low-level code)
3       | Asembling                   | .s           | .o            | gcc -c <file.c>           | Assembly level code (.s file) is converted into a machine-understandable code (object files .o)
4       | Linking                     | .o           |               | gcc -o <file.c>           | links all object codes, or library memory address in case of dynamic library



### Library
The creation of a library is is as follows for both static and shared
* Compile a list of object files
* Insert them all into a library file

os            | Static library f_ext  | Dynamic library f_ext
--------------|-----------------------|------------------------
Windows       | .lib                  | .dll
Linux         | .a                    | .so

sn        |     operation     |          Static library                           |       Dynamic library     | Description
----------|-------------------|---------------------------------------------------|---------------------------|------------------------
0         | Description       | Static archive library                            |      Shared                     |
1         | Object files (gcc)| gcc -c <inp1.c>                                   | gcc -fPIC -c <inp1.c>
1         | Lib creation      | ar rc <out.a> <inp.o> <inp2.o> ....              |                             | list of object codes 
2         | indexing          | ranlib <out.a>                      
3         | Link (gcc)        | gcc -o <out> <ip1.o> <ip2.o> ...  -L. <lib.a> 


**Position Independent Code (PIC)** - When the object files are generated, we have no idea where in memory they will be inserted in a program that will use them. Many different programs may use the same library, and each load it into a different memory in address. Thus, we need that all jump calls ("goto", in assembly speak) and subroutine calls will use relative addresses, and not absolute addresses. Thus, we need to use a compiler flag that will cause this type of code to be generated.
In most compilers, this is done by specifying '-fPIC' or '-fpic' on the compilation command.

```
 To use the created library in c code.
 Just create the function signature header from the lib

void fn(int); /* function from local lib  header*/

```
  



