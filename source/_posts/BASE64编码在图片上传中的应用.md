---
title: BASE64编码在图片上传中的应用
date: 2017-08-12 16:07:13
categories: Java学习笔记
tags: BASE64编码
description: 上篇博客介绍了常用的Java加密解密方法，其中有一个就是BASE64编码，严格地说它并不是加密方法，而是最常见的用于传输8Bit字节码的编码方式之一，用于在HTTP环境下传递较长的标识信息。
---
# 概述 #
---
在实际工作中，遇到许多同事，也包括网上的一些例子建议在图片上传过程中将图片转换成base64编码上传，问其原因，竟然没有人知道，只是说看到别人这么做了(⊙o⊙)…

# 正文 #
---
## 关于BASE64的利弊 ##
关于base64编码的原理就不在说了，可自行搜索，接下来说说上传base64编码的利弊。  
好处就是，把图片转换成base64编码后图片转换成了文本，可以嵌入css或者js文件中把图片缓存到本地，而不需要每次载入页面先从服务器获取src地址，然后在下载图片，大大减少http请求次数，减轻服务器压力，加快图片在页面的显示速度，而这在移动端应用中可以提高用户体验。
有利的是可以作为文本，而弊端也是文本。
对于前端，特别是移动端，把图片转换成base64编码会带来CPU的开销，而且编码后文本的大小比图片要增加1/3,也就是说上传一张图片消耗的流量增加1/3,这些对于手机端应用来说是不可容忍的。
对于服务器端，还要把base64编码解析为图片，而且如果上传的base64编码过大，后端会报错，前端传入的参数也不会得到，post提交方式和http协议规范虽然没有对上传数据的大小有限制，但问题取决于服务器处理程序的处理能力，比如Tomcat服务器默认的post大小限制为2M。  

## 图片转base64上传 ##
图片转base64上传的大致思路是前台传以data:image/jpeg;base64,开头的base64编码的String字符串，后台接收字符串以后先进行base64解码 .decodeBuffer()，转换成二进制编码，然后使用字节输出流FileOutputStream（）将文件保存到指定目录下。  

### Java将图片转base64 ###  

	public class ImageUtils {  
	    /** 
	     * 将网络图片进行Base64位编码 
	     * @param imgUrl 图片的url路径
	     * @return 
	     */  
	    public static String encodeImgageToBase64(URL imageUrl) {
			// 将图片文件转化为字节数组字符串，并对其进行Base64编码处理  
	        ByteArrayOutputStream outputStream = null;  
	        try {  
	            BufferedImage bufferedImage = ImageIO.read(imageUrl);  
	            outputStream = new ByteArrayOutputStream();  
	            ImageIO.write(bufferedImage, "jpg", outputStream);  
	        } catch (MalformedURLException e1) {  
	            e1.printStackTrace();  
	        } catch (IOException e) {  
	            e.printStackTrace();  
	        }  
	        // 对字节数组Base64编码  
	        BASE64Encoder encoder = new BASE64Encoder();  
	        return encoder.encode(outputStream.toByteArray());// 返回Base64编码过的字节数组字符串  
	    }
		
		/** 
	     * 将本地图片进行Base64位编码 
	     * @param imgUrl 图片的url路径
	     * @return 
	     */  
	    public static String encodeImgageToBase64(File imageFile) {
			// 将图片文件转化为字节数组字符串，并对其进行Base64编码处理  
	        ByteArrayOutputStream outputStream = null;  
	        try {  
	            BufferedImage bufferedImage = ImageIO.read(imageFile);  
	            outputStream = new ByteArrayOutputStream();  
	            ImageIO.write(bufferedImage, "jpg", outputStream);  
	        } catch (MalformedURLException e1) {  
	            e1.printStackTrace();  
	        } catch (IOException e) {  
	            e.printStackTrace();  
	        }  
	        // 对字节数组Base64编码  
	        BASE64Encoder encoder = new BASE64Encoder();  
	        return encoder.encode(outputStream.toByteArray());// 返回Base64编码过的字节数组字符串  
	    }
	}

### js将图片转base64 ### 

	//将图片转base64
	function convertImgToBase64(url, callback, outputFormat){
	  	var canvas = document.createElement('CANVAS'),
	  	ctx = canvas.getContext('2d'),
	    img = new Image;
	  	img.crossOrigin = 'Anonymous';
	  	img.onload = function(){
		    canvas.height = img.height;
		    canvas.width = img.width;
		    ctx.drawImage(img,0,0);
		    var dataURL = canvas.toDataURL(outputFormat || 'image/png');
		    callback.call(this, dataURL);
		    canvas = null; 
		    console.log(dataURL);
	  };
	  img.src = url;
	}
	 
	//测试
	convertImgToBase64('http://p2.so.qhimgs1.com/sdr/720_1080_/t01d713cf1ad6717772.jpg', function(base64Img){
	  // Base64DataURL
	}); 

convertImgToBase64函数url可以是网络url也可以是本地的。

### bitmap将图片转base64 ###
	
	function GetBase64Code(path){//path绝对路径
		var bitmap = new plus.nativeObj.Bitmap("test"); //test标识谁便取
		// 从本地加载Bitmap图片
		bitmap.load(path,function(){
			var base4=bitmap.toBase64Data();
			var datastr=base4.split(',',3)
			if(datastr.length>1)
			{
			   pics.push(datastr[1]);
			}else
			{
			   pics.push(datastr[0]);
			}
			console.log('加载图片：'+base4);
		},function(e){
			console.log('加载图片失败：'+JSON.stringify(e));
		});
	}

这种方式在移动端应用中使用，兼容android和ios。


### base64编码转图片文件 ###

后台接收到base64编码字符串后要转换成图片文件存放到服务器，这里指展示部分代码：

** 将base64编码转成二进制 **

	imageFile = imageFile.replaceAll("data:image/jpeg;base64,", "");      
	BASE64Decoder decoder = new BASE64Decoder();
	// Base64解码      
	byte[] imageByte = null;
	try {
		imageByte = decoder.decodeBuffer(imageFile);      
		for (int i = 0; i < imageByte.length; ++i) {      
			if (imageByte[i] < 0) {// 调整异常数据      
				imageByte[i] += 256;      
			}      
		}      
	} catch (Exception e) {
		 e.printStackTrace(); 
	} 

** 将二进制转成file文件 **

	// 生成文件名
    String files = new SimpleDateFormat("yyyyMMddHHmmssSSS")
            .format(new Date())
            + (new Random().nextInt(9000) % (9000 - 1000 + 1) + 1000)
            + ".png";
    // 生成文件路径
    String filename = Constant.UPLOAD_PATH + files;     
    try {
        // 生成文件         
        File imageFile = new File(filename);
        imageFile.createNewFile();
           if(!imageFile.exists()){
               imageFile.createNewFile();
            }
            OutputStream imageStream = new FileOutputStream(imageFile);
            imageStream.write(imageByte);
            imageStream.flush();
            imageStream.close();                    
    } catch (Exception e) {         
        e.printStackTrace();
    }

# 总结 #
---
将图片转base64上传虽然能减少http请求次数，但是因为中间需要转码解码所以过程要多一些，我的建议是如果想要把图片缓存到本地，可以转base64后存到本地，上传时直接上传图片文件，现在的框架对文件的上传也有很好的支持。

# 参考 #
---

[网站性能优化:base64:URL传输图片文件](http://blog.csdn.net/xymyeah/article/details/8559136)