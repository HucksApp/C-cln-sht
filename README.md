![image](https://github.com/HucksApp/C-cln-sht/assets/58187974/d242267e-822f-4533-b241-8d02a7b20167)# C Clean sheet 


## Compilation process

(f_ext -> File Extensions)

serial  |               process       | input F_ext  | output f_ext  |      gcc command          |  process  Description
--------|-----------------------------|--------------|---------------|---------------------------|---------------------------------
1       | Preprocessing               | .c           | .i            | gcc -E <file.c>           | All the statements starting with the # symbol in a C program are processed by the pre-processor (directives are processed)
2       | Compiling                   | .i           | .s            | gcc -S <file.c>           | convert the intermediate (.i) file into an Assembly file (.s) having assembly level instructions (low-level code)
3       | Asembling                   | .s           | .o            | gcc -c <file.c>           | Assembly level code (.s file) is converted into a machine-understandable code (object files .o)
4       | Linking                     | .o           |               | gcc -o <file.c>           | links all object codes, or library memory address in case of dynamic library



### C Library 🏛
The creation of a library is as follows for both static and shared
* Compile a list of .c files to object files
* Insert all object file into a library file

  Libraries format lib\<name>.a, lib\<name>.so ...

os            | Static library f_ext  | Dynamic library f_ext
--------------|-----------------------|------------------------
Windows       | .lib                  | .dll
Linux         | .a                    | .so

sn        |     operation     |          Static library                           |               Dynamic library               | Description
----------|-------------------|---------------------------------------------------|---------------------------------------------|------------------------
0         | Description       | Static archive library                            | Shared                                      |
1         | Object files (gcc)| gcc -c <inp1.c>                                   | gcc -fPIC -c <inp1.c>                       |           
1         | Lib creation      | ar rc <out.a> <inp.o> <inp2.o> ....               | gcc -shared lib\<name>.so <ip1.o> <ip2.o> ...| list of object codes 
2         | indexing          | ranlib <out.a>                                    |
3         | Link (gcc)        | gcc -o <out> <ip1.o> <ip2.o> ...  -L. lib\<name>.a | gcc <ip1.o> <ip2.o>  -L. -l \<name> -o \<out>|


**Position Independent Code (PIC)** - When the object files are generated, we have no idea where in memory they will be inserted in a program that will use them. Many different programs may use the same library, and each load it into a different memory in address. Thus, we need that all jump calls ("goto", in assembly speak) and subroutine calls will use relative addresses, and not absolute addresses. Thus, we need to use a compiler flag that will cause this type of code to be generated.
In most compilers, this is done by specifying '-fPIC' or '-fpic' on the compilation command.


The linker will look for the file 'lib\<name>.so' (-l\<name>) in the current directory (-L.)
use the 'LD_LIBRARY_PATH' environment variable to tell the dynamic loader to look in other directories.
```


$ echo $LD_LIBRARY_PATH


#LD_LIBRARY_PATH is not defined:

$ LD_LIBRARY_PATH=/full/path/to/library/directory
$export LD_LIBRARY_PATH


# LD_LIBRARY_PATH already defined:

$ LD_LIBRARY_PATH=/full/path/to/library/directory:${LD_LIBRARY_PATH}
$ export LD_LIBRARY_PATH
 
```

```
 To use the static created library in c code.
 Just create the function signature header from the lib

void fn(int); /* function from local lib  header*/

```


### Loading A Shared Library Using dlopen()
In order to open and load the shared library, one should use the dlopen() function.

```
#include <dlfcn.h>      /* defines dlopen(), etc.       */
.
.
void* lib_handle;       /* handle of the opened library */

lib_handle = dlopen("/full/path/to/library", RTLD_LAZY);
if (!lib_handle) {
    fprintf(stderr, "Error during dlopen(): %s\n", dlerror());
    exit(1);
}
```

### Calling Functions Dynamically Using dlsym()
```
struct local_file* (*readfile)(const char* file_path);
/* then define a pointer to a possible error string */
const char* error_msg;
/* finally, define a pointer to the returned file */
struct local_file* a_file;

/* now locate the 'readfile' function in the library */
readfile = dlsym(lib_handle, "readfile");

/* check that no error occured */
error_msg = dlerror();
if (error_msg) {
    fprintf(stderr, "Error locating 'readfile' - %s\n", error_msg);
    exit(1);
}

/* finally, call the function, with a given file path */
a_file = (*readfile)("hello.txt");
```




### Unloading A Shared Library Using dlclose()

```
$ dlclose(lib_handle);
```

## MAKE
make can be considered the center of the development process by providing a roadmap of an application’s components and how they fit together.

## makefile 📄
this contains actions (Goals) for make process.

### ***Goal*** Structure 🔎📄
```
# this is a one goal from the goals in a make file

target ... : prerequisites
  ... recipe
  ... ...
```

Terms        |    Description
-------------|-----------------
\#            | commenting 
\\            | string new line or escape character for special characters like `\#`
Goal         | Complete single make process -> target, prerequisites, recipe
Target       | name of a file that is generated by a program or name of an action to carry out eg name.o,  clean 
prerequisite | required conditions needed to exits, mostly files needed to create the target
recipe       | Actions that make carries out. It more than one command, either on the same line or each on its own line. its starts with tab character at the beginning of every recipe line. **if you wish to change recipe starting character. set `.RECIPEPREFIX` Variable**
Default Goal | the first goal in the makefile is default. To change default goal. set `.DEFAULT_GOAL` variable like `.DEFAULT_GOAL := <prefered default goal>`


Variables  |    Description
-----------|------------------ 
