# Introduction of AMX Applications
Intel의 AMX (Advanced Matrix Instruction)는 AMX 명령어를 직업 호출 하는 API를 사용하거나, oneAPI-oneMKL를 호출하여 사용 가능합니다. 
LLM의 다양한 크기와 모양의 GEMM/GEMV 모양에 맞춰 커널을 작성하고 최적화 하는 작업을 매번 할 수 없기 때문에 보통 BLAS API (oneMKL)을 사용합니다. 
최근 공개된 MLPerf MLPerf Inference Benchmark v3.1 (GPTJ-99)에 제출된 결과중 사피이어 래피즈 (SPR)은 AMX를 사용하여 추론을 수행합니다. 
본 문서는 AMX를 사용할 수 있도록, oneMKL (BLAS-GEMM)과 MLPef Inference v3.1 세팅 방법을 설명합니다. 
GEMM의 동작을 확인하여 컴퓨팅 성능을 확인하였고, MLPerf는 Troubleshooting 작업 진행중입니다.

# Intel oneMKL with GEMM Benchmark Install
GEMM Benchmark를 사용하기 위해서는 먼저 oneMKL의 사용이 가능한 환경이 세팅되어야 하며, 인텔 컴파일러와 oneMKL 설치가 반드시 사전에 수행되어야 합니다. 
아래의 링크를 참조하여 인텔 컴파일러와 oneMKL을 먼저 설치하며, 이후 아래의 코드를 bashrc에 적용하면 별도의 세팅없이 편하게 사용이 가능합니다.

## 인텔 컴파일러 및 GEMM 코드 다운로드
```
컴파일러/MKL: https://www.intel.com/content/www/us/en/developer/tools/oneapi/hpc-toolkit.html#gs.6lqnvh   
GEMM 코드: https://github.com/oneapi-src/oneAPI-samples/tree/master/Libraries/oneMKL/matrix_mul_mkl   
```
## Bash shell에 적용하는 코드
```
# oneAPI Install
source /opt/intel/oneapi/setvars.sh --force
source /opt/intel/oneapi/mkl/2023.2.0/env/vars.sh
```

