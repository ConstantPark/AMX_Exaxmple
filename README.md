# Introduction of AMX Applications
Intel의 AMX (Matrix Instruction)을 사용하는 방법은 2가지가 존재합니다 (AMX Inst API 호출 또는 oneAPI-oneMKL 사용).  
LLM의 다양한 GEMM/GEMV에 맞춰서 매번 커널을 짜고 최적화 할수 없기 때문에 보통 BLAS API를 사용합니다.  
최근 공개된 MLPerf Inference Inference Benchmark v3.1 (GPTJ-99)는 SPR의 AMX를 사용하여 추론을 수행합니다.  
본 문서는 AMX를 사용할 수 있는 oneMKL (BLAS)를 사용하여 GEMM 연산을 수행하는 코드에 해당됩니다.  

코드는 다음의 링크에서 다운 받는 것이 가능합니다. 
(https://github.com/oneapi-src/oneAPI-samples/tree/master/Libraries/oneMKL/matrix_mul_mkl)  

# Intel oneMKL with GEMM Benchmark Install

MLPerf Inference Benchmark v3.1 Setting
Intel oneMKL 및 MLPerf Inference Benchmark v3.1 Setting Guide
