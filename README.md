# IAR C-RUN Walkthrough
[IAR C-RUN](https://iar.com/crun) is a runtime analysis tool fully integrated into the IAR platform.
It automatically instruments your C and C++ code to help you quickly spot runtime errors, ensuring improved code quality for your applications.

This quick walkthrough will guide you through the runtime error checking capabilities offered by IAR C-RUN.
Refer to [C-RUN runtime error checking](https://docs.iar.com/ewarm/10.1x/en/c-spy-debugging/c-run-runtime-error-checking.html) for more information.




<!-- --------------------------------------------------------------------------------------------------- -->
## Software requirements
For evaluating C-RUN using this guide, fulfill the following requirements:
| __Product__                           | __Minimum version__ | __Recommended version__
| -                                     | -                   | -
| IAR Embedded Workbench for Arm        | 8.11.1              | 10.10.1 or later 
| IAR Embedded Workbench for Renesas RX | 5.10.1              | 5.20.1 or later
 
>[!NOTE]
>IAR C-RUN is included with the IAR Platform subscription. Alternatively, it is available for
>[evaluation](https://www.iar.com/embedded-development-tools/free-trials) of the supported products.


<!-- --------------------------------------------------------------------------------------------------- -->
## Exploring C-RUN
A number of example projects, created as demonstrations of the feature set offered by C-RUN, can be found in this repository.

To run the examples:

1. Clone this repository **-or-** download (and extract) its [.zip][url-repo-zip] archive.
2. Launch the IAR Embedded Workbench IDE.
3. Open the corresponding `Workspace.eww` file for the desired target architecture (`arm` **-or-** `rx`).
4. If offered, allow the IDE to upgrade the workspace projects.

![Upgrade projects](https://github.com/user-attachments/assets/301db0c4-d8da-4e16-950e-2700f94a022b)

This workspace contains 5 projects:

![Workspace-overview](https://github.com/user-attachments/assets/f7e5829d-5f53-452c-90d8-bbaa9369462e)

The following sections will present detailed instructions for each of these projects,
enabling you to take full control of the feature set offered by IAR C-RUN.




<!-- --------------------------------------------------------------------------------------------------- -->
## Arithmetic Checking
The first example explores the most straightforward functionality in C-RUN: the _Arithmetic_ checks.

To run this example, do this:

1. Make sure the _Arithmetic_ project is set as active.

2. Choose __Project__ → __Options__ (<kbd>Alt</kbd>+<kbd>F7</kbd>) → __Runtime Checking__ → __C-RUN__ and make sure it is enabled.

3. Make sure the following checks are enabled:

![crun-arith-checks-enabled](https://github.com/user-attachments/assets/707f3ea5-48c3-41be-897d-2280908796a7)

4. Click `  OK  ` to close the __Project Options__ dialog box.

5. Choose __Debug__ → __Download and Debug__ (<kbd>Ctrl</kbd>+<kbd>D</kbd>) to start executing the application.

6. The C-SPY Debugger will hit a breakpoint at `main()`. Press <kbd>F5</kbd> to resume the program execution.

>The execution will _Stop_, with the following line of code highlighted:
>
>![image](https://github.com/IARSystems/crun-walkthrough/assets/54443595/5737b5b0-2b25-4b67-9242-07a4e4bb0fa3)
>
>During the debug session, every time the application hits a statement causing a runtime error, C-RUN will halt the execution.
>
>![crun-messages-arith](https://github.com/user-attachments/assets/f9d31240-8a66-497e-82f5-702be97d7f55)
>
> In addition, the __C-RUN Messages__ window (__View__ → __C-RUN__ → __Messages__) will show detailed information about the root cause alongside the associated call stack.
> Clicking on the call stack will help you to navigate throughout the application's source code.

7. Press <kbd>F5</kbd> repeatedly to resume the program execution and examine the remaining detected C-RUN messages until the execution ends.

![crun-messages-arith-all](https://github.com/user-attachments/assets/4be56c89-35c4-4739-9584-1655f185caab)

8. Stop the debug session (<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>D</kbd>).


### Fine-tuning arithmetic checks
There are programing situations in which taking advantage of the wraparound property of overflown unsigned integers is beneficial and efficient.
In such cases, specific checks can simply be deselected.
Disabling the checks you do not need in your application will
generally reduce the code size used by the instrumentation and execute faster.

In the project's options, disable the following C-RUN checks:
- [ ] Including unsigned
- [ ] Including explicit casts
- [ ] Including unsigned shifts 

Rebuild the application, __Download and Debug__ (<kbd>Ctrl</kbd>+<kbd>D</kbd>), and inspect the changes in the error detection.


### Tweaking the rules
By default, C-RUN stops at each detected error. In the __C-RUN Messages__ window, the _Default action_ can be switched to __Log__ or to __Ignore__.

__C-RUN Messages__ can also be filtered by rules.
For details, refer to [Creating rules for messages](https://docs.iar.com/ewarm/10.1x/en/c-spy-debugging/c-run-runtime-error-checking/using-c-run/creating-rules-for-messages.html).




<!-- --------------------------------------------------------------------------------------------------- -->
## Bounds checking
The second example explores the _Bounds Checking_ capabilities provided by C-RUN.

To run this example, do this:

1. With a right-click, set the _Bounds-checking_ project as active.

![set-project-as-active](https://github.com/user-attachments/assets/8d566fec-4036-4d07-bd4a-8a680a768fd3)

2. Choose __Debug__ → __Download and Debug__ (<kbd>Ctrl</kbd>+<kbd>D</kbd>) to start executing the application.

3. After the execution reaches the breakpoint in the `main()` function, resume the execution (<kbd>F5</kbd>).
> Note how the application finishes executing (`exit()`) without any apparent errors of any kind.

4. Stop the debug session (<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>D</kbd>).

5. Choose __Project__ → __Options__ (<kbd>Alt</kbd>+<kbd>F7</kbd>) → __Runtime Checking__ → __C-RUN__ and _Enable bounds checking_:

![bounds-checking](https://github.com/user-attachments/assets/ebf2bdd6-bdc9-4575-8259-0b774241032d)

6. Rebuild and run the project.

![image](https://github.com/IARSystems/crun-walkthrough/assets/54443595/bfd4cb27-f942-4cc4-a690-418ae14b8cf5)

> Note that now C-RUN highlighted an out-of-bounds access for `*(ap+2)`. However, the execution stopped earlier, at the first
> `printf()` statement. The reason is that the compiler determines that pointer addresses used as parameters for those `printf()`
> calls are related and, with that, C-RUN efficiently can test them in one go.

7. Press <kbd>F5</kbd> to resume the execution and inspect the remaining C-RUN Message:

![image](https://github.com/user-attachments/assets/6b5b86ff-761a-4a04-b7bd-3a28664d65dd)

The program will stop at the last statement, indicating that the assignment is performed out of bounds. 
From a bounds-checking perspective, dynamically allocated memory is no different from local pointers, static buffers, or buffers on the stack.


### Optimizing the Global Bounds Table
When performing bounds checking, pointers accessed through other pointers must have information in a global bounds table stored in memory. The table size is defined by the __Number of entries__ field. 

>[!CAUTION]
>Make sure to set the __Number of entries__. Leaving this field empty will result in a table with 4000 entries.


>[!TIP]
>The number of entries you need to keep in the table is often fairly low, so you might want to experiment with shrinking
>it in your own project. Shrinking the table will result in reduced code size required by the C-RUN instrumentation
>in your application.
>Whenever the chosen __Number of entries__ is too low, C-RUN will warn you that "the global bounds table is running out of slots".

1. Choose __Project__ → __Options__ (<kbd>Alt</kbd>+<kbd>F7</kbd>) → __Runtime Checking__ → __C-RUN__.

2. Under __Bounds checking__ verify the __Number of entries__ field.

3. Set the __Number of entries__ to `1` and close the __Project Options__ dialog box.

4. Rebuild the aplication then __Download and Debug__ (<kbd>Ctrl</kbd>+<kbd>D</kbd>).

Since there is only one pointer in this example, a single entry in the global bounds table is enough.




<!-- --------------------------------------------------------------------------------------------------- -->
## Bounds checking and libraries
In scenarios where the main project relies on pre-built (third-party) libraries, _bounds checking_ requires extra consideration for shared pointers.

The project _Bounds-checking+libs_ is a modified version of the _Bounds-checking_ project.
The functionality available from the `IntMax.c` file was moved to a _Library project_ that produced a static library named `MaxLib.a`,
pre-built with no C-RUN bounds-checking instrumentation, typical for third-party libraries where the code cannot be changed. 

When you use pointers and pointer arguments between the _instrumented application code_ and the _non-instrumented library code_,
you must inform the compiler (and linker) that functions in the library code do not have bounds-checking information. 
The best way to do so is to use the `#pragma default_no_bounds` directive when including a library header, 
to inform the compiler that the library was not built with C-RUN Bounds-checking information:

![image](https://github.com/IARSystems/crun-walkthrough/assets/54443595/788c9bba-3610-4197-a9c5-43b7beb01e0c)


### Building without pointer checking from non-instrumented code
In most cases, returned pointers will not be equipped with bounds-checking instrumentation but 
will have associated bounds that are always "large enough" to accommodate them. 
In other words, this means that pointers originating from your code will be checked, 
but not pointers from the library. 
Under normal circunstances this should be perfectly acceptable. 
The process is non-intrusive in terms of code changes.

To run this example's build configuration, do this:

1. Set the  _Bounds-checking+libs_ project as active.
2. Use the Workspace selector and choose the "DoNotCheckPointersFromNonInstrumentedCode" build configuration.

<img width="547" height="213" alt="image" src="https://github.com/user-attachments/assets/dcc7f004-6f91-456d-90dd-6c5732b6fc89" />

3. Choose __Project__ → __Options__ (<kbd>Alt</kbd>+<kbd>F7</kbd>) → __Runtime Checking__ → __C-RUN__ and confirm these settings:

![image](https://github.com/user-attachments/assets/ff0bf99e-bdbb-4592-8d13-ed27cf5be58c)

4. Close the __Project Options__ dialog box, and choose __Debug__ → __Download and Debug__ (<kbd>Ctrl</kbd>+<kbd>D</kbd>) to start executing the application.

You should get no C-RUN errors, or any other indications that something is wrong.
In the `DoNotCheckPointersFromNonInstrumentedCode` build configuration, turning off bounds-checking information for library headers worked well.

5. Stop the debug session (<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>D</kbd>).


### Building with pointer checking from non-instrumented code
The `CheckPointersFromNonInstrumentedCode` build configuration demonstrates a situation where 
it is desirable to have _bounds checking_ for pointers and returned pointers defined in the library code.

To run this example's build configuration, do this:

1. Make sure the  _Bounds-checking+libs_ project is set as active.
2. Switch to the `CheckPointersFromNonInstrumentedCode` build configuration.

<img width="547" height="213" alt="image" src="https://github.com/user-attachments/assets/91c0f719-f48f-4684-b3a2-e0d92df480ba" />

3. Choose __Project__ → __Options__ (<kbd>Alt</kbd>+<kbd>F7</kbd>) → __Runtime Checking__ → __C-RUN__ and confirm these settings:

<img width="519" height="279" alt="image" src="https://github.com/user-attachments/assets/0a435fe7-5218-45e8-9e23-052b504be0c0" />

4. Build the application and run it again. This time you should see one C-RUN message for the third `printf()` statement.
>Read the source code and compare the use of `__as_make_bounds()` for the pointer `ap` to how the bounds are set for the pointer used in the next `printf()` statement.
>We have also defined a project-specific macro to control the use of `__as_make_bounds()`.
>This built-in function is used when the returned pointer must be given sensible bounds.
>If no bounds are given for returned pointers, a bounds error will be generated on the first access made to the pointer.
>Note that this build configuration defines a preprocessor symbol called `CONFIG` that conditionally compiles with `__as_make_bounds()` calls to give pointers sensible bounds.

5. Comment out one of the calls to the `CRUN_MAKE_BOUNDS()` macro, rebuild and run.
>:grey_question: Did it change the output in the __C-RUN Messages__ window?




<!-- --------------------------------------------------------------------------------------------------- -->
## Heap checking capabilities
C-RUN can check for errors in how heap memory is being used. 
_Heap checking_ can catch when the application tries to use already freed memory, non-matching deallocation attempts, and leaked heap blocks.

When you use _heap checking_, each memory block is expanded with bookkeeping information and a buffer area,
so that the blocks as seen by the application are not located side-by-side.

The various checker functions examine the bookkeeping information and the buffer areas and report violations of correct heap usage.

To run this example, do this:

1. Set the _Heap_ project as active.

2. Choose __Project__ → __Options__ (<kbd>Alt</kbd>+<kbd>F7</kbd>) → __Runtime Checking__ → __C-RUN__. 

3. Make sure that _Use checked heap_ is enabled:

<img width="798" height="145" alt="image" src="https://github.com/user-attachments/assets/bda9935a-64e8-48b2-83b4-b852b89f52be" />

4. Choose __Debug__ → __Download and Debug__ (<kbd>Ctrl</kbd>+<kbd>D</kbd>) to start executing the application.

5. Examine the source code comments for each reported error.

<img width="828" height="320" alt="image" src="https://github.com/user-attachments/assets/c81d4a9e-38a1-49dc-aae6-4e6405ae590a" />

6. Stop the debug session (<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>D</kbd>).

>[!NOTE]
> - Using _heap checking_ makes it much easier to find heap usage errors, but it is **not** fail-safe.
> - _Heap checking_ and _bounds checking_ can complement each other in identifying dynamic memory usage errors. However, because of the potential impact in terms of performance and overhead, you are advised **not** to enable both at the same time.
> - The function `HeapFunc3()` in `Heap.c` can be enabled by uncommenting the `CRUN_FULL_EDITION` macro definition. Including the function exceeds the code size limitation in the trial mode. For unlocking such a limitation, [contact us](https://www.iar.com/about/contact).




<!-- --------------------------------------------------------------------------------------------------- -->
## Using C-RUN in non-interactive mode
There are scenarios in which might not be possible to debug an application built with C-RUN information directly from the IDE.
Below you will find some examples on how to use C-RUN in non-interactive mode.


### C-RUN Runtime Analysis from the command line
The IAR C-SPY Command Line Utility (`CSpyBat`) can run applications directly from the command line.
This utility is suitable for non-interactive debugging and can be used in conjunction with IAR C-RUN where runtime analysis is desired.
One typical scenario for considering `CSpyBat` is within automated tests from a continuous integration environment.

1. Close the IDE, launch a terminal and change to the project directory (e.g., arm):
```
cd /path/to/crun-walkthrough/arm
```

>[!TIP]
> The IDE automatically generates scripts for running `CSpyBat` under the `settings/` sub-directory. In the latest IDE version, the relevant files are:
>
> | File                                               | Description
> | -                                                  | -
> | `launch.json`                                      | JSON file with launch configurations for all projects in the workspace.
> | `<project>.<cfg>.cspy_launch_json.{bat\|ps1\|sh}`  | Scripts for launching `CSpyBat` from the corresponding terminal/shell.
>
> These scripts inherits your project settings. They are ready to run with C-RUN from the command line.

2. Execute the script corresponding to your terminal. For example, on Linux Bash, execute:
```console
$ ./settings/Heap.Debug.cspy_launch_json.sh

     IAR C-SPY Command Line Utility V9.5.3.2051
     Copyright 2026 IAR Systems AB.

Heap usage error @ 0x2222 Core: 0
The address 0x20001249 does not appear to be the start of a heap block.
Call Stack:
    HeapFunc1 in "/home/user/crun-walkthrough/common/Heap.c", 32:3 - 32:10
    main in "/home/user/crun-walkthrough/common/Heap.c", 101:3 - 101:13
    [_call_main + 0xd]
Heap usage error @ 0x2228 Core: 0
The address 0x20001208 does not appear to be the start of a heap block.
Call Stack:
    HeapFunc1 in "/home/user/crun-walkthrough/common/Heap.c", 34:3 - 34:10
    main in "/home/user/crun-walkthrough/common/Heap.c", 101:3 - 101:13
    [_call_main + 0xd]
Heap usage error @ 0x222e Core: 0
The address 0x20001249 does not appear to be the start of a heap block.
Call Stack:
    HeapFunc1 in "/home/user/crun-walkthrough/common/Heap.c", 35:1 - 35:1
    main in "/home/user/crun-walkthrough/common/Heap.c", 101:3 - 101:13
    [_call_main + 0xd]
Out of heap space @ 0x223a Core: 0
There is no more heap space to allocate.
A request for 4120 bytes could not be satisfied.
A total of 36 bytes have been allocated in 1 heap blocks.
Call Stack:
    HeapFunc2 in "/home/user/crun-walkthrough/common/Heap.c", 44:3 - 44:9
    main in "/home/user/crun-walkthrough/common/Heap.c", 102:3 - 102:13
    [_call_main + 0xd]

     CSpyBat terminating.
```



## Issues
For technical support contact [IAR Customer Support][url-iar-customer-support].

For questions or suggestions related to this tutorial: try the [wiki][url-repo-wiki] or check [earlier issues][url-repo-issue-old].
If those don't help, create a [new issue][url-repo-issue-new] with detailed information.


<!-- Links -->
[url-iar-customer-support]: https://iar.my.site.com/mypages/s/contactsupport

[url-repo]:                https://github.com/iarsystems/crun-walkthrough
[url-repo-zip]:            https://github.com/iarsystems/crun-walkthrough/archive/refs/heads/master.zip
[url-repo-wiki]:           https://github.com/iarsystems/crun-walkthrough/wiki
[url-repo-issue-new]:      https://github.com/iarsystems/crun-walkthrough/issues/new
[url-repo-issue-old]:      https://github.com/iarsystems/crun-walkthrough/issues?q=is%3Aissue+is%3Aopen%7Cclosed
