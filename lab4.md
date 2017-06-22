# OpenCL实验四 - 直方均衡化

### 1.直方均衡原理

直方图均衡化的作用就是图像增强

如果一副图像的像素占有很多的灰度级而且分布均匀，那么这样的图像往往有高对比度和多变的灰度色调。直方图均衡化就是一种能仅靠输入图像直方图信息自动达到这种效果的变换函数。它的基本思想是对图像中像素个数多的灰度级进行展宽，而对图像中像素个数少的灰度进行压缩，从而扩展像原取值的动态范围，提高了对比度和灰度色调的变化，使图像更加清晰。直方图均衡化的英文名称是Histogram Equalization. 
　　图像对比度增强的方法可以分成两类:一类是直接对比度增强方法;另一类是间接对比度增强方法。直方图拉伸和直方图均衡化是两种最常见的间接对比度增强方法。直方图拉伸是通过对比度拉伸对直方图进行调整，从而“扩大”前景和背景灰度的差别，以达到增强对比度的目的，这种方法可以利用线性或非线性的方法来实现;直方图均衡化则通过使用累积函数对灰度值进行“调整”以实现对比度的增强。
　　直方图均衡化处理的“中心思想”是把原始图像的灰度直方图从比较集中的某个灰度区间变成在全部灰度范围内的均匀分布。直方图均衡化就是对图像进行非线性拉伸，重新分配图像像素值，使一定灰度范围内的像素数量大致相同。直方图均衡化就是把给定图像的直方图分布改变成“均匀”分布直方图分布。
　　缺点：
　　1）变换后图像的灰度级减少，某些细节消失； 
　　2）某些图像，如直方图有高峰，经处理后对比度不自然的过分增强。 
　　直方图均衡化是图像处理领域中利用图像直方图对对比度进行调整的方法。 
　　这种方法通常用来增加许多图像的局部对比度，尤其是当图像的有用数据的对比度相当接近的时候。通过这种方法，亮度可以更好地在直方图上分布。这样就可以用于增强局部的对比度而不影响整体的对比度，直方图均衡化通过有效地扩展常用的亮度来实现这种功能。
　　这种方法对于背景和前景都太亮或者太暗的图像非常有用，这种方法尤其是可以带来X光图像中更好的骨骼结构显示以及曝光过度或者曝光不足照片中更好的细节。这种方法的一个主要优势是它是一个相当直观的技术并且是可逆操作，如果已知均衡化函数，那么就可以恢复原始的直方图，并且计算量也不大。这种方法的一个缺点是它对处理的数据不加选择，它可能会增加背景杂讯的对比度并且降低有用信号的对比度。

### 2.操作步骤

  直方图均衡化的基本思想是把原始图的直方图变换为均匀分布的形式,这样就增加了象素灰度值的动态范围从而可达到增强图像整体对比度的效果.设原始图像在(x,y)处的灰度为f,而改变后的图像为g,则对图像增强的方法可表述为将在(x,y)处的灰度f映射为g.在灰度直方图均衡化处理中对图像的映射函数可定义为:g = EQ (f),这个映射函数EQ(f)必须满足两个条件(其中L为图像的灰度级数):　　(1)EQ(f)在0≤f≤L-1范围内是一个单值单增函数.这是为了保证增强处理没有打乱原始图像的灰度排列次序,原图各灰度级在变换后仍保持从黑到白(或从白到黑)的排列.　　(2)对于0≤f≤L-1有0≤g≤L-1,这个条件保证了变换前后灰度值动态范围的一致性.

### 3.直方图均衡的实现

  直方图均衡的实现如下图，处理前后的对比。

  ![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab4-1.png)

### 4.重要代码部分
	
