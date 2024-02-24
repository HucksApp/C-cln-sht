# C Clean sheet


Compilation process

(f_ext -> File Extensions)

serial  |               process       | input F_ext  | output f_ext  |      gcc command          |  process  Description
--------|-----------------------------|--------------|---------------|---------------------------|---------------------------------
1       | Preprocessing               | .c           | .i            | gcc -E <file.c>           | All the statements starting with the # symbol in a C program are processed by the pre-processor (directives are processed)
2       | Compiling                   | .i           | .s            | gcc -S <file.c>           | convert the intermediate (.i) file into an Assembly file (.s) having assembly level instructions (low-level code)
3       | Asembling                   | .s           | .o            | gcc -c <file.c>           | Assembly level code (.s file) is converted into a machine-understandable code (object files .o)
4       | Linking                     | .o           |               | gcc -o <file.c>           | links all object codes, or library memory address in case of dynamic library



Library

os            | Static library f_ext  | Dynamic library f_ext
--------------|-----------------------|------------------------
Windows       | .lib                  | .dll
Linux         | .a                    | .so

sn        |     operation     |          Static library                           |       Dynamic library     | Description
----------|-------------------|---------------------------------------------------|---------------------------|------------------------
0         | Description       | Static archive library                            |                           |
1         | create            | ar rc <out.a> <inp.o> <inp2.o> .... |             |                           | list of object codes 
2         | indexing          | ranlib <out.a>                      |
3         | Link (gcc)        | gcc -o <out> <ip1.o> <ip2.o> ...  -L. <lib.a> 



```
 To use the created library in c code.
 Just create the function signature header from the lib

void fn(int); /* function from local lib  header*/

```
  