설치가 완료된 이후 소스코드 위치 (matrix_mul_mkl.cpp)에서 컴파일 (make)을 수행하면 컴파일 결과로 sgemm, dgemm binary를 얻을 수 있습니다.
```
# 2소켓 CPU 기준으로 코어가 112이기 때문에 그에 해당되는 스레드를 생성하여 컴파일을 수행합니다.
make -j 112 
```
컴파일의 결과를 수행하면 FLOPS 정보를 얻는 것이 가능하며, 이때 Device에 출련된 내용을 확인하여야 합니다 (ES 샘플 제품이라 0000로 출력됨). 
```
Problem size:  A (8000x8000) * B (8000x8000)  -->  C (8000x8000)
Benchmark interations: 100
Device: Genuine Intel(R) 0000
Launching oneMKL GEMM calculation...
SGEMM performance : 8578.78 GFLOPS
```
## FP32 및 BF16 측정 코드 
인텔에서 제공하는 oneMKL를 사용하는 GEMM 성능을 측정하는 코드는 Square Matrix를 계산하며 행렬의 크기, 연산에 사용되는 데이터 타입을 변경 할 수 있습니다.
소스 코드에서 FLOAT를 정의하는 부분을 주석처리하고, float 또는 bfloat16에 따라 FLOAT의 타입을 정의합니다.
FP32로 사용할 경우 float로 정의, BF16을 사용할 경우 oneapi~를 정의합니다. 행렬의 크기는 MSIZE에 값을 입력하며, 이후 ```make clean, make -j 112```를 수행합니다.
```
/*
#ifndef USE_DOUBLE
#define FLOAT   float
#else
#define FLOAT   double
#endif
*/
#define FLOAT float // FP32
#define FLOAT  oneapi::mkl::bfloat16 // BF
#define MSIZE  8000 // ORG is 8192
```
인텔에서 공식적으로 배포한 GEMM 결과 (https://url.kr/ugxz15)와 동일한 크기의 행렬 크기를 정의하여 성능을 확인하고 정리하여 말씀하여 주시면 감사하겠습니다.

# MLPerf Inference v3.1 Benchmark
MLperf는 딥러닝의 Inference/Training을 평가하는 Benchmark로 최근 공개된 v3.1 버전은 LLM (GPT-J)에 관한 성능 측정 기능이 추가되었습니다. 
NVIDIA, Intel 등 HW 벤더들이 각 하드웨어에서 성능 테스를 진행하고, 그때 사용된 코드와 성능 결과를 올려 공개하고 있습니다.
올해 8월 초에 SPR (Sapphire Rapids)에서 BF16/Int8로 추론을 했을때 얻을 수 있는 성능을 공개하였습니다. 이러한 
MLPerf Benchmark를 사용하기 위해서는 코드 다운로드와 MLPerf와 관련된 추가적인 설치가 사전에 필요합니다.
사용을 위해서는, MLPef의 LoadGen을 먼저 설치해야 하며 이것은 특정 입력/출력 길이에 대한 Workload를 생성하는 기능에 해당됩니다.

<img src = https://raw.githubusercontent.com/mlcommons/inference/master/loadgen/loadgen_integration_diagram.svg width = "80%" height = "80%">  

## MLPerf Inference v.31 Benchmark Link
```
MLPerf 설명자료: http://www.kibme.org/resources/journal/20200206100621546.pdf
성능 Board: https://mlcommons.org/en/inference-datacenter-31/
MLPerf (LoadGen) 설치: https://github.com/mlcommons/inference/blob/master/loadgen/README_BUILD.md
MLPerf 코드: https://github.com/mlcommons/inference_results_v3.1
```

## Install 방법 with Bare Metal and Docker build
설치 순서는 LoadGen, MLPerf Inference benchmark 순으로 수행 되어야 합니다.
LoadGen의 설치는 위의 링크를 참조하여 진행가능하며, 세팅 과정에서 native 환경에서 설정을 진행하였습니다.
MLPerf는 inference_results_v3.1/closed/Intel/code/gptj-99/pytorch-cpu 폴더에 GPT-J의 성능 평가를 수행할 수 있는 코드가 있습니다.
해당 코드는 Docker build 또는 Bare Metal로 설치와 사용이 가능하지만, 직접 설치 했을때는 둘다 문제가 발생하였습니다.
발생된 문제의 Log는 다음과 같으며 원인은 현재 파악중에 있습니다설명드신 순서에 맞춰서 진행을 부탁드립니다.

## Docker Build 설치시 발생하는 Log
```
1113.4 ERROR: No matching distribution found for torch==2.1.0.dev20230711+cpu
------
Dockerfile_int4:43
--------------------
  42 |     ARG PYTHON_VERSION=3.9
  43 | >>> RUN curl -fsSL -v -o ~/miniconda.sh -O  https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh  && \
  44 | >>>     chmod +x ~/miniconda.sh && \
  45 | >>>     ~/miniconda.sh -b -p /opt/conda && \
```
MLPerf에 올라간 결과는 앞서 말씀드린 것처럼 BF16/Int8로 추론의 결과입니다. 하지만 도커 파일은 현재 Int8/Int4가 올라와 있는 상태입니다. 
보통 MLPerf에 올린 결과만 보기 때문에, 코드 관리가 잘 되는 편은 아닌것으로 생각되며, Int8/Int4 모두 설치 과정에서 문제가 발생하는 것을 확인하였습니다.
문제의 원인은 Docker Makefile 내의 torch가 현재 없는 버전이라 그렇습니다. 다른 유사 버전으로 변경하면 다른 문제가 발생하는 것을 확인하였습니다.
따라서, Docker 보다는 Bare Metal로 설치하는 것이 현재 가능한 방법이라는 생각이 들었습니다.

## Bare Metal 설치시 발생하는 Log
```
ERROR: ld.so: object '/home/smrc/miniconda3/lib/libtcmalloc.so' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.
ERROR: ld.so: object '/home/smrc/miniconda3/lib/libiomp5.so' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.
ERROR: ld.so: object '/home/smrc/miniconda3/lib/libtcmalloc.so' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.
ERROR: ld.so: object '/home/smrc/miniconda3/lib/libiomp5.so' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.
ERROR: ld.so: object '/home/smrc/miniconda3/lib/libtcmalloc.so' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.
Traceback (most recent call last):
  File "/home/smrc/inference_results_v3.1/closed/Intel/code/gptj-99/pytorch-cpu/runner.py", line 6, in <module>
    from SUT import SUT
  File "/home/smrc/inference_results_v3.1/closed/Intel/code/gptj-99/pytorch-cpu/SUT.py", line 14, in <module>
    import thread_binder
ModuleNotFoundError: No module named 'thread_binder'
ERROR: ld.so: object '/home/smrc/miniconda3/lib/libiomp5.so' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.
ERROR: ld.so: object '/home/smrc/miniconda3/lib/libtcmalloc.so' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.
cp: cannot stat './offline-output-DSOL-CRB-Test-01-batch-14-procs-4-ins-per-proc-2-10-05-03-04/*': No such file or directory
```
Bare Metal을 사용한 설치 방법도 Docker와 마찬가지로 Int8/Int4만 공개되었습니다. 내부 코드에 BF16을 지원하는 코드가 있으나 설명은 없습니다.
BF16, Int4/Int8 모두 설치과정에서 오류가 발생하였으며, 해당 오류가 ES 샘플 제품으로 인한 것인지 아니면 다른 것인지 현재 확인 중입니다.
앞서 설명드린 LoadGen의 SUT라는 Package에서 현재 오류가 발생하며, thread_bind라는 라이브러리를 찾지 못하고 있습니다.
해당 패키지는 구글링, MLerf Github를 찾았을때 없는 이슈이며, OpenMP5 라이브러리를 사용하는 과정에서 문제가 발생한 것으로 추측하고 있습니다. 
문제 해결을 위해서 OpenMP5 라이브러리를 따로 설치하고 다시 설치를 하였으나 동일한 문제가 계속 발생하고 있습니다. 


Docker, Bare Metal을 사용한 방법 모두 해보고, 결과 알려주시면 감사하겠습니다.

