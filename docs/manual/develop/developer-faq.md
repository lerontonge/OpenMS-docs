Developer FAQ
=============

The following contains answers to typical questions from developers about OpenMS.

## General

The following section provides general information to new contributors.

### I am new to OpenMS. What should I do first?

* Check out the development version of OpenMS (see website).
* Build OpenMS by following the installation instructions or [from source](/about/installation.rst).
* Read the [OpenMS Coding Conventions](https://abibuilder.cs.uni-tuebingen.de/archive/openms/Documentation/nightly/html/coding_conventions.html)
* Read the [OpenMS User Tutorial](/tutorials/knime-user-tutorial.md).
* Create a GitHub account.
* Subscribe to the [open-ms-general](https://sourceforge.net/projects/open-ms/lists/open-ms-general) 
  or [contact-us](/about/communication.md).

### I have written a class for OpenMS. What should I do?

Follow the [OpenMS coding conventions](https://abibuilder.cs.uni-tuebingen.de/archive/openms/Documentation/nightly/html/coding_conventions.html).

Coding style (brackets, variable names, etc.) must conform to the conventions.

* The class and all the members should be properly documented.
* Check your code with the tool  `tools/checker.php`. Call `php tools/checker.php` for detailed instructions.

Please open a pull request and follow the [pull request guidelines](/manual/contribute/pull-request-checklist.md).

### Can I use QT designer to create GUI widgets?

Yes. Create a class called `Widget: Create .ui-File` with `QT designer` and store it as `Widget.ui.`, add the class to
`sources.cmake`. From the .ui-File the file `include/OpenMS/VISUAL/UIC/ClassTemplate.h` is generated by the build 
system.

```{note}
Do not check in this file, as it is generated automatically when needed.
```

Derive the class `Widget` from `WidgetTemplate`. For further details, see the `Widget.h` and `Widget.cpp` files.

### Can the START_SECTION-macro not handle template methods that have two or more arguments?

Insert round brackets around the method declaration.

### Where can I find the binary installers created?

View the binary installers at the [build archive](https://abibuilder.cs.uni-tuebingen.de/archive/openms/OpenMSInstaller/nightly/).
Please verify the creation date of the individual installers, as there may have been an error while creating 
the installer.

## Troubleshooting

The following section provides information about how to troubleshoot common OpenMS issues.

### OpenMS complains about boost not being found but I'm sure its there

`CMake` got confused. Set up a new build directory and try again. If you build from source (not recommended), deleting
the `CMakeCache.txt` and `cmake` directory might help.

## Build System

The following questions are related to the build system.

### What is CMake?

`CMake` builds BuildSystems for different platforms, e.g. VisualStudio Solutions on Windows, Makefiles on Linux etc.
This allows to define in one central location (namely `CMakeLists.txt`) how OpenMS is build and have the platform 
specific stuff handled by `CMake`.

View the [cmake website](http://www.cmake.org) for more information.

### How do I use CMake?

See Installation instructions for your platform.
In general, call `CMake(.exe)` with some parameters to create the native build-system.

```{tip}
whenever `ccmake` is mentioned in this document, substitute this by `CMake-GUI` if your OS is Windows. Edit the
`CMakeCache.txt` file directly.
```

### How do I generate a build-system for Eclipse, KDevelop, CodeBlocks etc?

Type `cmake` into a console. This will list the available code generators available on your platform; use them with
`CMake` using the `-G` option.

### What are user definable CMake cache variables?

They allow the user to pass options to `CMake` which will influence the build system. The most important option which
should be given when calling `CMake.exe` is:

`CMAKE_FIND_ROOT_PATH`, which is where `CMake` will search for additional libraries if they are not found in the default
system paths. By default we add `OpenMS/contrib`.

If you have installed all libraries on your system already, there is no need to change `CMAKE_FIND_ROOT_PATH`. For
`contrib` libraries, set the variable `CMAKE_FIND_ROOT_PATH`.

On Windows, `contrib` folder is required, as there are no system developer packages. To pass this variable to
`CMake` use the `-D` switch e.g. `cmake -D CMAKE_FIND_ROOT_PATH:PATH="D:\\somepath\\contrib"`.

Everything else can be edited using `ccmake` afterwards.

The following options are of interest:

- `CMAKE_BUILD_TYPE` To build Debug or Release version of OpenMS. Release is the default.
- `CMAKE_FIND_ROOT_PATH` The path to the `contrib` libraries.
   ```{tip}
    Provide more then one value here (e.g., `-D CMAKE_FIND_ROOT_PATH="/path/to/contrib;/usr/"` will search in your
    `contrib` path and in `/usr` for the required libraries)
   ```
- `STL_DEBUG` Enables STL debug mode.
-  `DB_TEST` (deprecated) Enables database testing.
-  `QT_DB_PLUGIN` (deprecated) Defines the db plugin used by Qt.

View the description for each option by calling `ccmake`.

### Can I use another solver other than GLPK?

Other solvers can be used, but by default, the build system only links to GLPK (this is how OpenMS binary packages must
be built). To to use another solver, use `cmake ... -D USE_COINOR=1 ....` and refer to the documentation of the
`LPWrapper` class.

### How do I switch to debug or release configuration?

For Makefile generators (typically on Linux), set the `CMAKE_BUILD_TYPE` variable to either Debug or Release by
calling `ccmake`. For Visual Studio, this is not necessary as all configurations are generated and choose the one you
like within the IDE itself. The 'Debug' configuration enabled debug information. The 'Release' configuration disables
debug information and enables optimisation.

### I changed the `contrib` path, but re-running `CMake` won't change the library paths?

Once a library is found and its location is stored in a cache variable, it will only be searched again if the
corresponding entry in the cache file is set to false.

```{warning}
If you delete the `CMakeCache.txt`, all other custom settings will be lost.
```

The most useful targets will be shown to you by calling the targets target, i.e. make targets.

### `CMake` can't seem to find a `Qt` library (usually `QtCore`). What now?

`CMake` finds `QT` by looking for `qmake` in your `PATH` or for the Environment Variable `QTDIR`. Set these accordingly.

Make sure there is no second installation of Qt (especially the MinGW version) in your local environment.
```{warning}
This might lead ``CMake`` to the wrong path (it's searching for the ``Qt*.lib`` files).
You should only move or delete the offending `Qt` version if you know what you are doing!
```

A save workaround is to edit the `CMakeCache` file (e.g. via `ccmake`) and set all paths relating to `QT`
(e.g. `QT_LIBRARY_DIR`) manually.

### (Windows) What version of Visual Studio should I use?

It is recommended to use the latest version. Get the latest `CMake`, as its generator needs to support your VS. If
your VS is too new and there is no `CMake` for that yet, you're gonna be faced with a lot of conversion issues.
This happens whenever the Build-System calls `CMake` (which can be quite often, e.g., after changes to `CMakeLists.txt`).

### How do I add a new class to the build system?

1. Create the new class in the corresponding sub-folder of the sub-project. The header has to be created 
  in `src/<sub-project>/include/OpenMS` and the `.cpp` file in `src/<sub-project>/source`, 
  e.g., `src/openms/include/OpenMS/FORMAT/NewFileFormat.h` and `src/openms/source/FORMAT/NewFileFormat.cpp`.
2. Add both to the respective `sources.cmake` file in the same directory (e.g., `src/openms/source/FORMAT/` 
  and `src/openms/include/OpenMS/FORMAT/`).
3. Add the corresponding class test to `src/tests/class_tests/<sub-project>/` 
  (e.g., `src/tests/class_tests/openms/source/NewFileFormat_test.cpp`).
4. Add the test to the `executables.cmake` file in the test folder 
  (e.g., `src/tests/class_tests/openms/executables.cmake`).
5. Add them to git by using the command `git add`.

### How do I add a new directory to the build system?

1. Create two new `sources.cmake` files (one for `src/<sub-project>/include/OpenMS/MYDIR`, 
  one for `src/<sub-project>/source/MYDIR`), using existing `sources.cmake` files as template.
2. Add the new `sources.cmake` files to `src/<sub-project>/includes.cmake`
3. If you created a new directory directly under `src/openms/source`, then have a look 
  at `src/tests/class_tests/openms/executables.cmake`.
4. Add a new section that makes the unit testing system aware of the new (upcoming) tests.
5. Look at the very bottom and augment `TEST_executables`.
6. Add a new group target to `src/tests/class_tests/openms/CMakeLists.txt`.

### How can I speed up the compile process of OpenMS?

To speed up the compile process of OpenMS, use several threads. If you have several processors/cores, build OpenMS
classes/tests and `TOPP` tools in several threads. On Linux, use the `make option -j: make -j8 OpenMS TOPP test_build`.

On Windows, Visual Studio solution files are automatically build with the `/MP` flag, such that Visual Studio uses all
available cores of the machine.

## Release

View [preparation of a new OpenMS release](https://github.com/OpenMS/OpenMS/wiki/Preparation-of-a-new-OpenMS-release#release_developer) 
to learn more about contributing to releases.


## Working in Integrated Development Environments (IDEs)

### Why are there no `source/TEST` and `source/APPLICATIONS/TOPP` folder?

All source files added to an IDE are associated with their targets. Find the source files for each test within
its own subproject. The same is true for the `TOPP` classes.

### I'm getting the error "Error C2471: cannot update program database"

This is a bug in Visual Studio and there is a [bug fix](https://support.microsoft.com/en-us/topic/fix-error-message-when-you-compile-a-visual-c-2008-project-error-c2471-cannot-update-program-database-339dde4f-904c-6dfa-51cd-090674ece765) Only apply it if you
encounter the error. The bug fix might have unwanted side effects!

### Visual Studio can't read the clang-format file.

Depending on the Visual Studio version it might get an error like `Error while formating with ClangFormat`.
This is because Visual Studio is using an outdated version of clang-format. Unfortunately there is no easy way to update
this using Visual Studio itself. There is a plugin provided by LLVM designed to fix this problem, but the plugin doesn't
work with every Visual Studio version. In that case, update clang-format manually using the pre-build clang-format
binary. Both the binary and a link to the plugin can be found [here](https://llvm.org/builds/).
To update clang-format download the binary and exchange it with the clang-format binary in your Visual Studio folder.
For Visual Studio 17 and 19 it should be located at: `C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\Llvm\bin`.

### The indexer gets stuck at some file which ``#includes seqan``

It seems that SeqAn code is just too confusing for older eclipse C++ indexers. You should upgrade to eclipse galileo
(CDT 6.0.x). Also, increase the available memory limit in `eclipse.ini`, e.g. `-Xmx1024m` for one gig.

### The parser is confused after OPENMS_DLLAPI and does not recognize standard C++ headers

Go to ``Project -> Properties -> C/C++ Include Paths and Preprocessor Symbols -> Add Preprocessor symbol -> "OPENMS_DLLAPI="``.
This tells eclipse that the macro is defined empty. In the same dialog add an external include path to
e.g. ``/usr/include/c++/4.3.3/``, etc. The issue with C++ headers was fixed in the latest galileo release.

Hints to resolve the OPENMS_DLLAPI issue using the ``cmake`` generator are welcome!

## Debugging

The following section provides information about how to debug your code.

### How do I run a single test?

Execute an OpenMS class test using the CTest regular expressions:

```bash

$ ctest -V -R "^<class>_test"

# To build a class test, call the respective make target in ./source/TEST:

$ make <class>_test
```
To run a TOPP test, use:

```bash

$ ctest -V -R "TOPP_<tool>"
```

To build the tool, use:

```bash
$ make <tool>
```
### How do I debug uncaught exceptions?

Dump a core if an uncaught exception occurs, by setting the environment variable `OPENMS_DUMP_CORE`.

Each time an uncaught exception occurs, the `OPENMS_DUMP_CORE` variable is checked and a segmentation fault is caused,
if it is set.

### (Linux) Why is no core dumped, although a fatal error occured?

The `ulimit -c` unlimited command. It sets the maximum size of a core to unlimited.

```{warning}
We observed that, on some systems, no core is dumped even if the size of the core file is set to unlimited. We are not
sure what causes this problem.
```

### (Linux) How can I set breakpoints in gdb to debug OpenMS?

Imagine you want to debug the TOPPView application and you want it to stop at line 341 of SpectrumMDIWindow.C.

1. Enter the following in your terminal:

  ```bash
  Run gdb:
 shell> gdb TOPPView
```

2. Start the application (and close it):

  ```bash
 gdb> run [arguments]
```
3. Set the breakpoint:
  ```bash
 gdb> break SpectrumMDIWindow.C:341
```
4. Start the application again (with the same arguments):

  ```bash
 gdb> run
 ```

### How can I find out which shared libraries are used by an application?

Linux: Use `ldd`.

Windows (Visual studio console): See [Dependency Walker](http://www.dependencywalker.com/) (use x86 for 32 bit builds
and the x64 version for 64bit builds. Using the wrong version of depends.exe will give the wrong results) or
``dumpbin /DEPENDENTS OpenMS.dll``.

### How can I get a list of the symbols defined in a (shared) library or object file?

Linux: Use `nm <library>`.

Use `nm -C` to switch on demangling of low-level symbols into their C++-equivalent names. `nm` also accepts .a and .o 
files.

Windows (Visual studio console): Use ``dumpbin /ALL <library>``.

Use dumpbin on object files (.o) or (shared) library files (.lib) or the DLL itself e.g. `dumpbin /EXPORTS OpenMS.dll`.

## Cross-platform thoughts

OpenMS runs on three major platforms.. Here are the most prominent causes of "it runs on Platform A, but not on B. 
What now?"

### Reading or writing binary files

Reading or writing binary files causes different behaviour. Usually Linux does not make a difference between text-mode
and binary-mode when reading files. This is quite different on Windows as some bytes are interpreted as `EOF`, which 
lead might to a premature end of the reading process.

If reading binary files, make sure that you explicitly state that the file is binary when opening it.

During writing in text-mode on Windows a line-break (`\n`) is expanded to (`\r\n`). Keep this in mind or use the
`eol-style` property of subversion to ensure that line endings are correctly checked out on non-Windows systems.

### Paths and system functions

Avoid hardcoding e.g.`String tmp_dir = "/tmp";`. This will fail on Windows. Use Qt's `QDir` to get a path to the systems
temporary directory if required.

Avoid names like uname which are only available on Linux.

When working with files or directories, it is usually safe to use "/" on all platforms. Take care of spaces in directory
names though. Quote paths if they are used in a system call to ensure that the subsequent interpreter
takes the spaced path as a single entity.


## Doxygen Documentation

### Where can I find the definition of the main page?

Find a definition of the main page [here](https://github.com/OpenMS/OpenMS/edit/develop/doc/doxygen/public/Main.doxygen).

### Where can I add a new module?

Add a new module [here](https://github.com/OpenMS/OpenMS/edit/develop/doc/doxygen/public/Modules.doxygen).

### How is the parameter documentation for classes derived from DefaultParamHandler created?

Add your class to the program ``OpenMS/doc/doxygen/parameters/DefaultParamHandlerDocumenter.cpp``. This program 
generates a html table with the parameters. This table can then be included in the class documentation using the 
following `doxygen` command:`@htmlinclude OpenMS_<class name>.parameters`.

```{note}
Parameter documentation is automatically generated for `TOPP` included in the static `ToolHandler.cpp` tools list.
```

To include TOPP parameter documentation use following `doxygen` command:

`@htmlinclude TOPP_<tool name>.parameters`

Test if everything worked by calling `make doc_param_internal`. The parameters documentation is written to
`OpenMS/doc/doxygen/parameters/output/`.

### How is the command line documentation for TOPP tools created?

The program `OpenMS/doc/doxygen/parameters/TOPPDocumenter.cpp` creates the command line documentation for all classes
that are included in the static `ToolHandler.cpp` tools list. It can be included in the documentation using the 
following `doxygen` command:

`@verbinclude TOPP_<tool name>.cli`

Test if everything worked by calling `make doc_param_internal`. The command line documentation is written to
`OpenMS/doc/doxygen/parameters/output/`.

## Bug Fixes

### How to contribute a bug fix?

Read [contributor quickstart guide](/manual/contribute.md).

### How can I profile my code?

IBM's profiler, available for all platforms (and free for academic use): Purify(Plus) and/or Quantify.

Windows: this is directly supported by Visual Studio (Depending on the edition: Team and above). Follow their 
documentation.

Linux:

1. Build OpenMS in debug mode (set `CMAKE_BUILD_TYPE` to `Debug`).
2. Call the executable with valgrind: `valgrind –tool=callgrind`.
   ```{warning}
   Other processes running on the same machine can influence the profiling. Make sure your application gets enough
   resources (memory, CPU time).
   ```
3. Start and stop the profiling while the executable is running e.g. to skip initialization steps:
4. Start valgrind with the option `–instr-atstart=no`.
5. Call `callgrind -i [on|off]` to start/stop the profiling.
6. The output can be viewed with `kcachegrind callgrind.out`.

### (Linux) How do I check my code for memory leaks?

* Build OpenMS in debug mode (set ``CMAKE_BUILD_TYPE`` to ``Debug``).
* Call the executable with ``valgrind: valgrind --suppressions=OpenMS/tools/valgrind/openms_external.supp –leak-check=full <executable> <parameters>``.

Common errors are:

* ``'Invalid write/read ...'`` - Violation of container boundaries.
* ``'... depends on uninitialized variable'`` - Uninitialized variables:
* ``'... definitely lost'`` - Memory leak that has to be fixed
* ``'... possibly lost'`` - Possible memory leak, so have a look at the code

For more information see the [`valgrind` documentation](http://valgrind.org/docs/manual/) .