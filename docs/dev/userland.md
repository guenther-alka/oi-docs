<!--

The contents of this Documentation are subject to the Public Documentation License Version 1.01
(the "License"); you may only use this Documentation if you comply with the terms of this License.
A copy of the License is available at http://illumos.org/license/PDL.

The Original Documentation is _________________.

The Initial Writer of the Original Documentation is Aurelien Larcher Copyright (C) 2016 [Insert year(s)].
All Rights Reserved. (Initial Writer contact(s):________________[Insert hyperlink/alias]).

Contributor(s): Johnathan Jenkins, Michael Kruger, Theodore Seán Tubbs

Portions created by ______ are Copyright (C)_________[Insert year(s)].
All Rights Reserved. (Contributor contact(s):________________[Insert hyperlink/alias]).

-->

<img src = "../../Openindiana.png">

# Quick start for oi-userland

### Clone the repository from OpenIndiana's Github, it is set as origin:

```bash
git clone https://github.com/OpenIndiana/oi-userland.git
cd oi-userland
```

### Add your Github repository as remote:

```bash
git remote add my_name https://github.com/my_name/oi-userland.git
```

### Print for checking:

```bash
git remote -v
```

### Initial setup including creation of local on-disk repository and check:

```bash
gmake setup
gmake env-check
```

### Every time you add or modify a component, create a new branch:

```bash
git checkout -b my_feature
```

### Keep this branch synchronized with origin/oi/hipster:

```bash
git pull --rebase origin oi/hipster
```

Your local branch is forwarded to the last commit of oi/hipster and your additional commits are kept on top of the stack.

### A component consists of several files:

* `Makefile`: the recipe to build the software (in the `build/$MARCH` directory) and install it locally (to the `build/proto` directory)
* `patches/`: directory containing patches applied before the configuration
* `*.p5m`: manifests used to generate the IPS package
* `$(COMPONENT_NAME).license`: file containing the licenses applicable to the software

### Here is a list of important targets for `gmake`

| Target | Description |
|---|---|
| `clobber` | cleans up the component directory completely, including deleting source |
| `download` | fetches the source archive and verify its `SHA256` sum |
| `prep` | extract and apply patches |
| `build` | configure and build |
| `install` | install locally |
| `sample-manifest` | generate an IPS manifest based on the files installed locally |
| `publish` | generate dependencies and publish the package to the local repository |

### First you need to make sure that `gmake prep` passes, so you can start by changing the component's metadata:

Example with `components/libjpeg6-ijg/Makefile`:

```bash
COMPONENT_NAME# libjpeg6-ijg
COMPONENT_VERSION# 6.0.2
LIBJPEG_API_VERSION# 6b
COMPONENT_FMRI# image/library/libjpeg6-ijg
COMPONENT_CLASSIFICATION# System/Multimedia Libraries
COMPONENT_PROJECT_URL# http://www.ijg.org/
COMPONENT_SUMMARY# libjpeg - Independent JPEG Group library version 6b
COMPONENT_SRC# jpeg-$(LIBJPEG_API_VERSION)
COMPONENT_ARCHIVE# $(COMPONENT_NAME)-$(COMPONENT_VERSION).tar.gz
COMPONENT_ARCHIVE_HASH# \
 sha256:75c3ec241e9996504fe02a9ed4d12f16b74ade713972f3db9e65ce95cd27e35d
COMPONENT_ARCHIVE_URL# http://www.ijg.org/files/jpegsrc.v$(LIBJPEG_API_VERSION).tar.gz
COMPONENT_LICENSE# IJG,GPLv2.0
COMPONENT_LICENSE_FILE# $(COMPONENT_NAME).license
```

