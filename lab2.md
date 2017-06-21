#OpenCL实验二-矩阵乘法

###1.首先编译运行hello.cpp以及vadd.cpp，观察得到的结果

运行结果如下图所示：

![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab2-1.png)

如图所示，运行hello.cpp的结果为在终端输出"Hello World!"的句子，运行vadd的文件的结果，输出了OpenCL与cpu两种方法执行向量加法的时间，发现这两种方法的执行时间是不相同的。

在向量长度VECSIZE为6的情况下，串行执行时间为0.001ms，而OpenCL执行时间为0.017ms，可以看出OpenCL执行时间远远超过了串行执行时间。

我们接着修改VECSIZE的值，观察变化：

我们把VECSIZE的值改为60，结果如下：

![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab2-2.png)

我们再多进行机几组实验，继续改大VECSIZE的数值，将VECSIZE的值增加到600，6000，60000，结果分别如下：

![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab2-3.png)

![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab2-4.png)

![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab2-5.png)

通过这些结果，我们发现了一些规律。在VECSIZE的数值比较小的时候，OpenCL的执行时间远远大于串行执行时间，不过在VECSIZE变得越来越大的时候，串行执行时间逐渐的超过了OpenCL的执行时间，所以我们可以得到，在处理一些大数据的时候，OpenCL的执行有着很大的优势。

###2矩阵乘法

仿照vadd.cpp自己写了计算矩阵乘法的程序，分别用CPU和OpencCL的执行方式记录运行时间，通过改变W大小来改变执行时间

我们先令W为2，来看执行时间，结果如下：

![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab2-6.jpg)

我们再将W改为20，再运行程序，结果如下：

![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab2-7.jpg)

从结果可以看出，OpenCL执行有着时间上的明显优势。

kernel.cl
```
__kernel 
void matrix_mult(
    const int Ndim,
    const int Mdim,
    const int Pdim,
    __global const float* A, 
    __global const float* B, 
    __global float* C)
{
    int i = get_global_id(0);
    int j = get_global_id(1);
    int k;
    float tmp;
    if ((i < Ndim) && (j < Mdim)) {
        tmp = 0.0;
        for (k = 0; k < Pdim; k++)
            tmp += A[i*Pdim + k] * B[k*Mdim + j];
        C[i*Mdim + j] = tmp;
    }
}

```