OpenCV

	#include <CL/cl.h>
	#include <cv.h>
	#include "opencv2/core/utility.hpp"
	#include "opencv2/imgproc.hpp"
	#include "opencv2/imgcodecs.hpp"
	#include "opencv2/highgui.hpp"
	#include <iostream>
	
	using namespace std;
	using namespace cv;
	
	int main()
	{
	    Mat src = imread("img1.jpg",0);
	    imshow("OriginalImage",src); 
	    

	     MatND hist;     
	    int dims = 1;
	    float hranges[] = {0, 255};
	    const float *ranges[] = {hranges};   
	    int size = 256;
	    int channels = 0;
	    calcHist(&src, 1, &channels, Mat(), hist, dims, &size, ranges);  
	    int scale = 1;
	    Mat imageShow(size * scale, size, CV_8U, Scalar(0));
	    double minVal = 0;
	    double maxVal = 0;
	    minMaxLoc(hist,&minVal, &maxVal, 0, 0);
	    int hpt = saturate_cast<int>(0.9 * size);
	    for(int i = 0; i < 256; i++)
	    {
	        float value = hist.at<float>(i);       
	        int realValue = saturate_cast<int>(value * hpt/maxVal);
	        rectangle(imageShow,Point(i*scale, size - 1), Point((i+1)*scale - 1, size - realValue), Scalar(255));
	    }
	    imshow("Hist", imageShow);
	
	
	 
	    Mat result;
	    MatND hist1;
	    Mat imageShow1(size * scale, size, CV_8U, Scalar(0));
	    equalizeHist(src, result);
	    imshow("EqualizedImage_by_opencv", result);           
	    calcHist(&result, 1, &channels, Mat(), hist1, dims, &size, ranges);    
	    minMaxLoc(hist1,&minVal, &maxVal, 0, 0); cvGetMinMaxHistValue
	    for(int i = 0; i < 256; i++)
	    {
	        float value = hist1.at<float>(i);  
	        int realValue = saturate_cast<int>(value * hpt/maxVal);
	        rectangle(imageShow1,Point(i*scale, size - 1), Point((i+1)*scale - 1, size - realValue), Scalar(255));
	    }
	    imshow("EqualizedHist_by_opencv", imageShow1);
	
	
	   
	    double grayFre[256] = {0};
	    double grayAcc[256] = {0};
	
	    for(int i=0;i<src.rows;i++)   
	    for(int j=0;j<src.cols;j++)
	    {
	        grayFre[src.at<uchar>(i,j)]++;
	    }
	   
	
	    for(int i=0;i<256;i++) grayFre[i] = grayFre[i]/(src.rows * src.cols); 
	
	    for(int i=0;i<256;i++)
	    for(int j=0;j<=i;j++)
	    {
	        grayAcc[i] += grayFre[j]; 
	    }
	    for(int i=0;i<256;i++)
	    cout<<grayAcc[i]<<endl;
	    
	
	    for(int i=0;i<src.rows;i++)
	    for(int j=0;j<src.cols;j++)
	    {
	        int grayValue = grayAcc[src.at<uchar>(i,j)] * 255;   
	        if(grayValue > 255) grayValue = 255;
	        result.at<uchar>(i,j) = grayValue;
	    }
	    imshow("EqualizedImage_by_myself", result);  
	
	    calcHist(&result, 1, &channels, Mat(), hist, dims, &size, ranges);   
	    minMaxLoc(hist,&minVal, &maxVal, 0, 0);  cvGetMinMaxHistValue
	    for(int i = 0; i < 256; i++)
	    {
	        float value = hist.at<float>(i);        
	        int realValue = saturate_cast<int>(value * hpt/maxVal);
	        rectangle(imageShow,Point(i*scale, size - 1), Point((i+1)*scale - 1, size - realValue), Scalar(255));
	    }
	    imshow("EqualizedHist_by_myself", imageShow);
	
	    waitKey(0);
	    return 0;
	}
	```

	
	OpenCL
	
	```
	__kernel  void hist_cal( __global uchar * src_data, __global float * grayFree)					
	{    
		const int ix = get_global_id(0); 
	    grayFree[src_data[ix]]++;
	}
	
	
	
	__kernel  void hist_enq( __global uchar * src_data, __global uchar * dst_data, 
	__global const float * grayAcc )					
	{    
	    const int ix = get_global_id(0);     
	
	    int grayValue = grayAcc[src_data[ix]] * 255;
	    dst_data[ix] = grayValue;
	}
	

	#include <CL/cl.h>
	#include <cv.h>
	#include "opencv2/core/utility.hpp"
	#include "opencv2/imgproc.hpp"
	#include "opencv2/imgcodecs.hpp"
	#include "opencv2/highgui.hpp"
	#include <iostream>
	#include <fstream>
	
	using namespace std;
	using namespace cv;
	#pragma comment (lib. "OpenCL.lib")
	
	#define graySize 256
	
	int main()
	{
	
	    cl_int ret = 0;
	    cl_uint num_platform = 0;
	    cl_platform_id* platforms = NULL;
	    cl_platform_id platform = NULL;
	    cl_device_id device = NULL;
	    cl_context context = NULL;
	    cl_command_queue queue  = NULL;
	    cl_program program = NULL;
	    cl_kernel kernel = NULL;
	
	
	    ret = clGetPlatformIDs(0, NULL, &num_platform);
	    if(num_platform < 1)
	    cout<<"没有检测到OpenCL平台"<<endl;
	
	    ret = clGetDeviceIDs(platform, CL_DEVICE_TYPE_ALL, 1, &device, NULL); 
	    cl_bool imageSupport = CL_FALSE;
		clGetDeviceInfo(device, CL_DEVICE_IMAGE_SUPPORT, sizeof(cl_bool), &imageSupport, NULL);
		if (imageSupport != CL_TRUE) cout << "Such device does not support images!!!" <<endl;
		
	    context = clCreateContext(NULL, 1, &device, NULL, NULL, NULL); 
	
	    queue = clCreateCommandQueue(context, device, CL_QUEUE_PROFILING_ENABLE, &ret);
	
	
	    string photoName = "img1.jpg"; 
		Mat src = imread(photoName, 0);
		imshow("Original Image", src);
	
	
		int width, height;   
		width = src.cols;
		height = src.rows;
	    size_t mem_size = width*height;
	    cl_mem d_ip = clCreateBuffer(context, CL_MEM_READ_ONLY,mem_size*sizeof(unsigned char), NULL, NULL);
		cl_mem gray = clCreateBuffer(context, CL_MEM_WRITE_ONLY, graySize*sizeof(cl_float), NULL, NULL);
	    
	    
	
	    cl_event writeEvt;
	    ret = clEnqueueWriteBuffer( 
	        queue, d_ip, CL_TRUE, 
	        0, mem_size*sizeof(unsigned char), (void*)(src.ptr<uchar>(0)),
	        0, NULL, &writeEvt);
	    clWaitForEvents(1, &writeEvt);
	
	
	    FILE* program_handle = fopen("kernel.cl", "r"); 
	    int flag = fseek(program_handle, 0, SEEK_END);
	    size_t program_size = ftell(program_handle);
	    rewind(program_handle);
	    char* program_buffer = (char*)malloc(program_size + 1);
	    fread(program_buffer, sizeof(char), program_size, program_handle);
	    program_buffer[program_size] = '\0';
	    fclose(program_handle);
	
	    program = clCreateProgramWithSource(context, 1, (const char **)&program_buffer,
	     &program_size, &ret); 
	
	    ret = clBuildProgram(program, 1, &device, NULL, NULL, NULL);
	    if (ret != 0) {
	        char tbuf[0x10000];
	        clGetProgramBuildInfo(program, device, CL_PROGRAM_BUILD_LOG, 0x10000, tbuf, NULL);
	        printf("\n%s\n", tbuf);
	        return -1;
	    }
	
	    kernel = clCreateKernel(program, "hist_cal", &ret);
	    
		ret = clSetKernelArg(kernel, 0, sizeof(cl_mem), (void*)&d_ip);
		ret = clSetKernelArg(kernel, 1, sizeof(cl_mem), (void*)&gray);
	
	    cl_event ev;
	    size_t globalWorkSize = mem_size;
		ret = clEnqueueNDRangeKernel(queue, kernel, 1, NULL, &globalWorkSize, NULL, 0, NULL, &ev);
	    clWaitForEvents(1, &ev);
	
	
	    float * grayFre = new float[graySize];
	    float * grayAcc = new float[graySize];
	    for(int i=0;i<graySize;i++) grayAcc[i] = 0;
	    cl_event readEvt;
	     
	    grayFre = (cl_float *) clEnqueueMapBuffer( 
	        queue, gray, CL_TRUE,
			CL_MAP_READ,0, graySize*sizeof(cl_float),
			0, NULL, &readEvt, NULL);
	    clWaitForEvents(1, &readEvt);
	    
	
	    for(int i=0;i<graySize;i++)
	    if(grayFre[i] < 1) grayFre[i] = 0;
	
	    for(int i=0;i<graySize;i++) grayFre[i] = grayFre[i]/(src.cols * src.rows); 
	
	    for(int i=0;i<graySize;i++)
	    for(int j=0;j<=i;j++)
	    {
	        grayAcc[i] += grayFre[j]; 
	    }
	
	    for(int i=0;i<graySize;i++) cout<<grayAcc[i]<<endl;
	
	    
	    
	
	    cl_mem d_op = clCreateBuffer(context, CL_MEM_WRITE_ONLY,mem_size*sizeof(unsigned char), NULL, NULL);
	    cl_mem grayInput = clCreateBuffer(context, 
	            CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
	            graySize*sizeof(cl_float), grayAcc, NULL);
	    cl_kernel kernel1 = clCreateKernel(program, "hist_enq", &ret);
	    ret = clSetKernelArg(kernel1, 0, sizeof(cl_mem), (void*)&d_ip);
	    ret = clSetKernelArg(kernel1, 1, sizeof(cl_mem), (void*)&d_op);
		ret = clSetKernelArg(kernel1, 2, sizeof(cl_mem), (void*)&grayInput);
	
		ret = clEnqueueNDRangeKernel(queue, kernel1, 1, NULL, &globalWorkSize, NULL, 0, NULL, &ev);
	    clWaitForEvents(1, &ev);
	
	
	    unsigned char *op_data = new unsigned char [mem_size];
		op_data = (cl_uchar *) clEnqueueMapBuffer( 
	        queue, d_op, CL_TRUE,
			CL_MAP_READ,0, mem_size*sizeof(unsigned char),
			0, NULL, &readEvt, NULL);
		clWaitForEvents(1, &readEvt);
	    Mat result = Mat::zeros(src.rows, src.cols, src.type());
	    uchar *p = result.ptr<uchar>(0);
	    for(int i=0;i<mem_size;i++)
	    {
	        *(p++) = op_data[i];
	    }
	    imshow("EqualizedImage_by_opencl", result);
	
	

	     MatND hist;     
	    int dims = 1;
	    float hranges[] = {0, 255};
	    const float *ranges[] = {hranges}; 
	    int size = 256;
	    int channels = 0;
	    calcHist(&src, 1, &channels, Mat(), hist, dims, &size, ranges); 
	    int scale = 1;
	    Mat imageShow(size * scale, size, CV_8U, Scalar(0));
	    double minVal = 0;
	    double maxVal = 0;
	    minMaxLoc(hist,&minVal, &maxVal, 0, 0); 
	    int hpt = saturate_cast<int>(0.9 * size);
	    for(int i = 0; i < 256; i++)
	    {
	        float value = hist.at<float>(i);    
	        int realValue = saturate_cast<int>(value * hpt/maxVal);
	        rectangle(imageShow,Point(i*scale, size - 1), Point((i+1)*scale - 1, size - realValue), Scalar(255));
	    }
	    imshow("Hist", imageShow);
	
	
	    MatND hist1;
	    Mat imageShow1(size * scale, size, CV_8U, Scalar(0));
	    calcHist(&result, 1, &channels, Mat(), hist1, dims, &size, ranges);  
	    minMaxLoc(hist1,&minVal, &maxVal, 0, 0);  cvGetMinMaxHistValue
	    for(int i = 0; i < 256; i++)
	    {
	        float value = hist1.at<float>(i);       
	        int realValue = saturate_cast<int>(value * hpt/maxVal);
	        rectangle(imageShow1,Point(i*scale, size - 1), Point((i+1)*scale - 1, size - realValue), Scalar(255));
	    }
	    imshow("EqualizedHist_by_opencl", imageShow1);
	
	
	    cout<<"ret = "<<ret<<endl;
	    waitKey(0);
	    clReleaseProgram(program);
	    clReleaseCommandQueue(queue);
	    clReleaseContext(context);
	
	
	
	    return 0;
	}