| Variable | Value | Comment |
|---|---|---|
| COMPONENT_NAME | libjpeg6-ijg | Use the same name as in SFE or other Illumos userlands if applicable, otherwise follow Debian |
| COMPONENT_VERSION | 6.0.2 | Should be numerical only, not letters |
| LIBJPEG_API_VERSION | 6b | Local variable declared in the Makefile should be prefixed with the component's name |
| COMPONENT_FMRI | image/library/libjpeg6-ijg | Follow the convention for the FMRI, check a similar component |
| COMPONENT_CLASSIFICATION | System/Multimedia Libraries | This entry should be in the [OpenSolaris IPS Classification 2008](http://hub.openindiana.ninja/?q#content/opensolaris-ips-classification-2008) |
| COMPONENT_PROJECT_URL | http://www.ijg.org/ | Upstream project website |
| COMPONENT_SUMMARY | libjpeg - Independent JPEG Group library version 6b | Short description, one-liner |
| COMPONENT_SRC | jpeg-$(LIBJPEG_API_VERSION) |  |
| COMPONENT_ARCHIVE | $(COMPONENT_NAME)-$(COMPONENT_VERSION).tar.gz | Only change the extension |
| COMPONENT_ARCHIVE_HASH | sha256:75c3ec241e9996504fe02a9ed4d12f16b74ade713972f3db9e65ce95cd27e35d | To be generated |
| COMPONENT_ARCHIVE_URL | http://www.ijg.org/files/jpegsrc.v$(LIBJPEG_API_VERSION).tar.gz | Full path with archive filename if not equal to COMPONENT_ARCHIVE |
| COMPONENT_LICENSE | IJG,GPLv2.0 | Comma separated list of licenses |
| COMPONENT_LICENSE_FILE | $(COMPONENT_NAME).license | Do not change |

Run the first targets:

* `gmake download`: if the checksum fails, replace `COMPONENT_ARCHIVE_HASH` with the actual hash.
* `gmake unpack`: once the sources are extracted, concatenate the license files to $(COMPONENT_NAME).license, here "libjpeg6-ijg.license".
* `gmake patch`: to apply patches.

If you do not have any patches, you can as well run `gmake prep` directly.


### Patch, Build and install

The included .mk file depends on the build system, example:


```bash
include $(WS_TOP)/make-rules/configure.mk
```

Look in the `make-rules` directory for more

| File | Build |
|---|---|
| `ant.mk` | Ant |
| `attpackagemake.mk` | AT&T package tools |
| `cmake.mk` | CMake |
| `configure.mk` | Autotools |
| `gem.mk` | Ruby |
| `justmake.mk` | plain Makefile |
| `makemaker.mk` | Perl |
| `setup.py` | Python distutils |

Read the `.mk` file to see which variables you can modify, in general you can find variables such as:

* `*_ENV`
* `*_OPTIONS`
* `PRE_*_ACTION`
* `POST_*_ACTION`

For example, you may add this line for an Autotools-based component:


```bash
CONFIGURE_OPTIONS+# --enable-shared
```

It is now up to you to: patch, play with the configuration flags and such...
Do not hesitate to look around to see how it is done in other components !


### Prepare the IPS manifest

When the `install` target passes, you can run:


```bash
gmake sample-manifest
```

to generate a manifest from the list of installed files.

Copy the file `build/manifest-generated.p5m` to `$(COMPONENT_NAME).p5m` and edit it:

* Add you name as contributor
* Remove unused entries from the manifest:
    * directories: `:%g/^dir/d` (Vim)
    * static libs: `:%g/.a$/d` (Vim)
    * libtool files:: `:%g/.la$/d` (Vim)
    * Python *.pyc: `:%g/.pyc$/d` (Vim)

For some components, specific rules need to be applied: they can be implemented with *transforms*.
Some example can be found in the directory with the same name at the root directory of oi-userland.


### Publish the package to your local repository

Run `gmake publish`: if the dependencies are resolved and the manifest is valid, your package is published to the local repository.

To be able to search for the new packages in the local repository you need to rebuild search indexes:


```bash
pkgrepo refresh -s /path/to/my_repo
```

You can even rebuild the entire metadata:


```bash
pkgrepo rebuild -s /path/to/my_repo
```

### Install the package and test

After you've published the package to your local repository and rebuilt the repository index or metadata, you can install the package and perform whatever testing is appropriate.

When you ran `gmake setup` earlier, that step added the local repository with a default name of userland to the list of package publishers that `pkg` knows about.  It also made the local userland repository a higher priority than openindian.org.  You can verify the list of package publishers with:

```bash
pkg publisher
```

If the package you built and published to the local userland repository is not already part of hipster, it should be straightforward to install it:

```bash
pfexec pkg install your/package/name
```

If, however, the package you built is an updated version of an existing package, then you may have to take additional steps before it can be updated.

If `pkg` refuses to install the package from your local repository, it may be because the userland-incorporation is preventing updates to the version of the package:

```bash
$ pfexec pkg update print/filter/hplip
No updates available for this image.
$ pfexec pkg update pkg://userland/print/filter/hplip@3.15.9,5.11-2017.0.0.2:20170228T235543Z
pkg update: No matching version of print/filter/hplip can be installed:
  Reject:  pkg://userland/print/filter/hplip@3.15.9-2017.0.0.2
  Reason:  This version is excluded by installed incorporation consolidation/userland/userland-incorporation@0.5.11-2017.0.0.8131
```

If you will be installing many test versions of packages on your development system, you may find it easiest to uninstall userland-incorporation.  Alternately, if you want to test a package on a system while keeping userland-incorporation, you can use `pkg change-facet` to relax the version constraint for just that package:

```bash
pfexec pkg change-facet facet.version-lock.your/package/name/here=false
```

After you have performed one of those steps to remove the version constraint, there is one more issue you may encounter.  Because the installed version of the package came from the openindiana.org publisher but the updated version you want to install and test is associated with the userland publisher, `pkg` will by default not allow the package update to switch which publisher provides the package.

One option to work around this is to make the openindiana.org publisher non-sticky:

```bash
pfexec pkg set-publisher --non-sticky openindiana.org
```

You only need to perform that operation on your development system once.

Alternately, you can force `pkg` to apply an update from a different publisher by specifying the full FMRI for the package, including the publisher:

```bash
pfexec pkg update pkg://userland/print/filter/hplip@3.15.9,5.11-2017.0.0.2:20170228T235543Z
```


### Submit your component

Once you've installed and tested the package, you are ready to clean up your branch and then submit a pull-request.

First you need to *squash* all your commits into one, check how many commits are to be considered:


```bash
git log
```

then


```bash
git rebase -i HEAD~N
```

with N the number of commits to be squashed, and follow the instructions: the letter 's' should be put in place of 'pick' for the N - 1 commits before the last.

If you made a mistake with the commit message or author, use:


```bash
git commit --amend
```

with the relevant option.

Then you are ready to push:


```bash
git push my_name my_feature
```

or


```bash
git push -f my_name my_feature
```

if the branch you just rebased had already been pushed: since the history is rewritten you need to force the push, be careful.

Go to your Github profile and open a pull request.
