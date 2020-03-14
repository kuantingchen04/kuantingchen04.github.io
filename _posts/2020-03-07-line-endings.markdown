---
title: "Line Endings (CR/LR/CRLR)"
layout: post
date: 2020-03-07 10:48
image: /assets/images/markdown.jpg
headerImage: false
tag:
- engineering
- git
- 
category: blog
author: kevchen
description: Line endings summary on different OS
---

If you are developing softwares on cross-platform projects (e.g. Windows/Linux/MacOS), you will find that the line endings is sometimes pretty annoying, especially when reading files.

## Outline
1. [Line Endings (Line Seperators)](#i-line-ending-formats)
    * Carriage Return (CR)
    * Line Feed (LR)

2. [How to Look-Up / Change Text's Terminators](#ii-look-up--change-texts-terminators)

3. [[Git] Auto-Corrent Line Ending Format on Cross-Platform Projects](#iii-auto-correct-gits-text-line-ending)
    * [Git Configuration](#solution-1-git-configuration)
    * [Setup .gitattributes](#solution-2-gitattribute)

4. [[Git] Ignore File Mode on Cross-Platform Projects](#iv-ignore-file-mode)

---

## I. Line Ending Formats

There are two kinds of line endings, **Carriage Return** (CR, the code is \r) and **Line Feed** (LF, the code is \n).

On different OS, different line ending is used.

* OSX, Linux: **LF (\n)**
* Legacy MacOS (MacOS 9 and earlier): **CR (\r)**
* Windows: **CRLR (\r\n)** pair
    * Carriage return points the cursor to the beginning of the line horizontly and Line feed shifts the cursor to the next line vertically. Combination of both gives you new line(\n) effect.

---

## II. Look Up / Change Text's Terminators

If you download a file to your system from a different OS. You might notice that some weird behaviors occurs while processing the strings. Probably it is a result of inconsistency between text and OS.

### Ubuntu Users

For Ubuntu users, you can simply use ```file``` command to see the files line ending format.

```console
foo@bar:~$ file testfile1.txt
testfile.txt: ASCII text

foo@bar:~$ file testfile2.txt
testfile2.txt: ASCII text, with CRLF line terminators
```

To convert from DOS/Windows to Unix:

```
foo@bar:~$ dos2unix testfile2.txt
```


To convert from Unix to DOS/Windows: 

```
foo@bar:~$ unix2dos testfile1.txt
```

### Editor Users (Take JetBrains for Example)

Lots of editors/IDE allow user to set up line separators (line endings) for newly created files, and change line separator style for existing files.

![lineseperators_image](https://resources.jetbrains.com/help/img/idea/2019.3/cl_lineseparators_settings.png)
<figcaption class="caption">CLion Settings</figcaption><br/>

---

## III. Auto-Correct Git's Text Line Ending

If you are working on **cross-platform projects**, the subtle difference above could be incredibly annoying; many editors on Windows silently replace existing LF-style line endings with CRLF, or insert both line-ending characters when the user hits the enter key.

In order to gaurantee that the code fits your OS system, there are two common ways to set things up so git will git to auto-correct line ending formats.



### Solution 1: Git Configuration

Git can handle this by auto-converting CRLF line endings into LF when you add a file to the index, and vice versa when it checks out code onto your filesystem. That is, to change **core.autocrlf**. (You could add —-global to set all repos)

There are three values for this variable: **true**, **input**, **false**.

![test](/assets/images/2020-02-21-autocrlf.png)
<figcaption class="caption">Commit-Checkout Cycle</figcaption><br/>

```
git config core.autocrlf true
```

* Set **core.autocrlf=true**: Git will auto-convert CRLF line endings into LF when you add a file to the index; Git will convert LF to CRLF when checking out code.  (For cross-platform projects, this is the recommended setting on Windows)

```
git config core.autocrlf input
``` 

* Set **core.autocrlf=input**: When committing text files, CRLF will be converted to LF. Git will not perform any conversion when checking out text files. (For cross-platform projects this is the recommended setting on Unix, but don't use input under windows)

```
git config core.autocrlf false
```

* Set **core.autocrlf=false**: Git will not perform any conversions when checking out or committing text files. (Choosing this option is not recommended for cross-platform projects)

### Solution 2: .gitattribute

It is a good idea to keep a .gitattributes file as we don't want to expect everyone in our team set their config. This file should keep in repo's root path and if exist one, git will respect it.

```
text=auto  #Convert to OS’s line ending
```
* This will treat all files as text files and convert to OS's line ending on checkout and back to LF on commit automatically.

```
* text eol=crlf
* text eol=lf
```
* You can also tell it explicitly

## IV. Ignore File Mode

After you clone a repo, you may find that even if we set up `core.autocrlf` as we described above, it is still saying that files have been modified. The reason is that the file permissions may not be supported. (Linux has more complete access rights *-rwxrwxrwx* than Windows) You could try and set `core.fileMode` to **false**.

```
git config core.fileMode false
```

*If you set core.fileMode to false, make sure that executable's and script's permission is well handle.*

As <https://stackoverflow.com/a/1580644> suggested, `core.fileMode` is not the best practice and should be used carefully. This setting only covers the executable bit of mode and never the read/write bits. In many cases you think you need this setting because you did something like `chmod -R 777`, making all your files executable. But in most projects **most files don't need and should not be executable for security reasons**.

The proper way to solve this kind of situation is to handle folder and file permission separately, with something like:

```
find . -type d -exec chmod a+rwx {} \; # Make folders traversable and read/write
find . -type f -exec chmod a+rw {} \;  # Make files read/write
```

```
core.fileMode
    
    Tells Git if the executable bit of files in the working tree is to be honored.

    Some filesystems lose the executable bit when a file that is marked as executable is
    checked out, or checks out an non-executable file with executable bit on. [git-clone(1)]
    (https://www.kernel.org/pub/software/scm/git/docs/git-clone.html) or [git-init(1)]
    (https://www.kernel.org/pub/software/scm/git/docs/git-init.html) probe the filesystem to 
    see if it handles the executable bit correctly and this variable is automatically set as 
    necessary.

    A repository, however, may be on a filesystem that handles the filemode correctly, and 
    this variable is set to *true* when created, but later may be made accessible from 
    another environment that loses the filemode (e.g. exporting ext4 via CIFS mount, visiting 
    a Cygwin created repository with Git for Windows or Eclipse). In such a case it may be 
    necessary to set this variable to *false*. See [git-update-index(1)]
    (https://www.kernel.org/pub/software/scm/git/docs/git-update-index.html).

    The default is true (when core.filemode is not specified in the config file).
```



## Reference

##### Line Endings
* <https://en.wikipedia.org/wiki/Carriage_return#Computers>
* <https://stackoverflow.com/a/3570574>
* <https://www.jetbrains.com/help/clion/configuring-line-endings-and-line-separators.html>
* <https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration>
* <https://stackoverflow.com/a/20653073>

##### File Mode
* <https://www.jianshu.com/p/3b0a9904daca>
* <https://stackoverflow.com/a/1580644>
* <http://linuxcommand.org/lc3_lts0090.php>
