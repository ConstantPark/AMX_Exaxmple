# Introduction of AMX Applications
Intel의 AMX (Advanced Matrix Instruction)는 AMX 명령어를 직업 호출 하는 API를 사용하거나, oneAPI-oneMKL를 호출하여 사용 가능합니다. 
LLM의 다양한 크기와 모양의 GEMM/GEMV 모양에 맞춰 커널을 작성하고 최적화 하는 작업을 매번 할 수 없기 때문에 보통 BLAS API (oneMKL)을 사용합니다. 
최근 공개된 MLPerf MLPerf Inference Benchmark v3.1 (GPTJ-99)에 제출된 결과중 사피이어 래피즈 (SPR)은 AMX를 사용하여 추론을 수행합니다. 
본 문서는 AMX를 사용할 수 있도록, oneMKL (BLAS-GEMM)과 MLPef Inference v3.1 세팅 방법을 설명합니다. 
GEMM의 동작을 확인하여 컴퓨팅 성능을 확인하였고, MLPerf는 Troubleshooting 작업 진행중입니다.

# Intel oneMKL with GEMM Benchmark Install
GEMM Test는 다음의 코드를 사용하여 성능 평가 가능하며, AMX를 사용하기 위해서는 코드의 일부 수정이 필요합니다. 
먼저 GEMM Test를 위해서는 인텔 컴파일러와 oneMKL 설치가 반드시 사전에 수행되어야 합니다.  
```
컴파일러/MKL 링크 (https://www.intel.com/content/www/us/en/developer/tools/oneapi/hpc-toolkit.html#gs.6lqnvh)  
소스코드 링크 (https://github.com/oneapi-src/oneAPI-samples/tree/master/Libraries/oneMKL/matrix_mul_mkl)  
```
설치가 완료된 이후 소스코드 위치 (matrix_mul_mkl)을 수행하면 컴파일 결과로 sgemm, dgemm binary를 얻을 수 있습니다.
	1. 간결하다.
	2. 별도의 도구없이 작성가능하다.
	3. 다양한 형태로 변환이 가능하다.
	4. 텍스트(Text)로 저장되기 때문에 용량이 적어 보관이 용이하다.
	5. 텍스트파일이기 때문에 버전관리시스템을 이용하여 변경이력을 관리할 수 있다.
	6. 지원하는 프로그램과 플랫폼이 다양하다.
MLPerf Inference Benchmark v3.1 Setting
Intel oneMKL 및 MLPerf Inference Benchmark v3.1 Setting Guideasdasd



s
