# Adding to OpenSees

This document includes instructions and best practices for adding new classes (e.g., materials, elements) to the OpenSees framework, and adding these changes to the OpenSees repository hosted on github.
The target reader: is aware of OpenSees, does not necessarily know a lot about git, and is willing to google things they don't know.

## Preliminaries

### Reading

Read these articles before getting started to understand the OpenSees design:
* 10.1061/(ASCE)0733-9445(2008)134:4(562)
* 10.1061/(ASCE)CP.1943-5487.0000002

### OpenSees is hosted on github...

... this means that you need to have some knowledge of `git` (the software behind github) and github itself (which is the website that hosts the OpenSees git repository).
Unfortunately, the less you know the more you will have to google.
There is jargon in this document (not only regarding git) - this is on purpose so that you can google and get accurate results to questions you may have (e.g., "add existing item to visual studio project").
* I cannot explain everything about git and github - there are thousands of guides, tutorials, and stackexchange posts that explain things.
* I will provide some basic commands and some basic ideas, however, you will probably need to google several things along the way unless you already have this knowledge.
* I provide the command-line commands for using git, you will need to install `git` itself if it is not installed on your system.
* There are tools for git integrated into most editors that you can use instead of the command line - you can explore these instead of the commands provided in this document.

### Windows: Visual Studio

I recommend to use the edition of Microsoft Visual Studio (VS) Community that corresponds to the OpenSees codebase.
At the time of writing this is VS 2019, and it can change without any major notice as far as I know, you can figure it out by the build tool version.
If you have an older version you just need to change the build tool version for each project in VS.
I recommend that you use VS as an editor and to build OpenSees.

The Win64 solution is found in the directory: `OpenSees/Win64/OpenSees.sln`.
This file can be opened in VS.

### Mac and Linux

I recommend using Visual Studio Code as en editor on Mac and Linux, although this is more about personal preference, rather than necessity (as in the case of VS on Windows).
Xcode, vim, emacs, Notepad++, etc., are fine too.
Generally, it is more difficult to build OpenSees on a Mac/Linux than Windows the first time since there are a lot more system dependent includes.
My advice would be to start with Windows if you have the choice.


## Guide

### Step 1. Fork the main OpenSees repository and create a clone

#### Create a fork

A "fork" is your own personal copy of an existing repository, forks are necessary because you do not have permission to edit the main OpenSees repository.
You can only suggest edits to make to the main repository, and those with permission will decide whether or not to incorporate your changes.
In the case of OpenSees, those people are Frank McKenna and Michael Scott.

To create your OpenSees fork you will need a github account.
After you have an account go to the OpenSees repository https://github.com/OpenSees/OpenSees and create a fork.
For the purpose of this document the fork will exist at https://github.com/resslabUser/OpenSees, where resslabUser would be replaced your username.

#### Clone your fork

Once the fork is created, it only exists on some github server, you now need a local copy (a clone) to edit on your own computer.
I provide the instructions using a terminal (e.g., command prompt or powershell on windows, or terminal on Mac), however, many editors provide a user interface for these same commands.
Navigate your terminal to the directory where you want the source files to be located, on Windows this can be anywhere on your system, on Mac this has to be your home directory (the "~" directory) due to how the file system is currently set-up.

Create a clone of your fork using
```
git clone git@github.com:resslabUser/OpenSees.git
```
a directory named OpenSees should now exist.
Note: you can print-out help on any git commands using
```
git help <command>
```
e.g., `git help clone`.

### Step 2. Create a new local branch and add the new source files

#### Create the branch

Branches help to organize changes made to git repositories.
It is best practice not to make edits on the master branch directly, changes should be committed to another branch and merged into master.

Navigate to the OpenSees directory in your terminal.
Create a new branch called "add_sl_mat" and checkout the branch using
```
git checkout -b add_sl_mat
```
The branch name should be short and descriptive of what changes will be made on it, here I chose "add_sl_mat" since I want to add the SLModel material.

#### Add / create / edit the source files

