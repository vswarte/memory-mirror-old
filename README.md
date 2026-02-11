# MemoryMirror

> [!CAUTION]
> This tool was moved [here](https://github.com/vswarte/memory-mirror).

## What is this?
This tool allows you to grab a running process and dump its memory such that it can (mostly) immediately be imported
into Ghidra for analysis.

## How do I use this?

### Dumping memory
You'll want to invoke memory_mirror like so:
```shell
$ ./memory_mirror.exe pid <pid> -p <output directory>  
```
or
```shell
$ ./memory_mirror.exe name some_program.exe -p <output directory>  
```

This will write dumped memory to `<output directory>`.
In the case of dumping by name, any process with that name will be dumped into a folder with the process ID.

### Importing the modules into Ghidra
Then you can import any concatted binaries into Ghidra. Run your analysis after doing this as running analysis with a
heap dump might cause trouble and will be very slow. You can only supply one executable/dll as a program to Ghidra but
you can import others (like additional DLLs) using File > Add to Program in Ghidra.
**Make sure to specify the base address for an imported memory region!**

### Importing heap data into Ghidra
Once analysis is all done on the modules you can once more use Ghidra's File > Add to Program to import any heap data.
**Make sure to specify the base address for an imported memory region!**
If the bases are specified properly you should now see that pointers to heap data start actually pointing somewhere :-)

## How does it work under the hood?
When you invoke this utility, it enumerates all process threads, freezes them, dumps all memory regions that are
not in a `MEM_FREE` state, corrects the `PointerToRawData` and `RawDataSize` fields on found section headers such that
Ghidra can use them to locate the sections again, and finally unfreezes the threads again.

## Why would I ever want to have a runtime-dumped memory image in Ghidra?
Having the .data section filled-in and having the heap data in the repo can be very useful if you're looking to reverse
big, complex, structures.

## How to use this for MAX DUMPING CAPABILITIES?
There are a few things you can do to improve both your dumps and your analysis. Namely:

### Turn off ASLR on the program you're dumping
ASLR randomizes the base address for a given executable. You probably want it turned on for security reasons but it also
makes conversion between address and RVA becomes tedious.
You can disable ASLR by disabling the `IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE` bit from 
`PEHeader.FileHeader.OptionalHeader.DllCharacteristics` in the on-disk executable's PE header.

### Order matters
In my experience it works best to do the dump, import just the memory image of your software and once that's done you
import the heap data. This is to prevent Ghidra from taking ages on the analysis but I also feel like running an
analysis on the heap causes Ghidra to goof on the analysis here and there.
