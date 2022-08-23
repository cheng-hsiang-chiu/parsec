# PARSEC Benchmark 

Download the compressed file from PARSEC's official website, http://parsec.cs.princeton.edu/download/3.0/parsec-3.0.tar.gz

Modifications are needed to fix the issues while building ferret of the gcc-tbb version,
1. Adopt item 4 from the link, https://yulistic.gitlab.io/2016/05/parsec-3.0-installation-issues/
2. In "pkgs/libs/gsl/parsec/gcc.bldconf", add "-lstdc++" in the variable `build_env`, such that 
```
build_env="LIBS=\"${LIBS} -lstdc++ -lm\""
```
3. In "config/gcc-tbb.bldconf", move "-ltbb" to the right of "-ltbbmalloc" in the variable `TBB_LIBS`, such that 
```
TBB_LIBS="-ltbbmalloc_proxy -ltbbmalloc -ltbb -ldl -lrt -lpthread"
```
4. In "config/gcc.bldconf", add "-std=c++11" in `CXXFLAGS`, such that 
```
export CXXFLAGS="-O3 -std=c++11 -g -funroll-loops -fprefetch-loop-arrays -fpermissive -fno-exceptions ${PORTABILITY_FLAGS}"
```
5. In "config/gcc.bldconf", add "-lstdc++" in `LDFLAGS`, such that 
```
export LDFLAGS="-lstdc++ -L${CC_HOME}/lib64 -L${CC_HOME}/lib"
```