Here is the programming part, and I can't say much about that here.
If you already have the source files written you just copy them to the correct directory in `OpenSees/SRC/`.
Otherwise, create new files directly in the correct directory, or edit existing files.
The correct directory depends on the files being added, for example:
* Material -> `OpenSees/SRC/material/<uniaxial or nD>/`
* Element -> `OpenSees/SRC/element/<your element dir>/`

Best practice is to use autoformatting for all your source files, see the Useful Information section at the end of this document.

##### Windows: Let VS know about your additions / new files

Only the 64bit OpenSees solution is supported since around 2019, open this solution in VS from `OpenSees/Win64/OpenSees.sln`.

If you add the existing source files you need to let VS know about the additions.
The VS solution needs to be updated because you or others will compile OpenSees on Windows at some point.
For example, for the SLModel, add the .h and .cpp files as existing items to the `material` project under the `uniaxial` filter.
If you want to create new files then add new items (the .h and .cpp files) to the appropriate project under the appropriate filter.

If you are confused, see the following links for more information regarding projects and the VS build system:
- https://docs.microsoft.com/en-us/cpp/build/creating-and-managing-visual-cpp-projects?view=vs-2019
- https://docs.microsoft.com/en-us/cpp/build/projects-and-build-systems-cpp?view=vs-2019 

#### Update the Makefiles

`Makefile`s contain the instructions for Unix (i.e., Mac and Linux) build systems.
The Makefiles need to be updated because you or others will compile OpenSees on these systems at some point.
The editing is simple for materials, e.g., for a uniaxial material add a line in `OpenSees/SRC/material/uniaxial/Makefile`:
```
SLModel.o \
```
The idea is the same for adding elements, but slightly more complicated, look at the directory of an existing element for an example.

#### Add the updates to the interpreters

Interpreters read input files, create OpenSees models, and run these models.
If the interpreter is not "aware" of e.g., your new material, then you cannot actually use it in an analysis.
If you are just modifying existing code then you can ignore this.

There are two interpreters that are used from what I see: the TCL interpreter, and the Python interpreter.
At the moment (early 2020) the TCL interpreter is most common.
However, I see that there is a big push on developing the Python interpreter since Python is a better programming language than TCL, and package distribution is extremely convenient using `pip`.
It is a good idea to make your additions available to both interpreters for maximum up-take now and in the future.
I provide instructions for uniaxial materials only, the processes is similar for ND materials (but use the ND material instead of uniaxial), and see other elements for an example.

Add to `OpenSees/SRC/material/uniaxial/TclModelBuilderUniaxialMaterialCommand.cpp` for the TCL interpreter. 
Near the top of the file:
```C++
extern void *OPS_SLModel(void);
```
and somewhere near the middle of the file:
```C++
else if (strcmp(argv[1], "SLModel") == 0) {
      void *theMat = OPS_SLModel();
      if (theMat != 0)
        theMaterial = (UniaxialMaterial *)theMat;
      else
        return TCL_ERROR;
    }
```
See the context in these files for these to make sense.
    
Add to `OpenSees/SRC/interpreter/OpenSeesUniaxialMaterialCommands.cpp` for the Python interpreter.
Near the top of the file:
```C++
void* OPS_SLModel();
```
and further down in the file:
```C++
uniaxialMaterialsMap.insert(std::make_pair("SLModel", &OPS_SLModel));
```

#### Add the class tag

Increment the tag number and add a line for the `MAT_TAG_SLModel` in the `OpenSees/SRC/classTags.h` file.

### Step 3. Compile OpenSees

Compiling OpenSees can be tricky and time consuming depending on the current state of the source code.
The problem is that users can introduce errors without checking if the code compiles (or maybe it compiles for them but not others), and then those changes get incorporated into the main OpenSees repository.
Ideally you will compile on both Windows and Mac to catch as many errors as possible.
I recommend starting with Windows since it is easier in my opinion.

Use the Useful Information section at the end for additional details regarding compiling OpenSees.

### Step 4. Validate your additions

Your code needs to be validated prior to pushing your changes to the main OpenSees repository.
After you have built an OpenSees executable create a simple input file to validate your code, then run the input file and verify that the output is exactly as expected.

