This is a proof of concept demonstrating how to add gmp and mpfr to a cmake
build project via `ExternProject_add`. Specifically, this builds static
libraries and borrows configuration flags and patches from the homebrew builds
of gmp and mpfr for macs.

gmp and mpfr are easy to install using homebrew:

```bash
brew install gmp mpfr
```

Alternatively, build them inside this project using:

```bash
mkdir build
cd build
cmake ../
make
```

To build x86_64 architecture files (e.g., for using Rosetta on a mac m1), you
could use

```bash
mkdir build
cd build
cmake ../ -DCMAKE_OSX_ARCHITECTURES=x86_64
arch -x86_64 make
```

gmp and mpfr sources will be downloaded on the first call to `make` so an
internet connection is necessary at (first) build time.
