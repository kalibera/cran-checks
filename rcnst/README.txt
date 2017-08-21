Checks of corruption of constants.  Constants here mean R objects with NAMED
value of 2, which must be treated as immutable.  Immutability of these
objects is enforced in R code, but cannot be enforced in C/C++, and thus
incorrect C/C++ package code may modify (corrupt) constants.  The
consequences are unpredictable, the program may crash, may produce incorrect
result or may run normally, depending on where the corrupted constant
happens to be used next.  Commonly constant corruptions leads to that inside
an R function all constant literals such as number 1 will change their value
to say 0.  These bugs often originate from C/C++ code in packages that
modify input arguments of a function called via .Call.  Such a modification
is a violation of R semantics (almost always one has to duplicate an
argument and change only its copy, as detailed in CONSTANTS.md linked
below).

The checks are based on run-time detection of corruption of constants used
by the byte-code compiler.  Hence, for these checks, all executing code is
byte-compiled: packages are pre-compiled via R_COMPILE_PKGS=1 and
R_JIT_STRATEGY=4 is set to ensure that all code is byte-compiled by the JIT,
even when it is expected to harm performance.  Then, package tests are run
with R_CHECK_CONSTANTS=5, which is the most strict but also slowest level of
checking of the constant corruption.  These tests can be easily re-run by
developers as they use an unmodified version of R.

Only packages that failed due to constants corruption are shown here. 
Packages that failed for other reasons were excluded (a package may fail
these checks also due to other problems, such as due to incompatibility with
the byte-code compiler).

Detailed information on how to interpret the reports on corrupted constants
is available at
https://github.com/kalibera/cran-checks/blob/master/rcnst/CONSTANTS.md
