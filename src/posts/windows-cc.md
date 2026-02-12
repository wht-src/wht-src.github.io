---
title: "Compiling C/C++ on Windows Made Easy"
date: 2026-01-19
tags: 
  - posts
layout: post.njk
---

You know how it is, compiling c++ or c codebase on Windows 
is somewhat like setting your hair on fire. It doesn't have to be like that.

Note that all the commands in this guide should be ran in `powershell`. Press 
the `Windows` button on your keyboard to open up search menu, and search 
`powershell`, then open it.

## Install

A package manager will automatically install the things for us. In this case,
let's use the scoop package manager. Open powershell and run these two commands.
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

After it is done, let's install the compiler. We have 2 compiler options, both 
can do basically the same thing, so you would only need to run one of the commands.
For beginners, i recommend only installing `gcc`. You can also install both.
```powershell
# install gcc, which gives you the command gcc to compile c code, and g++ to compile c++ code
scoop install mingw 

# install clang, which gives you the command clang to compile c code, and clang++ to compile c++ code
scoop install llvm
```

## LLVM Instructions
Skip this step if you have not run `scoop install llvm`.

For `clang`, you will need to install more things. Go [here](https://visualstudio.microsoft.com/downloads/)
then scroll down to the `All Downloads` section. Click `Tools for Visual Studio`.
Download `Build Tools for Visual Studio 2026`.

When installing the build tools, make sure to select `Desktop development with C++`, 
after that, click `Install`. Wait for it to finish. After that, reboot your 
computer just to make sure, and you should be golden.

## Testing the Installations 
Just to make sure, let's see if it works:
```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "goodbye world" << endl;
    return 0;
}
```
Save it as `main.cpp` and figure out the location where you saved it. In 
my case it is `C:\Users\User\Documents\Projects\ctest\main.cpp`.
> for notepad users, when saving, you should select the option `All files (*.*)`
    for `Save as type` or else your code will be saved as `main.cpp.txt`, which 
    will probably confuse your compiler

Now, go to the folder where the code is saved.
```powershell
cd C:\Users\User\Documents\Projects\ctest
```

After that, run either one of the command depending on which compiler 
you have installed:
```powershell
# if you installed clang / llvm
clang++ main.cpp 

# if you installed gcc / mingw
g++ main.cpp # gcc
```
If nothing went wrong, then these two commands should print absolutely **nothing**.
They will only output something when there are errors.

The compiler should generate an `a.exe` file in the directory.
Run `ls` to see what files are in the directory.
```powershell
ls 
# main.cpp 
# a.out
```

Finally, run em:
```powershell
.\a.exe
# goodbye world
```

Congrats, you have now installed a C/C++ compiler in Windows. 
