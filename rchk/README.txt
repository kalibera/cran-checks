Checks of native code (C/C++) based on static code analysis - no R code is
run, but the native code is built into LLVM bitcode using Clang, and this
bitcode is analyzed offline.

Currently the analysis reports potential errors in the use of PROTECT and
related macros for protecting objects on the R heap against memory
reclamation.  More information on how to use PROTECT is available in §5.9.1
of 'Writing R Extensions'.

Some reports are false alarms, but some of those still deserve to be fixed
in the interest of better code readability and robustness to modifications
of R internals. 

The results are obtained using rchk, which is available at
https://github.com/kalibera/rchk, but are further filtered in order to
reduce the false alarms rate. The tool has been written primarily for C and
works less well with C++.

Detailed information on how to interpret the reports and additional
recommendations on how to use PROTECT is available at
https://github.com/kalibera/cran-checks/blob/master/rchk/PROTECT.md

Rchk is also available in a docker container, which can be used on Linux,
macOS and Windows, e.g.

docker pull kalibera/rchk:latest

# check package audio from CRAN
docker run kalibera/rchk:latest audio

# check package lazy from a tarball on Linux or macOS
mkdir packages
cp lazy_1.2-16.tar.gz packages
docker run -v `pwd`/packages:/rchk/packages kalibera/rchk:latest /rchk/packages/lazy_1.2-16.tar.gz 

# check package lazy from a tarball on Windows
mkdir packages
copy Downloads\lazy_1.2-16.tar.gz packages
docker run -v %cd%/packages:/rchk/packages kalibera/rchk:latest /rchk/packages/lazy_1.2-16.tar.gz

More information about the docker image and rchk:

https://github.com/kalibera/rchk/blob/master/doc/DOCKER.md
https://github.com/kalibera/rchk
