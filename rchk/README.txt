Checks of native code (C/C++) based on static code analysis - no tests are
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
reduce the false alarms rate.  The tool has been written primarily for C and
works less well with C++.

Detailed information on how to interpret the reports and additional
recommendations on how to use PROTECT is available at
https://github.com/kalibera/cran-checks/blob/master/rchk/PROTECT.md
