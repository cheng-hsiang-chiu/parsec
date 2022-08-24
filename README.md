# PARSEC Benchmark 

Download the compressed file from PARSEC's official website, http://parsec.cs.princeton.edu/download/3.0/parsec-3.0.tar.gz

## Building ferret of the gcc-tbb version
Modifications are needed to fix the issues,
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


## Building dedup (no tbb implementation on PARSEC)
Modification is needed to fix the issues,
1. Adopt item 1 from the link, https://yulistic.gitlab.io/2016/05/parsec-3.0-installation-issues/


## Building x264 (no tbb implementation on PARSEC)
Modifications are needed to fix the issues,
1. In src/common/common.h, change the declarations of `dequant4_mf` and `dequant8_mf` to the followings,
```
//int (*dequant4_mf[4])[4][4]; /* [4][6][4][4] */
//int (*dequant8_mf[2])[8][8]; /* [2][6][8][8] */
int   (*dequant4_mf[4])[16];   /* [4][6][16]   */                                                                 
int   (*dequant8_mf[2])[64];   /* [2][6][64]   */
```
2. In src/common/set.c, change the declarations of `quant4_mf` and `quant8_mf` to the followings,
```
//int quant4_mf[4][6][4][4];  
//int quant8_mf[2][6][8][8];
int quant4_mf[4][6][16];
int quant8_mf[2][6][64];
```
3. In src/common/set.c, change the first `for loop` to the following in the function `x264_cqm_init( x264_t *h )`,
```
#define QP_MAX_SPEC 51
#define CQM_ALLOC( w, count )\
    for( int i = 0; i < count; i++ )\
    {\
        int size = w*w;\
        int start = w == 8 ? 4 : 0;\
        int j;\
        printf("i = %d, j = %d, size = %d, start = %d, w = %d\n", i, j, size, start, w); \
        for( j = 0; j < i; j++ )\
            if( !memcmp( h->pps->scaling_list[i+start], h->pps->scaling_list[j+start], size*sizeof(uint8_t) ) )\
                break;\
        if( j < i )\
        {\
            h->  quant##w##_mf[i] = h->  quant##w##_mf[j];\
            h->dequant##w##_mf[i] = h->dequant##w##_mf[j];\
            fprintf(stderr, "h->dequant%d_mf[%d] is set\n", w, i); \
            h->unquant##w##_mf[i] = h->unquant##w##_mf[j];\
        }\
        else\
        {\
            h->  quant##w##_mf[i] = x264_malloc((QP_MAX_SPEC+1)*size*sizeof(uint16_t));\
            h->dequant##w##_mf[i] = x264_malloc(6*size*sizeof(int));\
            fprintf(stderr, "h->dequant%d_mf[%d] is set\n", w, i); \
            h->unquant##w##_mf[i] = x264_malloc((QP_MAX_SPEC+1)*size*sizeof(int));\
        }\
        for( j = 0; j < i; j++ )\
            if( deadzone[j] == deadzone[i] &&\
                !memcmp( h->pps->scaling_list[i+start], h->pps->scaling_list[j+start], size*sizeof(uint8_t) ) )\
                break;\
        if( j < i )\
        {\
            h->quant##w##_bias[i] = h->quant##w##_bias[j];\
        }\
        else\
        {\
            h->quant##w##_bias[i] =x264_malloc((QP_MAX_SPEC+1)*size*sizeof(uint16_t));\
        }\
    }

    CQM_ALLOC( 4, 4 )
    CQM_ALLOC( 8, 2 )
```
