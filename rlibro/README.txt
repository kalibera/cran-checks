Checks with read-only library directory (the directory where packages are
installed).  Packages should never attempt to write into the directory where
they have been installed.  This restrictions is primarily for security
reasons and for transparency when the library is to be shared by multiple
users.  Packages sometimes violate this restriction and attempt to save test
results or download data into their installation directory.  This is
undetected on personal installations of R where the library is usually
writable, but leads to crashes on installations where the library is mounted
read-only.

These checks are just regular package checks run with the library mounted
read-only. There are no false alarms (check outputs are searched for error
messages related to writing to a read-only file systems), but in principle a
package may be reported while its dependency is at fault. Also, some true
errors may be missed.