It is best practice to create a test suite.
A test suite contains all the input files used to validate your material/element/etc, and will be a separate repository.
For example, you may have a personal repository called `github.com/resslabUser/SLModel-Testing`.
As you encounter bugs, add a new input file that re-creates this bug and add this file to your test suite.
You will build-up a suite that creates confidence in your implementation.
Every time you update your source files, re-run the entire test suite to make sure that you didn't break anything that worked before, and ensure that you fixed the bug that you wanted to.


### Step 5. Commit your changes in git

First check the the files that you have modified using
```
git status
```
Make sure that only the files that you have modified are changed before you stage (add) the changes.
Note that if you update the VS projects certain files associated with the Win64 solution will be modified (e.g., "materials.vcxproj.filters"), make sure to commit these changes as well.
If you are curious about any changes, you can check the changes to individual files using
```
git diff <path to your file>
```

Once you are happy with the changes to branch, stage all the changes using
```
git add -A
```
or individual files can be staged using `git add <path to file>`.
Then commit the changes to your branch using
```
git commit
```
You will have to write a commit message, more information about commit messages is provided below.

#### Writing commit messages

Quick recommendations for writing commits:
* Use max 80 characters in each line.
* Be concise.
* Use imperative statements (e.g., "Add the SLModel by Suzuki and Lignos", imagine you are giving commands to the computer).
* The first line should summarize what you are changing, add additional lines starting with "-" to give specifics if necessary.
* I like to group additional items if I made more than one task in a single commit, for example, a commit message may look like:
```
<Major task that I did>

-Add <something new you added>
-Add <something else new>

-Modify <something you modified>

-Delete <something you deleted>
```

#### How often to commit?

In my opinion, it is best practice to commit after completing a particular task.
The reason is that git keeps track of the differences between commits
If you made an error in a particular task it is easier to identify the error when there is only one task per commit.
This task structure also helps to break down the logic of actually writing the code.

For example, if your task was to fix the material tag for your material, you may spend 30 seconds to change one line and commit this change.
The commit message could be:
```
Fix the material tag for the SLModel.
```
If your task was to write the time-integration for your material model, you may spend 3 hours and commit 100 lines and the commit message could be:
```
Add the time integration to the SLModel.
```

### Step 6. Push to your fork and create a pull request for the main OpenSees repository

Once you have completed the first version of your material/element/etc. it is time to start the process of it being added to the OpenSees repository.
Completed means that the code compiles (ideally tested on both Mac and Windows), and the output is validated with examples.

After commiting to a branch, push the changes to your fork on github using
```
git push origin <your branch>
```

When you go to the page for your fork on github a green bar should appear at the top to create a new pull request for the main OpenSees repository.
Start a new pull request, below are some best practices:
* Use a short descriptive title.
* Add some concise details in the comments.
* Add an academic reference if possible.
* Add a link to the test suite or some examples.
* Ensure that there are no conflicts the with main repository. Your pull request will not be merged if there are conflicts.
* Check over all the differences in the files on github before submitting the pull request to make sure that everything looks OK (this is a good last place to catch obvious errors).

After the pull request is made you have to wait for a user with merge permissions to accept the pull request.
In my experience this depends on:
1. If there are conflicts with the main repository (it will never be merged)
1. The magnitude and complexity of the changes
1. How well documented and clear the commits and changes are
1. If you are a known contributor

A small bugfix from a known user can be merged in minutes if someone is active at that time, while new analysis modes and materials with errors in the pull request can take months of iterations.
The `add_sl_mat` branch should be deleted after the pull request has been accepted.


## Useful information


### Keeping your local clone up-to-date

After some period of time you may return to your local repository, any changes merged into the main (upstream) OpenSees repository will not be applied to your local repository.
To avoid conflicts, you should first update your local repository, then make the changes you have in mind.

You can update your local repository by pulling from upstream.
First, you add a new url named `upstream` that targets the main repository:
```
git remote add upstream git@github.com:OpenSees/OpenSees.git
```
Verify that the new remote has been added using
```
git remote -v
```
This step only need to be done once per local clone.

Second, pull the changes from the upstream repository into your local master branch using
```
git checkout master
git pull upstream master
```
This needs to be done every time you want to update your local clone.
If there are any conflicts you need to resolve these and commit the changes before continuing.

