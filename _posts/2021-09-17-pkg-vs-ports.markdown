---
layout: post
title: PKG vs Ports
date: 2021-09-17
---

FreeBSD is bundled with a rich collection of system tools as part of the base system. In addition, FreeBSD provides two complementary technologies for installing third-party software: 

* the FreeBSD Ports Collection, for installing from source

* packages, for installing from pre-built binaries. 

Either method may be used to install software from local media or from the network.

For more detailed information you can refer to [Chapter 4. Installing Applications: Packages and Ports](https://docs.freebsd.org/en/books/handbook/ports/)

 
While the two technologies are similar, packages and ports each have their own strengths. You should select the technology that meets your requirements for installing a particular application.

## What I prefer

My policy is to use pre-compiled binaries as much as possible to:

* avoid the need of tinkering with foreign source code and compiler options

* have easily reproducible build

* update more easily

* use more reliable and stable versions.

## Package (pkg) Benefits

* A compressed package tarball is typically smaller than the compressed tarball containing the source code for the application.

* Packages do not require compilation time. For large applications, such as Mozilla, KDE, or GNOME, this can be important on a slow system.

* Packages do not require any understanding of the process involved in compiling software on FreeBSD.

## Port Benefits

* Packages are normally compiled with conservative options because they have to run on the maximum number of systems. By compiling from the port, one can change the compilation options.

* Some applications have compile-time options relating to which features are installed. For example, Apache can be configured with a wide variety of different built-in options.

* In some cases, multiple packages will exist for the same application to specify certain settings. For example, Ghostscript is available as a ghostscript package and a ghostscript-nox11 package, depending on whether or not Xorg is installed. Creating multiple packages rapidly becomes impossible if an application has more than one or two different compile-time options.

* The licensing conditions of some software forbid binary distribution. Such software must be distributed as source code which must be compiled by the end-user.

* Some people do not trust binary distributions or prefer to read through source code in order to look for potential problems.

* Source code is needed in order to apply custom patches.
