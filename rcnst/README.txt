These checks detect incorrect modifications of R constants.  To reproduce,
run a package check with these enviroment variables set in R 4.x or newer:

env R_COMPILE_PKGS=1 R_JIT_STRATEGY=4 R_CHECK_CONSTANTS=5 R CMD check p.tar.gz

The CRAN checks are run with R-devel.  Expect a significant slow-down
compared to regular execution.

Constants here mean R objects flagged by macro MARK_NOT_MUTABLE [2]. 
Immutability is enforced in R code, but cannot be enforced in C/C++, and
thus incorrect C/C++ package code may modify (corrupt) constants.

The consequences are unpredictable.  The program may crash, produce an
incorrect result or run normally, depending on where the corrupted constant
happens to be used next.  Commonly constant corruptions leads to that all
constant literals such as number 1 will change their value to say 0 inside
an R function.

These bugs often originate from C/C++ code in packages that modify input
arguments of a function called via .Call.  Such a modification is a
violation of R semantics: almost always one has to duplicate an argument and
change only its copy, as detailed in [1].

The checks are based on run-time detection of corruption of constants used
by the byte-code compiler.  Hence, for these checks, all executing code is
byte-compiled.  To ensure packages are pre-compiled, R_COMPILE_PKGS=1 is
set, even though this is already the default.  To ensure other code is
byte-compiled even when that is not beneficial for performance,
R_JIT_STRATEGY=4 is set.  Finally, R_CHECK_CONSTANTS=5 is set to enable
checks of corrupted compiler constants, where 5 is the strictest but also
the slowest level of checking.

Only packages that failed due to constants corruption are shown here. 
Packages that failed for other reasons were excluded.  These checks cannot
produce false alarms, but a package may fail due to constants corruption
caused by some of its dependencies.  See [1] for more details on how to
interpret outputs from these checks.

[1] https://github.com/kalibera/cran-checks/blob/master/rcnst/CONSTANTS.md

[2] https://cran.r-project.org/doc/manuals/r-release/R-exts.html#Named-objects-and-copying
    Writing R Extensions, section 5.9.10
