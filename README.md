
Simple MTPFS for mac (and also linux)
=====================================

SIMPLE-MTPFS (Simple Media Transfer Protocol FileSystem) is a file system, capable of reading and writing data on an MTP device. **This can be used for transfering data between an android phone and a computer.** The original source code repo of SIMPLE-MTPFS can be found at [https://github.com/phatina/simple-mtpfs](). The current project is a fork of SIMPLE-MTPFS, with improvements, so that it works properly on macOS (tested on macOS 14). Files can be viewed in the Finder app with no problems, offering a seamless experience. A GUI app is on the way.

To simplify the build process, I have changed from using autotools to using CMake. (I could not get autotools to work probably due to some issues with configuration.)

## Why using this instead of some other app?

Windows file explorer itself supports MTP. On macOS, we can use apps like android file transfer. However, many of these options are good enough, because they typically do not mount the android device as part of the filesystem tree. In practice this means that (1) you cannot operate on android files using programs written and compiled for the mac and (2) inside GUI for accessing android files, whether it is the UI of native file manager or another app, some context menu options will be missing.

With SIMPLE-MTPFS-MAC, you can **mount** your phone just like an USB drive, to enjoy the full set of file operations.

## Build and Install

Firstly, install the packages `libusb, libmtp, libfuse`. On macOS (if you want to compile on linux, install those package using whatever method avaliable), to get `libfuse`, install macFUSE with `brew`. This can be done with

```shell
brew install libusb libmtp macfuse
```

We of course also need a C++ 17 compiler and CMake. We can do

```C
brew install llvm cmake
```

For macFUSE, you may need to do some configurations on allowing kernel extensions. Follow the instructions on the macFUSE website. 

Then, please figure out the installation path of the libraries above and the corresponding header paths, and set those paths manually in `CMakeLists.txt`, by setting the value of variables. For instance, change `set(LIBUSB_LIBRARIES)` to `set(LIBUSB_LIBRARIES /path/to/library)`. I would really welcome any suggestions on how to make CMake find those automatically - but at the moment that's what I have got.

Then we are ready to run the usual thing in the terminal

```shell
mkdir build
cd build
cmake ..
cmake --build .
```

Then copy the executable produced to anywhere you want.

It should also be easy to port the program to windows, with winfsp and msys2. However, you might need to modify some codes.

## Use

Run

```
mtpfs -f <mountpoint>
```

to mount the android device currently connected to the computer, assuming there is only one device connected.

Please do not omit the `-f` option; otherwise it does not work. I haven't yet debugged this issue.

## How it works

I will explain some of the wired details behind this.

The original project https://github.com/phatina/simple-mtpfs probably compiles on macOS with some modifications to the build scripts. However, the binary produced does not work well because some system calls are not implemented. When trying to copy files into the android device in finder, finder will raise error or even crash. Sometimes you need to hard reboot by pressing the power button.

Thus, I have added vacuous implementations of some system calls. For anyone else who are writing a FUSE driver for macOS, this is something to keep in mind.

Another thing to note is that macOS on apple silicon chips run with memory page size 16KB. If some older versions of libraries assume a 4KB page size, it will not work properly.

## Changes to the original repo

1. All the files related to autotools are removed. This includes all files in the root directory of the project except for `config.h`. (`config.h` is now not auto-generated.) The following files and directories are removed from the original repo:
   ```
   NEWS              ChangeLog        debian
   makefile.am
   autogen.sh        man
   INSTALL           configure.ac      simple-mtpfs.spec
   ```
2. The old README.md is replaced by a new one.
3. `src/simple-mtpfs-fuse.cpp` is modified to support more system calls.
4. `src/simple-mtpfs-ls.cpp` removed.
5. `CMakeLists.txt` added.
6. `.gitignore` changed to accommodate all the above modifications.