Matrix.cpp
```
	#include <CL/cl.h>
	#include <stdio.h>
	#include <stdlib.h>
	#include <time.h>
	#include <iostream>
	#include <fstream>
	#include <cmath>
	#include <cstdio>
	#include <cstdlib>
	#include <cstring>
	#include <ctime>
	
	using namespace std;
	
	#define NWITEMS 6
	#define W 2
	#pragma comment (lib,"OpenCL.lib")
	
	//把文本文件读入一个 string 中
	int convertToString(const char *filename, std::string& s)
	{
	    size_t size;
	    char* str;
	    std::fstream f(filename, (std::fstream::in | std::fstream::binary));
	    if (f.is_open())
	    {
	        size_t fileSize;
	        f.seekg(0, std::fstream::end);
	        size = fileSize = (size_t)f.tellg();
	        f.seekg(0, std::fstream::beg);
	        str = new char[size + 1];
	        if (!str)
	        {
	            f.close();
	            return NULL;
	        }
	        f.read(str, fileSize);
	        f.close();
	        str[size] = '\0';
	        s = str;
	        delete[] str;
	        return 0;
	    }
	    printf("Error: Failed to open file %s\n", filename);
	    return 1;
	}

	int main()
	{

	    cl_mem memObjects[3] = { 0, 0, 0 };
	    memObjects[0] = clCreateBuffer(context, CL_MEM_READ_ONLY |  CL_MEM_COPY_HOST_PTR,
	        sizeof(float)* szA, A, NULL);
	    memObjects[1] = clCreateBuffer(context, CL_MEM_READ_ONLY |  CL_MEM_COPY_HOST_PTR,
	        sizeof(float)* szB, B, NULL);
	    memObjects[2] = clCreateBuffer(context, CL_MEM_READ_WRITE | CL_MEM_COPY_HOST_PTR,
	        sizeof(float)* szC, C, NULL);
	    if (memObjects[0] == NULL || memObjects[1] == NULL ||memObjects[2] == NULL) 
	        perror("Error in clCreateBuffer.\n");
	
	    const char * filename = "kernel.cl";
	    std::string sourceStr;
	    status = convertToString(filename, sourceStr);
	    if (status)
	        cout << status << "  !!!!!!!!" << endl;
	    const char * source = sourceStr.c_str();
	    size_t sourceSize[] = { strlen(source) };
	    
	    cl_program program = clCreateProgramWithSource(
	        context,
	        1,
	        &source,
	        sourceSize,
	        NULL);
	    
	    status = clBuildProgram(program, 1, &device, NULL, NULL, NULL);
	    if (status)
	        cout << status << "  !!!!!!!!" <<endl;
	    if (status != 0)
	    {
	        printf("clBuild failed:%d\n", status);
	        char tbuf[0x10000];
	        clGetProgramBuildInfo(program, device, CL_PROGRAM_BUILD_LOG, 0x10000, tbuf,
	            NULL);
	        printf("\n%s\n", tbuf);
	        //return −1;
	    }
	
	    
	    cl_kernel kernel = clCreateKernel(program, "matrix_mult", NULL);
	
	    
	    cl_int clnum = NWITEMS;
	    status = clSetKernelArg(kernel, 0, sizeof(int), &Ndim);
	    status = clSetKernelArg(kernel, 1, sizeof(int), &Mdim);
	    status = clSetKernelArg(kernel, 2, sizeof(int), &Pdim);
	    status = clSetKernelArg(kernel, 3, sizeof(cl_mem), &memObjects[0]);
	    status = clSetKernelArg(kernel, 4, sizeof(cl_mem), &memObjects[1]);
	    status = clSetKernelArg(kernel, 5, sizeof(cl_mem), &memObjects[2]);
	    if (status)
	        cout << "参数设置有误" << endl;
	
	    size_t global[2];
	    cl_event prof_event;
	    cl_ulong ev_start_time = (cl_ulong)0;
	    cl_ulong ev_end_time = (cl_ulong)0;
	    double gputime;
	    global[0] = (size_t)Ndim;
	    global[1] = (size_t)Mdim;
	    status = clEnqueueNDRangeKernel(commandQueue, kernel, 2, NULL,
	             global, NULL, 0, NULL, &prof_event);
	    if (status)
	        cout << "执行内核时错误" << endl;
	    clFinish(commandQueue);
	
	    status = clGetEventProfilingInfo(prof_event,CL_PROFILING_COMMAND_START,
	        sizeof(cl_ulong),&ev_start_time,NULL);
	    status = clGetEventProfilingInfo(prof_event,CL_PROFILING_COMMAND_END,
	        sizeof(cl_ulong),&ev_end_time,NULL);
	    if (status) 
	        perror("读取时间的时候发生错误\n");
	    gputime = (double)(ev_end_time - ev_start_time)* 1e-6;
	    printf("OpenCL执行时间：%8.3f ms\n", gputime);
	
	
	    clock_t timestamp;
	    float *D;
	    D = (float *)malloc(szC * sizeof(float));
	    timestamp = clock();
	    for(int i =0 ;i<Ndim;i++)
	    {
	       
	        for(int j=0;j<Mdim;j++)
	        {
	            float tmp = 0;
	            for(int t=0;t<Pdim;t++)
	            tmp = tmp + A[i*Pdim + t] * B[t*Pdim+j];
	            D[i*Mdim + j] = tmp;
	        }
	        
	    }
	    double cputime = (double)(clock() - timestamp) / CLOCKS_PER_SEC * 1000;
	    printf("cpu串行执行时间：%8.3f ms\n", cputime);
	
	    
	    status = clEnqueueReadBuffer(commandQueue, memObjects[2],CL_TRUE, 0,
	            sizeof(float)* szC, C,0, NULL, NULL);
	    if (status) 
	        perror("读回数据的时候发生错误\n");
	
	    
	    printf("\n输入矩阵  Array A:\n");
	    for (i = 0; i < Ndim; i++) {
	        for (j = 0; j < Pdim; j++)
	             printf("%.3f\t", A[i*Pdim + j]);
	        printf("\n");
	    }
	    printf("\n输入矩阵  Array B:\n");
	    for (i = 0; i < Pdim; i++) {
	        for (j = 0; j < Mdim; j++)
	             printf("%.3f\t", B[i*Mdim + j]);
	        printf("\n");
	    }
	    printf("\nOpenCL的计算结果  Array C:\n");
	    for (i = 0; i < Ndim; i++) {
	        for (j = 0; j < Mdim; j++)
	             printf("%.3f\t", C[i*Mdim + j]);
	        printf("\n");
	    }
	    printf("\n串行的计算结果  Array D:\n");
	    for (i = 0; i < Ndim; i++) {
	        for (j = 0; j < Mdim; j++)
	             printf("%.3f\t", D[i*Mdim + j]);
	        printf("\n");
	    }
	    cout << endl;
	    
	
	    if (A)
	        free(A);
	    if (B)
	        free(B);
	    if (C)
	        free(C);
	
	    
	    clReleaseMemObject(memObjects[2]);
	    clReleaseMemObject(memObjects[1]);
	    clReleaseMemObject(memObjects[0]);
	    clReleaseProgram(program);
	    clReleaseCommandQueue(commandQueue);
	    clReleaseContext(context);
	    
	
	    return 0;
	}

```
