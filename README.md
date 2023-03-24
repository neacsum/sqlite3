# SQLite3 Build Package
This is a small package that creates static and shared library versions from the latest release of SQLITE. Current version is 3.41.2.

Note that this package does not contain the SQLITE source code; it only downloads the "official" amalgamation and creates the library files and the SQLITE3 shell program.

## Building
Use the [CPM package manager](https://github.com/neacsum/cpm) and CMake. Use the following command to build all libraries and shell:
````
cpm sqlite3
````