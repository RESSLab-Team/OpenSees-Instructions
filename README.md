# Adding to OpenSees

This document includes instructions and best practices for adding new classes (e.g., materials, elements) to the OpenSees framework, and adding these changes to the OpenSees repository hosted on github.
The target reader: is willing to google, has a background in civil/structural engineering, and has limited knowledge of object-oriented programming.

## Preliminaries

### Reading

Read these articles before getting started to understand the logic behind the OpenSees design:
* 10.1061/(ASCE)0733-9445(2008)134:4(562)
* 10.1061/(ASCE)CP.1943-5487.0000002

### OpenSees is hosted on github...

... this means that you need to have some knowledge of `git` (the software behind github) and github itself (which is the website that hosts the OpenSees git repository).
The less you know the more you will google.
There is jargon in this document (not only regarding git) - this is on purpose so that you can google and get accurate results to questions you may have (e.g., "add existing item to visual studio project").
* I cannot explain everything about git and github - there are thousands of guides, tutorials, and stackexchange posts that explain things.
* I will provide some basic commands and some basic ideas, however, you will probably need to google several things along the way unless you already have this knowledge.
* I provide the command-line commands for using git, you will need to install `git` itself if it is not installed on your system.
* There are tools for git integrated into VS that you can use instead of the command line - you can explore these instead of the commands provided in this document.

### Windows: Visual Studio

I recommend to use the edition of Microsoft Visual Studio (VS) Community that corresponds to the OpenSees codebase.
At the time of writing this is VS 2019, and it can change without any major notice as far as I know, you can figure it out by the build tool version :-).
If you have an older version you just need to change the build tool version for each project in VS.

### Mac and Linux

TBD



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

Once the fork is created, it only exists on some github server, you now need a local copy to edit (a clone) on your own computer.
Open a terminal (e.g., command prompt or powershell on windows, or terminal on Mac).
Navigate your terminal to the directory where you want the source files to be located, on Windows this can be anywhere on your system, on Mac this has to be your home directory (the "~" directory).

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
Otherwise, you create new files directly in the correct directory, or edit existing files.
The correct directory depends on the files being added, for example:
* Material -> `OpenSees/SRC/material/<uniaxial or nD>/`
* Element -> `OpenSees/SRC/element/<your element dir>/`

Best practice is to use autoformatting for all your source files, see the Useful Information section at the end of this document.

##### Windows: Let VS know about your additions / new files

Only the 64bit OpenSees solution is supported since around 2019, open this solution in VS from `OpenSees/Win64/OpenSees.sln`.

If you add the existing source files you need to let VS know about the additions.
For example, for the SLModel, add the .h and .cpp files as existing items to the material project under the uniaxial filter.
If you want to create new files then add new items (the .h and .cpp files) to the appropriate project under the appropriate filter.

#### Update the Makefiles

`Makefile`s contain the instructions for Unix (i.e., Mac and Linux) build systems.
 Makefiles need to be updated because you or others will compile OpenSees on these systems at some point.
This step must be done even if *you* are compiling on Windows.
The editing is simple for materials, e.g., for a uniaxial material add a line in `OpenSees/SRC/material/uniaxial/Makefile`:
```
SLModel.o \
```
The idea is the same for adding elements, but slightly more complicated, look at the directory of an existing element for an example.

#### Add the updates to the interpreters

Interpreters read input files, create OpenSees models, and run these models.
If the interpreter does not "know" about your additions then you cannot use any added materials/elements/etc.
If you are just modifying existing code then you can ignore this.

There are two interpreters that are used from what I see: the "standard" TCL interpreter, and the "up-and-coming" Python interpreter.
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
```
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
You will build-up a suite that builds confidence in your implementation and eliminates bugs.
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
Then commit the changes to your branch using
```
git commit
```
You will have to write a commit message, more information about commit messages is provided below.

#### Best practices

##### Writing commit messages

Quick recommendations for writing commits:
* Use max 80 characters in each line.
* Be concise.
* Use imperative statements (e.g., "Add the SLModel by Suzuki and Lignos", imagine you are giving commands to the computer).
* The first line should summarize what you are changing, add additional lines starting with "-" to give specifics if necessary.
* I like to group additional items as necessary
```
-Add <something new you added>
-Add <something else new>

-Modify <something you modified>

-Delete <something you deleted>
```

##### How often to commit?

It is best practice to commit after completing a particular task.
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

Push the changes made to your branch to your fork on github using, e.g.,
```
git push origin add_sl_mat
```

When you go to the page for your fork on github a bar should appear at the top to create a new pull request for the main OpenSees repository.
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
This needs to be done everytime you want to update your local clone.
If there are any conflicts you need to resolve these and commit the changes before continuing.

Your local master branch is now up-to-date with the upstream master.
You can apply these changes to other local branches using
```
git checkout <branch you want checked out>
git merge master
```
This workflow is very useful if you have a separate branch with local changes used for compiling, e.g., `git checkout VS_2019`, `git merge master`.


### C++ style advice

1. Use an editor that has an auto formatting feature (e.g., Visual Studio). If your editor does not, get a new editor and use that one instead.
1. Set the editor for your preferences (e.g., I use spaces instead of tabs and a tab width of 2 for C++ and Fortran).
1. *Always* autoformat your source code files before saving them (e.g., Ctrl-K, Ctrl-D in Visual Studio).


### Compiling on Windows

#### Link fortran compiler

You will require the Intel Fortran compiler for Windows, this comes with the student license of Intel Parallel Studio.
To link with the fortran compiler:
1. Right click on the OpenSees project in the Solution Explorer and select Properties
1. Make sure that in the Configuration bar "All Configurations" is selected
1. Expand Linker under Configuration Properties and select General under Linker
1. Click on Additional Include Directories in the main box and edit this entry
1. Add another directory that includes the library files for intel fortran (e.g,. `C:\Program Files (x86)\IntelSWTools\compilers_and_libraries_2019\windows\compiler\lib\intel64_win`)
1. Accept the change by clicking OK etc.

Note that the location of the folder will change depending on your version of Intel parallel studio and where you installed it.
You could probably use gfortran instead of Intel fortran, but I don't go into that here.

#### Building OpenSees
1. Choose the build configuration (either Debug or Release)
1. Right click on the OpenSees project in the Solution Explorer and select Build

If there are no errors an OpenSees.exe will be created in `OpenSees/Win64/bin/`.


### Compiling on Mac

TBD


### Recommended workflow for Visual Studio

Best practice is to not modify the upstream VS solution with system dependent changes.
For example, do not try to add `C:\Program Files (x86)\IntelSWTools\compilers_and_libraries_2019\windows\compiler\lib\intel64_win` to the upstream VS solution, since this directory will be different for almost every user.
The solution I use is to verify that the code compiles in a branch for local changes, then commit these changes in a different branch that does not include these local changes.
Then, pull requests for the main OpenSees repository are only made from branches different from the one that contains the local changes.

#### First time - create a branch for local changes
1. Create a brach off of master for local changes, e.g., `git checkout master`, `git checkout -b VS_2019`.
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