Your local master branch is now up-to-date with the upstream master.
You can apply these changes to other local branches using
```
git checkout <branch you want checked out>
git merge master
```
This workflow is very useful if you have a separate branch with local changes used for compiling, e.g., `git checkout VS_2019`, `git merge master`.


### C++ style advice

1. Use an editor that has an auto formatting feature (e.g., Visual Studio, Visual Studio Code).
1. Set the editor for your preferences (e.g., I use spaces instead of tabs and a tab width of 2 for C++ and Fortran).
1. *Always* autoformat your source code files before saving them (e.g., Ctrl-K, Ctrl-D in Visual Studio).


### Help for building OpenSees

Regardless of your system, ensure that you have TCL installed according to the instructions provided on the OpenSees download page (http://opensees.berkeley.edu/OpenSees/user/download.php).

#### Compiling on Windows

##### Linking fortran compiler

You will require the Intel Fortran compiler for Windows, this comes with the student license of Intel Parallel Studio.
To link with the fortran compiler:
1. Right click on the OpenSees project in the **Solution Explorer** and select **Properties**
1. Make sure that in the Configuration bar **"All Configurations"** is selected
1. Expand **Linker** under **Configuration Properties** and select **General** under Linker
1. Click on **Additional Library Directories** in the main box and edit this entry
1. Add another directory that includes the library files for intel fortran (e.g,. `C:\Program Files (x86)\IntelSWTools\compilers_and_libraries_2019\windows\compiler\lib\intel64_win`)
1. Accept the change by clicking OK etc.

Note that the location of the folder will change depending on your version of Intel parallel studio and where you installed it.
You could probably use gfortran instead of Intel fortran, but I don't go into that here.

##### Building OpenSees (TCL)

1. Link the fortran compiler
1. Choose the build configuration (either Debug or Release)
1. Right click on the **OpenSees** project in the **Solution Explorer** and select **Build**

If there are no errors an OpenSees.exe will be created in `OpenSees/Win64/bin/`.


##### Building OpenSeesPy36

Ensure you have installed a Python distribution, I recommend Anaconda Python3 64bit.
You will need to link some Python libraries and header files, to locate these directories open a terminal and enter `python` to open the Python prompt.
In the prompt, enter
```Python
# thanks to https://stackoverflow.com/a/35071359
from sysconfig import get_paths
from pprint import pprint

info = get_paths()  # a dictionary of key-paths

# pretty print it for now
pprint(info)
```
For me the output is
```Python
{'data': 'C:\\Users\\hartlope\\AppData\\Local\\Continuum\\Anaconda3',
 'include': 'C:\\Users\\hartlope\\AppData\\Local\\Continuum\\Anaconda3\\Include',
 'platinclude': 'C:\\Users\\hartlope\\AppData\\Local\\Continuum\\Anaconda3\\Include',
 'platlib': 'C:\\Users\\hartlope\\AppData\\Local\\Continuum\\Anaconda3\\Lib\\site-packages',
 'platstdlib': 'C:\\Users\\hartlope\\AppData\\Local\\Continuum\\Anaconda3\\Lib',
 'purelib': 'C:\\Users\\hartlope\\AppData\\Local\\Continuum\\Anaconda3\\Lib\\site-packages',
 'scripts': 'C:\\Users\\hartlope\\AppData\\Local\\Continuum\\Anaconda3\\Scripts',
 'stdlib': 'C:\\Users\\hartlope\\AppData\\Local\\Continuum\\Anaconda3\\Lib'}
```
I will call e.g., `C:\\Users\\hartlope\\AppData\\Local\\Continuum\\Anaconda3` as `<python_path>`

Now we are ready to build the OpenSeesPy36 project:
1. Build the OpenSees project (TCL interpreter, see above), this is necessary to generate `tcl.lib`.
1. Follow the instructions for linking the fortran compiler, but select the OpenSeesPy36 project instead.
1. Add another entry to the **Additional Library Directories** (similar to the fotran compiler directory): `<python_path>\libs`
1. **Under Configuration Properties**, select **VC++ Directories**
1. Add an additional entry to **Include Directories**: `<python_path>\Include`
1. Accept the changes
1. Choose the build configuration (either Debug or Release)
1. Right click on the **OpenSeesPy36** project in the **Solution Explorer** and select **Build**

If there are no errors an opensees.pyd will be created in `OpenSees/Win64/bin/py36/`.

#### Compiling on Mac/Linux

These instructions are adapted from ones provided by Dr. Albano de Castro e Sousa.
Disclaimer: I have only used Mac and not Linux, but in theory the steps are similar (also with the possibility of using apt-get instead of homebrew).

- Install Xcode
- Install tcl (follow guidelines from the OpenSees website: http://opensees.berkeley.edu/OpenSees/user/download.php )
- Install gfortran:
  - Install command line tools in Terminal: `xcode-select --install`
  - Install homebrew in Terminal: `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
  - Install gfortran as part of gcc in brew: brew install gcc
- Copy latest make definitions file (e.g. `OpenSees/MAKES/Makefile.def.MacOS10.8`) to `OpenSees/` as `Makefile.def`
- Make sure that all the links to the compilers are all right, i.e. that the g++ is not g++-8 and that it's in the bin folder
- Make sure all paths to libraries are okay in Makefile.def - e.g. `SUPERLUdir = $ (HOME)/OpenSees/OTHER/SuperLU5.1.1/SRC`
- Create two folders: `/Users/username/bin` for the output executable, and `/Users/username/lib` for compiled libraries
- Install openssl and add to `MACHINE_INCLUDES` in Makefile.def
  - `brew install openssl``
  - `-I/usr/local/Cellar/openssl/1.0.2r/include`
  - Remove the `-lssl` flag `MACHINE_SPECIFIC_LIBS` in Makefile.def
- Add library paths as needed to `MACHINE_NUMERICAL_LIBS` in Makefile.def e.g.:
  - `/usr/local/Cellar/gcc/8.3.0/lib/gcc/8/libgfortran.a`
  - `/usr/local/Cellar/gcc/8.3.0/lib/gcc/8/libquadmath.a`
  - `/usr/local/Cellar/openssl/1.0.2r/lib/libssl.dylib`
- `make -B` to compile everything from scratch

If there are no errors an opensees executable should appear in `/Users/username/bin`.


Instructions for compiling OpenSeesPy on Mac/Linux are TBD.

### Recommended workflow for Visual Studio

Best practice is to not modify the upstream VS solution with *local system* dependent changes.
For example, do not try to add `C:\Program Files (x86)\IntelSWTools\compilers_and_libraries_2019\windows\compiler\lib\intel64_win` to the upstream VS solution, since this directory will be different for almost every user.
The solution I use is to verify that the code compiles in a branch for local changes, then commit these changes in a different branch that does not include these local changes.
Then, pull requests for the main OpenSees repository are only made from branches different from the one that contains the local changes.
Let me explain the workflow.

#### First time - create a branch for local changes
1. Create a branch off of master for local changes, e.g., `git checkout master`, `git checkout -b VS_2019`.
1. Make local changes (e.g., link the Fortran compiler, link the Python headers).
1. Verify that OpenSees compiles.
1. Commit these changes to `VS_2019`.

#### Everytime afterwards (assume some time has passed)

1. Update your local master branch by pulling from upstream.
1. Create a new branch off of the updated master, e.g., `fix_sl_mat` that will contain your proposed changes.
1. Update the `VS_2019` branch: `git checkout VS_2019`, `git merge master`.
1. Stay on the `VS_2019` branch.
1. Write code to complete a task (still on the `VS_2019` branch), do not add or commit anything yet.
1. Verify that the code compiles.
1. Validate the changes with your test suite.
1. Checkout the `fix_sl_mat` branch: `git checkout fix_sl_mat`.
1. Commit the changes on `fix_sl_mat`.
1. Update the `VS_2019` branch with your changes: `git checkout VS_2019`, `git merge fix_sl_mat`.
1. Make more changes, verify compilation, validate, etc., as necessary.
1. Make the pull request using `fix_sl_mat` when you are finished.
