Checks of corruption of constants.  Constants here mean R objects with NAMED
value of 2, which must be treated as immutable.  Immutability of these
objects is enforced in R code, but cannot be enforced in C/C++, and thus
incorrect C/C++ package code may modify (corrupt) constants.  The
consequences are unpredictable, the program may crash, may produce incorrect
result or may run normally, depending on where the corrupted constant
happens to be used next.  Commonly constant corruptions leads to that inside
an R function all constant literals such as number 1 will change their value
to say 0.  These bugs ofter originate from C/C++ code in packages that
modify input arguments of a function called via .Call, which violates R
semantics (almost always one has to duplicate an argument and change only
its copy, as detailed in CONSTANTS.md linked below).

The checks are based on run-time detection of corruption of constants used
by the byte-code compiler.  Hence, for these checks, all executing code is
byte-compiled: packages are pre-compiled via R_COMPILE_PKGS=1 and
R_JIT_STRATEGY=4 is set to ensure that all code is byte-compiled by the JIT,
even when it is expected to harm performance.  Then, package tests are run
with R_CHECK_CONSTANTS=5, which is the most strict but also slowest level of
checking the constant corruption.  These tests can be easily re-run by
developers as they use an unmodified version of R.

A package may fail with rcnst but pass in normal checks for several reasons. 

The obvious one is constants corruption due to a bug in a package (possibly
in a dependency, not always in the package that is tested); when constants
corruption is detected, R is terminated and a detailed report is printed to
the output (look for "were modified").

Another possibility is a bug in the package that prevents it from running
with the byte-code compiler/interpreter - because in normal checks, the
default JIT heuristics are used, and hence code that is not deemed to
benefit from byte-code compilation is not compiled.  One can re-run tests
for the package with byte-code compiler disabled (e.g.  R_ENABLE_JIT=0 and
packages not compiled).  User code should not depend on implementation
details of the R runtime that differ between the AST interpreter and the
byte-code compiler/interpreter, but instead should be able to run in both. 

Sometimes the byte-code compiler or the registration of constants "wakes up"
old bugs that are not really related to interpretation, but are "woken up"
by the gc running at other times.  Often these are PROTECT bugs.  Sometimes
but not always these bugs can be also provoked with the base configuration
but GCTORTURE enabled, which is a way of proving this is the case. Sometimes
the diagnosis is harder.

The failure can also be random an independent on constants
checking/byte-code compilation, such as a temporary problem in network
connection, etc. The outputs should help to identify these causes.

Detailed information on how to interpret the reports on corrupted constants
is available at
https://github.com/kalibera/cran-checks/blob/master/rcnst/CONSTANTS.md
