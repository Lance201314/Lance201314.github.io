---
layout: post
title: UIImage Picker Color
category: iOS
keywords: iOS UIImage picker Color
description:
---

Date: 2015-10-15 晚上10:09


	本文主要是讲述，获取图片中对应某个像素点的颜色，类似于photoshop，取色器等。
	
	1.	获取图片的RGBA像素数据
	2.  生成对应的颜色
	
#### 效果图
![image](/images/llimageColor01.png) ![image](/images/llimageColor02.png) ![image](/images/llimageColor03.png)
	
### 获取图片数据

##### 1、获取图片的的RGB Context

	{% highlight objective-c %}
	+ (CGContextRef)getRGBContextByCGImageRef:(CGImageRef)imageRef {
    
    CGContextRef context = NULL;
    // bitmap data address
    void *bitmapData;
    // bytes per row
    size_t bitmapBytesPerRow;
    // bytes of image
    size_t bitmapByteCount;
    
    // image width, height
    size_t pixelsWidth = CGImageGetWidth(imageRef);
    size_t pixelsHeight = CGImageGetHeight(imageRef);
    
    // get the bytes per row
    bitmapBytesPerRow = 4 * pixelsWidth;
    // get all bytes
    bitmapByteCount = bitmapBytesPerRow * pixelsHeight;
    
    // color space  RGB
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    if (!colorSpace) {
        fprintf(stderr, "create color space fail \n");
        
        return NULL;
    }
    // alloc memory for bytesData
    bitmapData = malloc(bitmapByteCount);
    if (!bitmapData) {
        fprintf(stderr, "malloc memory fail \n");
        
        CGColorSpaceRelease(colorSpace);
        
        return NULL;
    }
    
    // For example, premultiplied ARGB
    context = CGBitmapContextCreate(bitmapData, pixelsWidth, pixelsHeight, 8, bitmapBytesPerRow, colorSpace, kCGImageAlphaPremultipliedFirst);
    if (!context) {
        fprintf(stderr, "create bitmapContext fail \n");
        free(bitmapData);
        
        return NULL;
    }
    CGColorSpaceRelease(colorSpace);
    
    return context;
    }
	{% endhighlight %}

##### 2、映射对应像素的数据, 通过对应像素点，获取对应的颜色

	{% highlight objective-c %}
	+ (UIColor *)getImagePixelColorByCGImageRef:(CGImageRef)imageRef withPoint:(CGPoint)point {
    UIColor *color = nil;
    
    CGContextRef context = [LLImageColorByPixel getRGBContextByCGImageRef:imageRef];
    if (!context) {
        NSLog(@"create RGBA context fail \n");
        
        return nil;
    }
    size_t width = CGImageGetWidth(imageRef);
    size_t height = CGImageGetHeight(imageRef);
    CGRect rect = CGRectMake(0,0, width, height);
    
    CGContextDrawImage(context, rect, imageRef);
    // get images data
    unsigned char *dataPoint = CGBitmapContextGetData(context);
    size_t bytesBitmapAll = CGBitmapContextGetBytesPerRow(context) * height;
    if (dataPoint) {
        
        int offset = 4 * (round(point.y) * width + round(point.x));
        
        if (offset >= bytesBitmapAll || offset > bytesBitmapAll - 3) {
            return nil;
        }
        
        //  kCGImageAlphaPremultipliedFirst  For example, premultiplied ARGB
        int alpha = dataPoint[offset];
        int red = dataPoint[offset + 1];
        int green = dataPoint[offset + 2];
        int blue = dataPoint[offset + 3];
        
        NSLog(@"偏移地址: %i colors: RGBA %i %i %i  %i",offset,red,green,blue,alpha);
        
        color = [UIColor colorWithRed:red / 255.0 green:green / 255.0 blue:blue / 255.0 alpha:alpha / 255.0];
    }
    
    /**
     *   release memory
     */
    CGContextRelease(context);
    if (dataPoint) {
        free(dataPoint);
    }
    
    return color;
	}
	{% endhighlight %}
	
#### 注意

&nbsp;&nbsp;&nbsp;&nbsp;在映射UIImageView与对应的UIImage数据的时候，需要做一个处理。

&nbsp;&nbsp;&nbsp;&nbsp;1、如果你的图片，没有进行压缩比例，则，可以直接使用对应的点去获取对应定的颜色
&nbsp;&nbsp;&nbsp;&nbsp;2、如果有压缩比例等contentModel改变，则需要做一个过渡处理....需要自己计算中对应的压缩比例。
&nbsp;&nbsp;&nbsp;&nbsp;例如：压缩比例是rota， touch获取的点是point，则有压缩的最后处理的方式是：point = CGPointMake(point.x / rota * 2, point.y / rota * 2); 没有压缩处理，使用原图的处理模式是, point = CGPointMake(point.x * 2, point.y * 2);
	
	
#### [源代码](https://github.com/Lance201314/LLImageColor)


