# snpe_fascv
This project can be used as a Android template project for using SNPE and FastCV on Snapdragon Platform

A Simple Use Case for Using FastCV and SNPE On Snapdragon Platform
Introduction
Snapdragon platforms are suitable platform for making a low power smart vision system by collaborating FastCV and SNPE. FastCV is a image processing library that can run on Hexagon DSP and take an advantage of HVX module on it and SNPE is DNN library that can also run on Hexagon DSP and can translate few kind of generic network structures (like Tensorflow, Caffe and ONNX).
Overview
Most of Qualcomm Snapdragon series contains multiple Hexagon DSP, as a subsystem that have same level of access to peripherals, in this heterogeneous platform, as ARM cores. Most of Hexagon DSPs are running in a real time operating system, called QuRT. QuRT provides POSIX platform for this multi hardware thread environment and most of the frameworks (like Elite, FastRPC, etc) are running in the user space of this OS. In this article, we will take a quick look on FastCV and SNPE and finish it up with an example on handwritten digit recognition system, using these two tools.
FastCV

The FastCV SDK is a collection of algorithms implemented for ARM and optimized for Qualcomm’s Snapdragon processor. You can find this SDK on QDN.
The libraries currently supported are:
1.	 Android 32 bit and 64 bit library
2.	 IA-32 (x86) Win32 and MS Visual C++ 2010, 2012, and 2013.
3.	 IA-32 (x86) Win64 and MS Visual C++ 2012, and 2013.
To get the advantages of algorithms implemented for Qualcomm’s Snapdragon processor, there are APIs should be called as part of initialization and de-initialization process. As part of initialization process, below API
FASTCV_API int fcvSetOperationMode( fcvOperationMode mode )
should be called. An option should be selected based on the application goal.
Below are the available fcvOperationMode options

Operation mode 
	Description
FASTCV_OP_LOW_POWER	The QDSP implementation will be used unless the QDSP speed is 3 times slower than CPU speed.
FASTCV_OP_PERFORMANCE 	The fastest implementation will be used.
FASTCV_OP_CPU_OFFLOAD	The QDSP implementation will be used when it’s available, otherwise it will find for GPU and CPU implementation.
FASTCV_OP_CPU_PERFORMANCE	The CPU fastest implementation will be used.

Here is the link to complete list of APIs
https://developer.qualcomm.com/docs/fastcv/api/index.html
SNPE
SNPE SDK is provided by Qualcomm and contains tools and example on how to convert and deploy DNNs on SnapDragon platform.
For more information about this SDK, you can check QDN
https://developer.qualcomm.com/docs/snpe/model_conv_tensorflow.html
The snpe-sample can be used for loading DLC ( network model file) and test that. As you know , you can run DNN on ARM, GPU or DSP. Depends on which language your target application is based on, you can refer to related example in SDK.
Example
This example is based on previous article on DNN, which you can find it on Intrinsyc’s website. The goal of this example is to design a handwritten digit recognizer for Android, using SNPE and FastCV.
Here is the block diagram for this example.


 

On the first block (screen capture), we use canvas in Android to create a bitmap screen with painting brush
drawPath = new Path();
drawPaint = new Paint();
drawPaint.setColor(Color.WHITE);
drawPaint.setAntiAlias(true);
drawPaint.setStrokeWidth(20);
drawPaint.setStyle(Paint.Style.STROKE);
drawPaint.setStrokeJoin(Paint.Join.ROUND);
drawPaint.setStrokeCap(Paint.Cap.ROUND);
canvasPaint = new Paint(Paint.DITHER_FLAG);

After capturing the bitmap buffer, we pass it to the resizing with filter function to convert it to a proper size of 28x28 image for our DNN.
short[] tout = resizeImage(pixelsBatched.array(), image.getWidth(), image.getHeight());
The resizeImage is a JNI function that call FastCV functions
fcvScaleDownBLu8(pJimgData, w, h, 0, pJimgDataOut, 28, 28, 0);

And finally pass the result image to our DNN network. So first we need to load the model
File modelFile = new File(networkModel);
builder.setDebugEnabled(false);
builder.setCpuFallbackEnabled(true);
builder.setUseUserSuppliedBuffers(false);
builder.setRuntimeOrder(NeuralNetwork.Runtime.DSP);
And then inject the input to the network
tensor.write(rgbBitmapAsFloat, 0, rgbBitmapAsFloat.length);
inputs.put(mInputLayer, tensor);
final Map<String, FloatTensor> outputs = network.execute(inputs);
The result will  be saved in the Map structure.
To compare the result on different platforms, you can select fcvOperationMode for different mode on FastCV and also different runtime processing for network, by changing NeuralNetwork.Runtime property.
The result for few of the runtime’s options are
1.	SNPE on CPU and FastCV on Performance mode ~ 105 ms
2.	SNPE on DSP and FastCV on LOW_POWER mode ~ 75 ms
3.	SNPE on GPU_FLOAT16 and FastCV on GPU ~ 84 ms
Summary
As it is mentioned, Heterogenous Snapdragon SOC can be a suitable platform for any system that targeted for low-power and high efficiency intelligent system. Using FastCV beside SNPE let users to take advantage of Hexagon DSP subsystems for better power consumptions and in most cases better performance.

