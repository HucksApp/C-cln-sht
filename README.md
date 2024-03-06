# C Clean sheet 


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
makefile contains a set of rules or actions (Goals) used to build an application
**usage**
* default file `make`
* named file `make -f <makefile named>` or `make --file <makefile named>`
* make target `make <Target>`

**make default file name are searched in this hierachy**
* GNUmakefile
* Makefile

**To use multiple makefile, use include directive**
```
include filenames…
```
or to neglect File Not Found error if included file do not exists add -

```
-include filenames…
```
The default search directories are in this order (1) /usr/local/include,  (2) /usr/gnu/include, (3) /usr/local/include, (4) /usr/include.
**To specify the directory the included file is at, use the***
`-I` or `--include-dir` options with make



### ***Goal*** Structure 🔎📄
```
# this is a one goal from the goals in a make file

target ... : prerequisites     -----------------------
  ... recipe                   # all recipe (shell commands and process) are passed to the shell directly
  ... ...                      # so recipe syntax should should follow shell scripting not makefile syntax
```                             

## Make command options

option             |   Description
-------------------|-------------
`-f or  --file `   | 
`-t or --touch `   |
`-q or --question` |
`-n or --just-print`| 
## Make variable
* Declare: `var1 = text text text`



## Make Conditional Statement
```
if conditional-directive-one          
text-if-true
else if conditional-directive-two
text--is-true
else
text-if-one-and-two-are-false
endif
```
**if conditional directive Types** 
* ifeq (arg1, arg2) -> if arg1 and arg2 are equal
* ifneq (arg1, arg2) -> if arg1 aand arg2 are not equal
* ifdef variable-name -> if variable is defined
* ifndef variable-name -> if variable is not defined

#### usage
```
ifeq ($(var1), 1)
  var3 := 1
else ifeq ($(var1),2)
  var3 := 2
else
 var3 := 0
endif
```


Terms        |    Description
-------------|-----------------
\-           | surpress any error `-rm leave.txt` will ignore error if the file do not exists and move to next command
@            | surpress printing recipe statement before executing the recipe which is a default make behaviour.
\#           | comments
\\           | string new line or escape character for special characters like `\#`
Goal         | Complete single make process -> target, prerequisites, recipe
Target       | name of a file that is generated by a program or name of an action to carry out eg name.o,  clean 
prerequisite | required conditions needed to exits, mostly files needed to create the target
recipe       | Actions(shell commands passed by make to the default shell) that make carries out. It more than one command, either on the same line or each on its own line. its starts with tab character at the beginning of every recipe line. **if you wish to change recipe starting character. set `.RECIPEPREFIX` Variable** e.g `.RECIPEPREFIX = >`.
Default Goal | the first goal in the makefile is default. To change default goal. set `.DEFAULT_GOAL` variable like `.DEFAULT_GOAL := <prefered default goal>`


Variables      |    Description
---------------|------------------ 
.PHONY         | This prevents make from getting confused that the target is a process not a file
.INCLUDE_DIRS  |  contain the current list of directories that make will search for included files
MAKEFILE_LIST  | Contains the name of each makefile that is parsed by make, in the order in which it was parsed.
MAKE_RESTARTS  | This variable is set only if this instance of make has restarted or remade. it will contain the number of times this instance has restarted
MAKE_TERMOUT   | Make stdout variable, if populated make sends to stdout
MAKE_TERMERR   | Make stderr variable, if populated make sends to stderr
.VARIABLES     | Expands to a list of the names of all global variables defined so far.
.FEATURES      | Expands to a list of special features supported by this version of make
.INCLUDE_DIRS  | Expands to a list of directories that make searches for included makefiles
.EXTRA_PREREQS | Each word in this variable is a new prerequisite which is added to targets for which it is set.
