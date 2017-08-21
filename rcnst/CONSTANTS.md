
## Value semantics of R

WARNING: this text ignores some details in the interest of readability

R has value semantics for almost all its objects, which means that arguments
are passed to functions by value.  Any modification of an argument in an R
function is executed on a copy of the original object.  Hence also if a
modified version of such object is needed outside of the function, it has to
be returned e.g.  as the return value of that function.  In an R program the
copying is done automatically by the R runtime and the user does not have to
worry and cannot prevent it from happening.  Environments are an exception,
they are passed by reference, and hence any modification made inside a
function to an environment passed in as argument will be visible
immediately outside such function.  The same applies to assignment, in `x
<- y`, the assignment is "by value", hence it is equivalent to a copy for
most R objects (environments are an exception).

As an optimization, R does not eagerly copy objects before every assignment
or before every function call, but it maintains a counter for each R object,
a.k.a `NAMED` value.  The counter counts the number of "names" (references,
bindings) to the R object.  If the count is 0, the R object is not
referenced and hence can be modified in place without damaging value
semantics.  If the count is 1, in-place modification is only possible in
special cases such as `x[1] <- 10`, that is if we know that the only "name"
of the object in existence will be immediately re-bound (so the old version
of the object won't be accessible anymore, and hence won't be needed).  In
all other cases, the R runtime needs to copy the object and modify its copy. 
Currently `NAMED` value only goes up to 2 and neither increases nor
decreases as it reaches 2.  More details can be found in [Writing R
Extensions](https://cran.r-project.org/doc/manuals/r-release/R-exts.html#Named-objects-and-copying)

## Maintaining NAMED value in C code

In C code of R runtime and packages, the rules above have to be enforced
manually.  One has to copy an R object before modifying it unless its
`NAMED` value is 0 (which is rarely the case in practice).  R optimizes code
based on the assumption that this rule is enforced.  A violation may cause a
crash or lead to incorrect results.

A freshly allocated object in C via `allocVector` or `CONS` or other
allocating function has `NAMED` value of 0, so it is safe to modify that
object and to return it as a return value, say from a .Call function. So
C/C++ functions that allocate a new object, but do not bind it to any R
variable, are free to modify the object prior to returning it.

Input arguments of .Call functions, however, cannot be modified in place
unless they have `NAMED` value of 0, which is rare. If modification is
needed, the safe and simple way is to duplicate the corresponding argument
unconditionally, modify the copy and return it. It is therefore a bad idea
to allocate an object in R and then pass it to C/C++ to be filled in - an
extra copy would often be needed.

In some cases one needs referential semantics for arguments.  This is
possible with environments, but one has to be careful about that elements of
the environments are also R objects and the same rule applies.  It is ok to
add a variable to an environment, to delete a variable, or to change a
binding of a variable to point to a different object.  One however cannot
change an object bound to some variable in such environment.  One instead
has to copy that object, modify the copy, and then change (in-place) the
binding in the environment to point to the new copy.

## Checking for constants corruption

The checks target modification of compiler constants.  All compiler
constants have `NAMED` value of 2, and hence all reported modifications are
true errors.  Many true errors will be missed, partly due to limited
coverage of package tests, partly because only compiler constants are being
checked.  Authors/maintainers of reported packages should thus ensure
manually that their package does not have additional errors similar to those
reported.  Sometimes the package reported is not the package at fault --
please contact the maintainer of the package at fault in such cases (this is
usually easy to see from the report).

The checking is based on keeping copies of the constants and repeatedly
comparing their actual values to the copies. To make the diagnosis easier,
it also specifically checks for `.Call` functions that modify their
arguments in place, because this is a common cause of the error. If a
corrupted constant is found, a verbose report is produced and R is
terminated. Excerpt from a report:

```
ERROR: modification of compiler constant of type character, length 1
ERROR: the modified value of the constant is:
[1] "2\t1364 ...
ERROR: the original value of the constant is:
[1] ""
ERROR: the modified constant is at index 20
ERROR: the modified constant is in this function body:
{
filename <- path.expand(filename)
xx <- vcf_open(filename)
...
Function read.vcf in namespace gaston has this body.
ERROR: detected compiler constant(s) modification after .Call invocation of function VCF_readLineRaw from library WhopGenome (/path/WhopGenome.so).
NOTE: .Call function VCF_readLineRaw modified its argument
(number 2, type character, length 1)
Fatal error: compiler constants were modified (in .Call?)!
```

When a `.Call` function is mentioned, such as in this case, the diagnosis is
very easy (here the function at fault is `VCF_readLineRaw` from
`WhopGenome`.  One can search the outputs for "were modified", which is part
of the message of the fatal error which terminates R to make sure the
corruption is definitely spotted.
