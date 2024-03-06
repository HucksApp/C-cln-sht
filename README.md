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
  ... ...                      # so recipe syntax should should follow ✅ shell scripting not  ❌ makefile syntax
```                             

## Make command options

option             |   Description
-------------------|-------------
`-f` or  `--file `   | 
`-t` or `--touch `   |
`-q` or `--question` |
`-n` or `--just-print`|
`-C dir` or  `--directory=dir` | Change to directory dir before reading the makefiles
`-d` or `--debug[=options]` |  Print debugging information in addition to normal processing.
➖                 |  options `a` all -> all types of debugging output are enabled. This is equivalent to using `-d'.
➖                 |        `b` basic -> basic Basic debugging prints each target that was found to be out-of-date, and whether the build was successful or not.
➖                 | `v` verbose -> A level above `basic`; includes messages about which makefiles were parsed, prerequisites that did not need to be rebuilt, etc.
➖                 | `i` implicit -> Prints messages describing the implicit rule searches for each target. This option also enables `basic' messages.
➖                 | `j` jobs -> Prints messages giving details on the invocation of specific subcommands.
➖                 |`m` makefile -> By default, the above messages are not enabled while trying to remake the makefiles. This option enables messages while rebuilding makefiles, too. Note that the `all' option does enable this option. This option also enables `basic' messages.

