---
layout: post
title:  "python自定义安装"
date:   2015-06-23 16:27:12
author: white clover
categories: ops
tags: ops
---

## Installation

You can easily build your own Python binaries and libraries by following the steps below:


1. Retrieve the sources from the official website: http://www.python.org/download/
2. Unpack, compile and install Python (any version, here with Python 2.7):

```
    cd <compilation-directory>
    tar jvzf Python-2.7.tar.bz2
    cd Python-2.7
    ./configure --enable-shared [--prefix=/your/custom/installation/path]
    make
    make test
    make install
```

The `-enable-shared` option ensures to build both static and dynamic Python libraries. This option is mandatory for a correct behaviour of the Gildas-Python binding. The -prefix option allows you to install Python in a custom location (instead of /usr/local). This is useful in particular if you do not have administrative priviledges. Finally you should refer to section 3.1.3 if you want to enable the command line history in the Gildas-Python binding.

## Make your new Python available

Fill the binary and library location in the corresponding environment variables:

```bash
export PATH=/your/custom/installation/path/bin:$PATH
export LD_LIBRARY_PATH=/your/custom/installation/path/lib:$LD_LIBRARY_PATH
```

You can make your custom Python the permanent default for yourself (i.e. overriding the system Python), or you can make it a transient default to be used only for Gildas compilation. This depends on your needs, see post-compilation instructions in section 3.1.6.

## Check your installation:

```bash
which python
python -V
python -c "import sqlite3"
```

sqlite3 Python module is optional, needed only for the extension named Weeds for Class. If import sqlite3 fails, install (or ask your system administrator) sqlite3 headers on your system (system package providing sqlite3.h i.e. sqlite3-devel), and restart from step 2.

