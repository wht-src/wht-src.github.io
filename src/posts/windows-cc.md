---
title: "Compiling C/C++ on Windows Made Easy"
date: 2026-01-19
tags: 
  - posts
layout: post.njk
---

You know how it is, installing something like `clang++` or `gcc` on Windows 
is somewhat like setting your hair on fire. It doesn't have to be like that.

Note that all the commands in this guide should be ran in `powershell`.

## Install

A package manager will automatically install the things for us. In this case,
let's use the scoop package manager. Open powershell and run these two commands.
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

After it is done, let's install the compiler. For starters, I recommend only 
getting `gcc/mingw`.
```powershell
scoop install mingw # gcc 
scoop install llvm # clang
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

Now, go to the folder where the code is saved, and compile.
```powershell
cd C:\Users\User\Documents\Projects\ctest

clang++ main.cpp # clang
g++ main.cpp # gcc
```

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