`-e` or `--environment-overrides` | Give variables taken from the environment precedence over variables from makefiles
`-f file` or  `--file=file` or `--makefile=file` | Read the file named file as a makefile
`-h` or `--help` | Remind you of the options that make understands and then exit.
`-i` or `--ignore-errors`  | Ignore all errors in commands executed to remake files
`-I dir` or `--include-dir=dir` |  Specifies a directory dir to search for included makefiles. If several `-I` options are used to specify several directories, the directories are searched in the order specified.
`-j [jobs]` or `--jobs[=jobs]` | Specifies the number of jobs (commands) to run simultaneously. With no argument, make runs as many jobs simultaneously as possible. If there is more than one `-j` option, the last one is effective
`-k` or `--keep-going`  | Continue as much as possible after an error. While the target that failed, and those that depend on it, cannot be remade, the other prerequisites of these targets can be processed all the same
`-l [load]` or `--load-average[=load]` or  `--max-load[=load]` |  Specifies that no new jobs (commands) should be started if there are other jobs running and the load average is at least load (a floating-point number). With no argument, removes a previous load limit
`-n'
`--just-print'
`--dry-run'
`--recon'
Print the commands that would be executed, but do not execute them. See section Instead of Executing the Commands.
`-o file'
`--old-file=file'
`--assume-old=file'
Do not remake the file file even if it is older than its prerequisites, and do not remake anything on account of changes in file. Essentially the file is treated as very old and its rules are ignored. See section Avoiding Recompilation of Some Files.
`-p'
`--print-data-base'
Print the data base (rules and variable values) that results from reading the makefiles; then execute as usual or as otherwise specified. This also prints the version information given by the `-v' switch (see below). To print the data base without trying to remake any files, use `make -qp'. To print the data base of predefined rules and variables, use `make -p -f /dev/null'. The data base output contains filename and linenumber information for command and variable definitions, so it can be a useful debugging tool in complex environments.
`-q'
`--question'
"Question mode". Do not run any commands, or print anything; just return an exit status that is zero if the specified targets are already up to date, one if any remaking is required, or two if an error is encountered. See section Instead of Executing the Commands.
`-r'
`--no-builtin-rules'
Eliminate use of the built-in implicit rules (see section Using Implicit Rules). You can still define your own by writing pattern rules (see section Defining and Redefining Pattern Rules). The `-r' option also clears out the default list of suffixes for suffix rules (see section Old-Fashioned Suffix Rules). But you can still define your own suffixes with a rule for .SUFFIXES, and then define your own suffix rules. Note that only rules are affected by the -r option; default variables remain in effect (see section Variables Used by Implicit Rules); see the `-R' option below.
`-R'
`--no-builtin-variables'
Eliminate use of the built-in rule-specific variables (see section Variables Used by Implicit Rules). You can still define your own, of course. The `-R' option also automatically enables the `-r' option (see above), since it doesn't make sense to have implicit rules without any definitions for the variables that they use.
`-s'
`--silent'
`--quiet'
Silent operation; do not print the commands as they are executed. See section Command Echoing.
`-S'
`--no-keep-going'
`--stop'
Cancel the effect of the `-k' option. This is never necessary except in a recursive make where `-k' might be inherited from the top-level make via MAKEFLAGS (see section Recursive Use of make) or if you set `-k' in MAKEFLAGS in your environment.
`-t'
`--touch'
Touch files (mark them up to date without really changing them) instead of running their commands. This is used to pretend that the commands were done, in order to fool future invocations of make. See section Instead of Executing the Commands.
`-v'
`--version'
Print the version of the make program plus a copyright, a list of authors, and a notice that there is no warranty; then exit.
`-w'
`--print-directory'
Print a message containing the working directory both before and after executing the makefile. This may be useful for tracking down errors from complicated nests of recursive make commands. See section Recursive Use of make. (In practice, you rarely need to specify this option since `make' does it for you; see section The `--print-directory' Option.)
`--no-print-directory'
Disable printing of the working directory under -w. This option is useful when -w is turned on automatically, but you do not want to see the extra messages. See section The `--print-directory' Option.
`-W file'
`--what-if=file'
`--new-file=file'
`--assume-new=file'
Pretend that the target file has just been modified. When used with the `-n' flag, this shows you what would happen if you were to modify that file. Without `-n', it is almost the same as running a touch command on the given file before running make, except that the modification time is changed only in the imagination of make. See section Instead of Executing the Commands.
`--warn-undefined-variables'
Issue a warning message whenever make sees a reference to an undefined variable. This can be helpful when you are trying to debug makefiles which use variables in complex ways.












## Make Variable

* Declare ->  `var1 = text text text` or   `r = text text text`
*  Assignment
  *  `=` recursively expanded -> value can not be another variable or function call ❌ `var2 = $(var1) text` but  ✅  `var2 = text text`
  *  `:=` simply expanded  ->  value can be another variable or function call ✅  `var2 := $(var1) text` and ✅ `var2 := text text`
  *  `+=` appending assignment -> append the new value to the end of old value `var1 += new value`
  *  `?=`  conditional assignment ->  set to a value to a variable only if variable is not already set `var1 ?= value`
* reference -> `$(var1)  ` or  `${var1}` and `$(r)` or `${r}` or can only be a letter not word if referenced withouth -> {}, () ✅ `$r` but not ❌ `$var1`

## Make Function
* Declare ->  
* reference -> 

## Directive

### Conditional Directive

**make conditional statement**
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
* `ifeq (arg1, arg2)`  or `ifeq "arg1" "arg2"` -> if arg1 and arg2 are equal
* `ifneq (arg1, arg2)` or `ifneq "arg1" "arg2"` -> if arg1 aand arg2 are not equal
* `ifdef variable-name` -> if variable is defined
* `ifndef variable-name` -> if variable is not defined

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


### override Directive
```
override variable = value
```
or
```
override variable := value
```
To append more text to a variable defined on the command line, use:
```
override variable += more text
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


Variables        |  Description
-----------------|------------------ 
`.PHONY`         | This prevents make from getting confused that the target is a process not a file
`.RECIPEPREFIX`  | Character that begings every recipe line. Default is a tab, e.g `.RECIPEPREFIX = >`.
`.DEFAULT_GOAL`  | the first goal in the makefile is default. e.g  `.DEFAULT_GOAL := <prefered default goal>`
`.INCLUDE_DIRS`  |  contain the current list of directories that make will search for included files
`MAKEFILE_LIST`  | Contains the name of each makefile that is parsed by make, in the order in which it was parsed.
`MAKE_RESTARTS`  | This variable is set only if this instance of make has restarted or remade. it will contain the number of times this instance has restarted
`MAKE_TERMOUT`   | Make stdout variable, if populated make sends to stdout
`MAKE_TERMERR`   | Make stderr variable, if populated make sends to stderr
`.VARIABLES`     | Expands to a list of the names of all global variables defined so far.
`.FEATURE`S      | Expands to a list of special features supported by this version of make
`.INCLUDE_DIRS`  | Expands to a list of directories that make searches for included makefiles
`.EXTRA_PREREQS` | Each word in this variable is a new prerequisite which is added to targets for which it is set.
`$@`             | The file name of the target of the rule
`$%`             | The target member name, when the target is an archive member
`$<`             | The name of the first prerequisite
`$^`             | The names of all the prerequisites, with spaces between them.
`$+`             | This is like `$^`, but prerequisites listed more than once are duplicated in the order they were listed in the makefile
`$?`             | The names of all the prerequisites that are newer than the target, with spaces between them
`$\|`            | The names of all the order-only prerequisites, with spaces between them
`$*`             | The stem with which an implicit rule matches 
`$(@D)`          | The directory part of the file name of the target, with the trailing slash removed
`$(@F)`          | The file-within-directory part of the file name of the target.
`$(*D)` or `$(*F)` |  The directory part and the file-within-directory part of the stem; dir and foo in this example.
`$(%D)` or `$(%F)` | The directory part and the file-within-directory part of the target archive member name.
`$(<D)` or `$(<F)` | The directory part and the file-within-directory part of the first prerequisite.
`$(^D)` or `$(^F)` | Lists of the directory parts and the file-within-directory parts of all prerequisites.
`$(+D)` or `$(+F)` | Lists of the directory parts and the file-within-directory parts of all prerequisites, including multiple instances of duplicated prerequisites.
`$(?D)` or `$(?F)` | Lists of the directory parts and the file-within-directory parts of all prerequisites that are newer than the target
