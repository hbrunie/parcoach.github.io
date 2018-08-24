# PARCOACH  (PARallel COntrol flow Anomaly CHecker) 

PARCOACH automatically checks parallel applications to verify the correct use of collectives.
This is PARCOACH v3.


----------------------------- Version ------------------------------

PARCOAH v1 : Intra-procedural control flow analysis 
PARCOACH v2 : Summary-based control flow analysis
PARCOACH v3 : Inter-procedural control- and data-flow analysis



----------------------------- Prerequisites ------------------------------

---- Cmake

PARCOACH uses CMake. CMake can be downloaded from http://www.cmake.org

---- LLVM 3.9.1

This version of PARCOACH is an LLVM pass. It works with LLVM 3.9.1
To get LLVM 3.9.1, follow these steps:

-> compiling LLvm requires that you have installed GNU CMAKE,
GCC (>=4.7.0), python (>=2.7), libtool (version 1.5.22) and zlib (>=1.2.3.4)

svn co http://llvm.org/svn/llvm-project/llvm/tags/RELEASE_391/final llvm

cd llvm/tools
svn co http://llvm.org/svn/llvm-project/cfe/tags/RELEASE_391/final clang

cd clang/tools/
svn co http://llvm.org/svn/llvm-project/clang-tools-extra/tags/RELEASE_391/final extra

cd ../../../projects/
svn co http://llvm.org/svn/llvm-project/openmp/tags/RELEASE_391/final openmp
svn co http://llvm.org/svn/llvm-project/compiler-rt/tags/RELEASE_391/final compiler-rt
svn co http://llvm.org/svn/llvm-project/libcxx/tags/RELEASE_391/final libcxx
svn co http://llvm.org/svn/llvm-project/libcxxabi/tags/RELEASE_391/final libcxxabi

cd ../..
mkdir build && cd build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=PATH_TO_LLVM/build -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ ../llvm
make -j4
make install
make omp


---- Berkeley UPC

The installation is done in two steps:
(1) Download and install a UPC-to-C translator or UPC compiler
(2) Download and build the UPC runtime

You must build BUPC with LLVM 3.9

(1)
cd berkeley_upc_translator-2.22.0
mkdir build
CC=clang CXX=clang++ make
make install PREFIX=PATH_TO_BUPC/berkeley_upc_translator-2.22.0/build
(2)
cd berkeley_upc-2.22.0
mkdir build && cd build
../configure --with-multiconf --enable-mpi CC=clang CXX=clang++ BUPC_TRANS=PATH_TO_BUPC/berkeley_upc_translator-2.22.0/build/targ --prefix=PATH_TO_BUPC/berkeley_upc-2.22.0/build
make
make install


See http://upc.lbl.gov/download/  for more details


---- MPI

No particular version of MPI is required. Use a version handling nonblocking collective if you want to check them.

---- OpenMP

We use the OpenMP from LLVM



------------- How to build and use Parcoach ------------

- Build Parcoach library:

cd PARCOACH_PATH/
mkdir build && cd build
cmake .. && make


- Use Parcoach:

Codes with errors can be found in the tests directory.
For runtime checking, PARCOACH needs to be linked to its dynamic library.

-> Example of the use of Parcoach with test.c :

/* CREATE dynamic library DynamicCheck.o  */
clang -g  -c $PATH_TO_DYN_LIBRARY/{MPI/UPC/OMP}_DynamicCheck.c

/* CREATE test bitcode  */
clang -g -emit-llvm -c -o test.bc test.c
/* MERGE bit codes if multiple files */
llvm-link tes1.bc test2.bc -o merge.bc

/* PARCOACH PASS */
opt -load $PATH_TO_PARCOACH_LIB/LLVMParcoach.dylib -postdomtree -parcoach [OPTIONS] < test.bc > ./test_optimized.bc
[OPTIONS] can be -loweratomic (needed for OpenMP codes), -dot-depgraph, -strong-update, -no-reg-name
Replace test.bc by merge.bc if you have multiple files

/* CREATE test_optimized.o  */
clang -c test_optimized.bc

/* LINK */
clang test_optimized.o {MPI/UPC/OMP}_DynamicCheck.o -o test $(LDFLAGS)

