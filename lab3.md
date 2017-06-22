# OpenCL实验三 - Sobel边缘检测

### 1.运行image rotate程序

编译命令：

    cmake CMakeList.txt
	make

程序直接执行即可，运行之后发现CMakeCache.txt路径有错，原因是因为原版的文件里面所给的路径是上个修改者根据自己的电脑自己定义的路径，并不是一个通用的路径。因此我们需要修改成我们需要的路径。里面有很多处都需要修改。

修改之后，我们再运行，得到结果：

![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab3-1.png)

运行之后会提示错误，是因为我们使用的opencl中内置的函数clWaitForEvents或者clFlush和用户自己定义的WaitForEventAndRelease这两种方法使用其中一种即可，两种方式都使用的话就重复了，因此我们需要删除其中一个。
要么注释掉clWaitForEvents
要么注释掉clFlush，waitForEventAndRelease

GPU设备运行结果如下：

![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab3-2.png)

图片结果如下：

GPU的旋转结果如下：

![](https://github.com/Valesail1/ES2016_14353414/blob/master/rotate.jpg)

CPU的旋转结果如下：

![](https://github.com/Valesail1/ES2016_14353414/blob/master/cpu_lenna_rotate.jpg)

### 2.编写Sobel程序，并设定sobel算子大小为3x3

原图如下：

![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab3-3.png)

彩色边缘图的边缘检测效果图：

![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab3-4.png)

灰度边缘检测效果图：

![](https://github.com/Valesail1/ES2016_14353414/blob/master/lab3-5.png)

### 相关的代码

灰度图边缘检测
kernel.cl
    
    const sampler_t sampler = CLK_ADDRESS_CLAMP_TO_EDGE | CLK_FILTER_NEAREST;
    kernel void sobel_gray(read_only image2d_t src,write_only image2d_t dst)
    {
        int x = (int)get_global_id(0);
        int y = (int)get_global_id(1);
        if (x >= get_image_width(src) || y >= get_image_height(src))
            return;
        float4 p00 = read_imagef(src, sampler, (int2)(x - 1, y - 1));
        float4 p10 = read_imagef(src, sampler, (int2)(x, y - 1));
        float4 p20 = read_imagef(src, sampler, (int2)(x + 1, y - 1));
        float4 p01 = read_imagef(src, sampler, (int2)(x - 1, y));
        float4 p21 = read_imagef(src, sampler, (int2)(x + 1, y));
        float4 p02 = read_imagef(src, sampler, (int2)(x - 1, y + 1));
        float4 p12 = read_imagef(src, sampler, (int2)(x, y + 1));
        float4 p22 = read_imagef(src, sampler, (int2)(x + 1, y + 1));
        float3 gx = -p00.xyz + p20.xyz + 2.0*(p21.xyz - p01.xyz) - p02.xyz + p22.xyz;
        float3 gy = -p00.xyz + p02.xyz + 2.0*(p21.xyz - p10.xyz) - p20.xyz + p22.xyz;

        float gs_x = 0.3333f * (gx.x + gx.y + gx.z);
        float gs_y = 0.3333f * (gy.x + gy.y + gy.z);

        float g = native_sqrt(gs_x* gs_x + gs_y * gs_y);       
        write_imagef(dst,(int2)(x,y),(float4)(g,g,g,1.0f));
    }


主代码

    #include<cv.h>
    #include "opencv2/core/utility.hpp"
    #include "opencv2/imgproc.hpp"
    #include "opencv2/imgcodecs.hpp"
    #include "opencv2/highgui.hpp"
    #include <iostream>
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
    #include <string>

    using namespace std;
    using namespace cv;
    #pragma comment (lib. "OpenCL.lib")

	size_t RoundUp(int groupSize, int globalSize)
	    {
	        int r = globalSize % groupSize;
	        if (r == 0)
	        {
	            return globalSize;
	        }
	        else
	        {
	            return globalSize + groupSize - r;
	        }
	    }


    cl_mem LoadImage(cl_context context, Mat &image1, int &width, int &height)
    {
        char *buffer = new char[width * height * 4];
        int w = 0;
        for (int v = height - 1; v >= 0; v--)
        {
            for (int u = 0; u <width; u++)
            {
                buffer[w++] = image1.at<cv::Vec3b>(v, u)[0];
                buffer[w++] = image1.at<cv::Vec3b>(v, u)[1];
                buffer[w++] = image1.at<cv::Vec3b>(v, u)[2];
                w++;
            }
        }

        // Create OpenCL image
        cl_image_format clImageFormat;
        clImageFormat.image_channel_order = CL_RGBA;
        clImageFormat.image_channel_data_type = CL_UNORM_INT8;

        cl_int errNum;
        cl_mem clImage;
        clImage = clCreateImage2D(context,
            CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
            &clImageFormat,
            width,
            height,
            0,
            buffer,
            &errNum);

        if (errNum != CL_SUCCESS)
        {
            std::cerr << "Error creating CL image object" << std::endl;
            return 0;
        }

        return clImage;
    }

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
        cl_sampler sampler = 0;

        ret = clGetPlatformIDs(0, NULL, &num_platform);
        if(num_platform > 0)
        {
            cout<<"该电脑共有"<<num_platform<<"个OpenCL平台"<<endl<<endl;
            platforms = new cl_platform_id[num_platform];  
            ret = clGetPlatformIDs(num_platform, platforms, NULL);
            cout<<"请输入0或1,  0是选择intel， 1是选择nvidia"<<endl;
            int index;
            cin>>index;
            platform = platforms[index];
            delete[] platforms;
        }
        else cout<<"没有检测到OpenCL平台"<<endl;

        ret = clGetDeviceIDs(platform, CL_DEVICE_TYPE_ALL, 1, &device, NULL);
        cl_bool imageSupport = CL_FALSE;
        clGetDeviceInfo(device, CL_DEVICE_IMAGE_SUPPORT, sizeof(cl_bool), &imageSupport, NULL);
        if (imageSupport != CL_TRUE) cout << "Such device does not support images!!!" <<endl;

        context = clCreateContext(NULL, 1, &device, NULL, NULL, NULL);  //创建上下文

        queue = clCreateCommandQueue(context, device, CL_QUEUE_PROFILING_ENABLE, &ret); 

        string photoName = "img.jpg"; 
        Mat src = imread("img.jpg");
        imshow("Original Image", src);
        int width, height;
        width = src.cols;
        height = src.rows;


        cl_mem imageObjects[2] = {0, 0};  
        imageObjects[0] = LoadImage(context, src, width, height);
        if(imageObjects[0] == 0) cout<<"Loading Image Failed !!"<<endl;
        cl_image_format clImageFormat;
        clImageFormat.image_channel_order = CL_RGBA;
        clImageFormat.image_channel_data_type = CL_UNORM_INT8;
        imageObjects[1] = clCreateImage2D(context, CL_MEM_WRITE_ONLY, &clImageFormat,
        width, height, 0, NULL, &ret);

        sampler = clCreateSampler(context, CL_FALSE, CL_ADDRESS_CLAMP_TO_EDGE, 
        CL_FILTER_NEAREST, &ret);

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

        kernel = clCreateKernel(program, "sobel_rgb", &ret);

        ret = clSetKernelArg(kernel, 0, sizeof(cl_mem), &imageObjects[0]);
        ret = clSetKernelArg(kernel, 1, sizeof(cl_mem), &imageObjects[1]);

        size_t localWorkSize[2] = { 16, 16 };
        size_t globalWorkSize[2] = { RoundUp(localWorkSize[0], width), RoundUp(localWorkSize[1], height) };
        ret = clEnqueueNDRangeKernel(queue, kernel, 2, NULL, globalWorkSize, localWorkSize, 0, NULL, NULL);

        char *buffer = new char[width * height * 4];
        size_t origin[3] = { 0, 0, 0 };
        size_t region[3] = { width, height, 1 };
        ret = clEnqueueReadImage(queue, imageObjects[1], CL_TRUE,
            origin, region, 0, 0, buffer, 0, NULL, NULL);


        Mat dst;
        dst.create(src.rows, src.cols, src.type());
        int w = 0;
        for (int v = dst.rows-1; v >=0; v--)
        {
            for (int u =0 ; u <dst.cols; u++)
            {
                dst.at<cv::Vec3b>(v, u)[0] = buffer[w++];
                dst.at<cv::Vec3b>(v, u)[1] = buffer[w++];
                dst.at<cv::Vec3b>(v, u)[2] = buffer[w++];
                w++;
            }
        }
        imshow("Sobel Result", dst);

        waitKey(0);
        delete[] buffer;
        clReleaseMemObject(imageObjects[1]);
        clReleaseMemObject(imageObjects[0]);
        clReleaseProgram(program);
        clReleaseCommandQueue(queue);
        clReleaseContext(context);
        return 0;
    }
