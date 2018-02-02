---
title: java数据传输中的加密与解密
date: 2017-07-30 13:14:57
categories: Java学习笔记
tags: 加密解密
description: 作为开发者，编写安全的代码比编写优雅的代码更重要，因为安全是一切应用的根本，所以为了确保数据传输和数据存储的安全，我们可以通过特定的算法，将数据明文加密成复杂的密文。
---
# 前言 #
---
在最近的工作中，用户需求要求在两个独立的系统通信中，要确保对外的接口不能被其他系统非法访问，这就涉及到了数据通信中对信息的加密和Token的验证，所以对这方面的知识做了一些了解。

# 正文 #
---
## 浅谈加密 ##

### 相关术语 ###

加密是以某种特殊的算法改变原有的信息数据，使得未授权的用户即使获得了已加密的信息，但不知道解密的方法，仍无法了解信息的内容。  
在加密系统中出现了一些重要的概念术语：
1. 明文：未加密的数据（plaintext ）。
2. 密文：明文经过加密后的数据（ciphertext ）。
3. 加密：将明文转换为密文的过程(encryption) 。 
4. 解密：将密文转换为明文的过程(decryption) 。
5. 加密算法：将明文转换为密文的转换算法。
6. 解密算法：将密文转换为明文的转换算法
7. 加密密钥：用于加密算法进行加密操作的密钥(encryption key)。
8. 解密密钥：用于解密算法进行解密操作的密钥(decryption key)。

>tip：加密算法和解密算法的操作通常是在一组密钥（ key ）的控制下进行的，分别称为加密密钥(encryption  key) 和解密密钥(decryption  key) 。  在加密系统中，加密算法和密钥是最重要的两个概念。在这里需要对加密算法和密钥进行一个解释。以最简单的“恺撒加密法”为例。 《高卢战记》有描述恺撒曾经使用密码来传递信息，即所谓的“ 恺撒密码” 。它是一种替代密码，通过将字母按顺序推后 3 位起到加密作用，如将字母 A 换作字母D，将字母 B换作字母E。如“China”可以变为“Fklqd”；解密过程相反。 在这个简单的加密方法中，“向右移位”，可以理解为加密算法；“3 ”可以理解为加密密钥。对于解密过程，“向左移位”，可以理解为解密算法；“3 ”可以理解为解密密钥。显然，密钥是一种参数，它是在明文转换为密文或将密文转换为明文的算法中输入的数据。 恺撒加密法的安全性来源于两个方面：第一，对加密算法的隐藏；第二，对密钥的隐蔽。单单隐蔽加密算法以保护信息，在学界和业界已有相当讨论，一般认为是不够安全的。公开的加密算法是给黑客长年累月攻击测试，对比隐蔽的加密算法要安全多。一般说来，加密之所以安全，是因为其加密的密钥的隐藏，并非加密解密算法的保密。而流行的一些加密解密算法一般是完全公开的。敌方如果取得已加密的数据，即使得知加密算法，若没有密钥，也不能进行解密。 

### 常见的加密算法 ###

加密技术从本质上说是对信息进行编码和解码的技术。加密是将可读信息（明文）变为代码形式（密文），解密是加密的逆过程，相当于将密文变为明文。
众多的加密手段大致可以分为**单向加密**和**双向加密**。  
单向加密指通过对数据进行摘要计算生成密文，密文不可逆推还原，比如有Base64、MD5、SHA等。双向加密则相反，指可以把密文逆推还原成明文。  
双向加密又分为**对称加密**和**非对称加密**。  
对称加密是指数据使用者必须拥有同样的密钥才可以进行加密解密，就像大家共同约定了以组暗号一样，对称加密的手段有DES、3DES、AES、IDEA、RC4、RC5等。而非对称加密相对于对称加密而言，无需拥有同一组密钥，它是一种信息公开的密钥交换协议。非对称加密需要公开密钥和私有密钥两组密钥，公开密钥和私有密钥是配对起来的，也就是说使用公开密钥进行数据加密，只有对应的私有密钥才能进行解密。此类加密手段有RSA、DSA等。

{% asset_img pic_01.png 常见的加密算法%}

## 单向加密 ##

单向加密在加密过程中不需要使用密钥，输入明文后由系统直接经过加密算法处理成密文，密文无法解密。只有重新输入明文，并经过同样的加密算法处理，得到相同的密文并被系统重新识别后，才能真正解密。该方法计算复杂，通常只在数据量有限的情形下使用。
常见的单向加密方式有：

- BASE64 （严格地说，属于编码格式，而非加密算法）
- MD5（Message Digest Algorithm，信息摘要算法第五版）
- SHA（Secure Hash Algorithm，安全散列算法）
- HMAC(Hash Message Authentication Code，散列消息鉴别码)

### BASE64 ###

BASE64编码算法不算是真正的加密算法。  
Base64被定义为：Base64内容传送编码被设计用来把任意序列的8位字节描述为一种不易被人直接识别的形式。常见于邮件、http加密，截取http信息，你就会发现登录操作的用户名、密码字段通过BASE64加密的。主要就是BASE64Encoder、BASE64Decoder两个类，我们只需要知道使用对应的方法即可。另，BASE加密后产生的字节位数是8的倍数，如果不够位数以=符号填充。

	/**
	 * base64解密
	 * @param key
	 * @return
	 * @throws Exception
	 */
	public static byte[] decryptBASE64(String key) throws Exception {  
	    return (new BASE64Decoder()).decodeBuffer(key);  
	}   

	/**
	 * base64加密
	 * @param key
	 * @return
	 * @throws Exception
	 */
	public static String encryptBASE64(byte[] key) throws Exception {  
	    return (new BASE64Encoder()).encodeBuffer(key);  
	}

>tip：BASEEncoder和BASEDecoder是非官方JDK实现类。虽然可以在JDK里能找到并使用，但是在API里查不到。JRE 中 sun 和 com.sun 开头包的类都是未被文档化的，如果要使用需要下载sun.misc.BASE64Decoder.jar。

### MD5 ###

Java一般需要获取对象MessageDigest来实现单项加密(信息摘要)，常用于文件校验。不管文件多大，经过MD5后都能生成唯一的MD5值。

	public byte[] eccrypt(String info) throws NoSuchAlgorithmException{  
        //根据MD5算法生成MessageDigest对象  
        MessageDigest md5 = MessageDigest.getInstance("MD5");  
        byte[] srcBytes = info.getBytes();  
        //使用srcBytes更新摘要  
        md5.update(srcBytes);  
        //完成哈希计算，得到result  
        byte[] resultBytes = md5.digest();  
        return resultBytes;  
    }

MD5算法具有一下特点：

- 压缩性：任意长度的数据，算出的MD值长度都是固定的。
- 容易计算：从原数据计算出MD值很容易。
- 抗修改性：对原数据进行任何改动，哪怕只修改个字节，所得到的MD值都有很大区别。
- 弱抗碰撞：已知原数据和其MD值，想找到一个具有相同MD值的数据（即伪造数据）是非常困难的。
- 强抗碰撞：想找到两个不同的数据，使它们具有相同的MD值，是非常困难的。

### SHA ###
SHA称为安全哈希算法。该算法的思想是接收一段明文，然后以一种不可逆的方式将它转换成一段（通常更小）密文，也可以简单的理解为取一串输入码（称为预映射或信息），并把它们转化为长度较短、位数固定的输出序列即散列值（也称为信息摘要或信息认证代码）的过程。

	public byte[] eccrypt(String info) throws NoSuchAlgorithmException{  
	    MessageDigest sha = MessageDigest.getInstance("SHA");  
	    byte[] srcBytes = info.getBytes();  
	    //使用srcBytes更新摘要  
	    sha.update(srcBytes);  
	    //完成哈希计算，得到result  
	    byte[] resultBytes = sha.digest();  
	    return resultBytes;  
	} 

### HMAC ###
HMAC(Hash Message Authentication Code)，散列消息鉴别码，基于密钥的Hash算法的认证协议。消息鉴别码实现鉴别的原理是，用公开函数和密钥产生一个固定长度的值作为认证标识，用这个标识鉴别消息的完整性。使用一个密钥生成一个固定大小的小数据块，即MAC，并将其加入到消息中，然后传输。接收方利用与发送方共享的密钥进行鉴别认证等。

	/**
	 * 初始化HMAC密钥
	 * @return
	 * @throws Exception
	 */
	public static String initMacKey() throws Exception {  
    	KeyGenerator keyGenerator = KeyGenerator.getInstance(KEY_MAC);  
    	SecretKey secretKey = keyGenerator.generateKey();  
    	return encryptBASE64(secretKey.getEncoded());  
	}

	/**
	 * HMAC加密
	 * @param data
	 * @param key
	 * @return
	 * @throws Exception
	 */
	public static byte[] encryptHMAC(byte[] data, String key) throws Exception {  
    	SecretKey secretKey = new SecretKeySpec(decryptBASE64(key), KEY_MAC);  
    	Mac mac = Mac.getInstance(secretKey.getAlgorithm());  
    	mac.init(secretKey);  
    	return mac.doFinal(data);   
	}  

## 双向加密 ##
### 对称加密 ###

在对称加密算法中，双方使用的密钥相同，要求解密方事先必须知道加密密钥。其特点是算法公开、计算量小、加密速度快、加密效率高。不足之处是，通信双方都使用同样的密钥，安全性得不到保证。此外，用户每次使用该算法，需要保证密钥的唯一性，使得双方所拥有的密钥数量很大，密钥管理管理较为困难。  

如上所述，对称加密算法过程中，发送方将明文和加密密钥一起经过加密算法处理，变成密文，发送出去；接收方收到密文后，使用加密密钥以及相同的算法的逆算法对密文解密，恢复为明文。双方使用的密钥相同，要求解密方事先必须知道加密密钥，从这里可以得出：

1. 加密时使用什么密钥，解密时必须使用相同的密钥，否则无法解密。
2. 对相同的信息，不同的密钥，加密结果和解密结果理论上不相同。

现在介绍三种常用的对称加密算法：DES、3DES和AES。  

#### DES ####

DES 是数据加密标准（Data Encryption Standard）的简称，出自 IBM  的研究工作。

	import java.io.UnsupportedEncodingException;
	import java.security.InvalidKeyException;
	import java.security.NoSuchAlgorithmException;
	import java.security.Security;
	import javax.crypto.BadPaddingException;
	import javax.crypto.Cipher;
	import javax.crypto.IllegalBlockSizeException;
	import javax.crypto.KeyGenerator;
	import javax.crypto.NoSuchPaddingException;
	import javax.crypto.SecretKey;	
	import org.junit.Test;	
	import com.thinkgem.jeesite.common.utils.Base64;
	
	public class DesTest {

		//KeyGennrator提供对称密钥生成器的功能，支持各种算法
		private KeyGenerator keygen;
		//SecretKey负责保存对称密钥
		private SecretKey deskey;
		//Cipher负责完成加密或解密工作
		private Cipher c;
		//保存加密的结果
		private byte[] cipherByte;
		
		public DesTest(){
			Security.addProvider(new com.sun.crypto.provider.SunJCE());
			try{
				//实例化支持DES 算法的密钥生成器(算法名称命名需按规定，否则抛出异常) 
	            keygen = KeyGenerator.getInstance("DES"); 
	            // 生成密钥 
	            deskey = keygen.generateKey(); 
	            // 生成Cipher 对象，指定其支持DES 算法 
	            c = Cipher.getInstance("DES"); 
			}catch(NoSuchAlgorithmException ex){
				ex.printStackTrace(); 
			} catch (NoSuchPaddingException e) {
				e.printStackTrace();
			}
		}
		
		/**
		 * @desc 对字符串str 加密
		 * @author yaotengfei
		 * @date 2017年8月2日下午3:17:19
		 * @param str
		 * @return
		 */
		public byte[] createEncryptor(String str){
			try{
				// 根据密钥，对Cipher 对象进行初始化,ENCRYPT_MODE 表示加密模式 
	            c.init(Cipher.ENCRYPT_MODE, deskey); 
	            byte[] src = str.getBytes("UTF-8"); 
	            // 加密，结果保存进cipherByte 
	            cipherByte = c.doFinal(src); 
			} catch (IllegalBlockSizeException e) {
				e.printStackTrace();
			} catch (BadPaddingException e) {
				e.printStackTrace();
			} catch (InvalidKeyException e) {
				e.printStackTrace();
			} catch (UnsupportedEncodingException e) {
				e.printStackTrace();
			}
			 return cipherByte; 
		}
		
		/**
		 * @desc 对字节数组buff 解密
		 * @param buff
		 * @return
		 */
		public byte[] createDecryptor(byte[] buff){
			try{
				// 根据密钥，对Cipher 对象进行初始化,ENCRYPT_MODE 表示解密模式 
	            c.init(Cipher.DECRYPT_MODE, deskey); 
	            // 得到明文，存入cipherByte字符数组 
	            cipherByte = c.doFinal(buff); 
			}catch(java.security.InvalidKeyException ex){
				ex.printStackTrace();
			} catch (IllegalBlockSizeException e) {
				e.printStackTrace();
			} catch (BadPaddingException e) {
				e.printStackTrace();
			}
			return cipherByte; 
		}
		
		/**
		 * @desc 测试
		 */
		@Test
		public void test(){
			DesTest desTest = new DesTest();
			String msg = "rickZone";
			System.out.println("明文："+msg);
			//加密
			byte[] enc = createEncryptor(msg);
			System.out.println("密文："+new String(Base64.encode(enc)));
			//解密
			byte[] dec = desTest.createDecryptor(enc);
			System.out.println("解密后："+new String(dec));
		}
	}

运行结果：
	
	明文：rick'Zone
	密文：PhbGVjfHArXWgF/GflRfxA==
	解密后：rick'Zone

在上面的代码中，不同情况下，密文的内容会不一样，因为KeyGenerator每次生成的密钥是随机的。  
在输出密文的时候为了可读性，可以把密文转换成Base64编码，否则显示的是乱码。

#### 3DES ####

3DES，也称为3DESede或TripleDES，是三重数据加密，并且可以逆推的一种算法方案，通过对DES算法进行改进，针对每个数据块进行三次DES加密，也就是3DES加密算法。  
java使用3DES加密解密的流程为：

1. 传入共同的密钥（keyBytes）以及算法（Algorithm），来构建SecretKey密钥对象。
 SecretKey deskey = new SecretKeySpec(keyBytes, Algorithm);
2. 根据算法实例化Cipher对象。它负责加密/解密。
 Cipher c1 = Cipher.getInstance(Algorithm);
3. 传入加密/解密模式以及SecretKey密钥对象，实例化Cipher对象。
 c1.init(Cipher.ENCRYPT_MODE, deskey);
4. 传入字节数组，调用Cipher.doFinal()方法，实现加密/解密，并返回一个byte字节数组。
 c1.doFinal(src);  

3DES加密代码：

	import java.io.UnsupportedEncodingException;
	import java.security.InvalidKeyException;
	import javax.crypto.BadPaddingException;
	import javax.crypto.Cipher;
	import javax.crypto.IllegalBlockSizeException;
	import javax.crypto.NoSuchPaddingException;
	import javax.crypto.SecretKey;
	import javax.crypto.spec.SecretKeySpec;
	import org.junit.Test;
	import com.thinkgem.jeesite.common.utils.Base64;

	public class Test_3DES {

		//定义加密算法，有DES、DESede(即3DES)、Blowfish
		private static final String Algorithm = "DESede";    
		private static final String PASSWORD_CRYPT_KEY = "201231332322";
		
		/**
		 * @desc 加密方法
		 * @param src 源数据的字节数组
		 * @return
		 */
		public static byte[] encryptMode(byte[] src){
			try{
				SecretKey deskey = new SecretKeySpec(build3DesKey(PASSWORD_CRYPT_KEY), Algorithm);
				//生成密钥
				Cipher c1 = Cipher.getInstance(Algorithm);    //实例化负责加密/解密的Cipher工具类
				c1.init(Cipher.ENCRYPT_MODE, deskey);    //初始化为加密模式
				return c1.doFinal(src);
			}catch(java.security.NoSuchAlgorithmException e1){
				e1.printStackTrace();
			} catch (NoSuchPaddingException e) {
				e.printStackTrace();
			} catch (InvalidKeyException e) {
				e.printStackTrace();
			} catch (IllegalBlockSizeException e) {
				e.printStackTrace();
			} catch (BadPaddingException e) {
				e.printStackTrace();
			} catch (UnsupportedEncodingException e) {
				e.printStackTrace();
			}
			return null;
		}
		
		/**
		 * @desc 解密函数
		 * @param src 密文的字节数组
		 * @return
		 */
		public static byte[] decryptMode(byte[] src) {
			try{
				SecretKey deskey = new SecretKeySpec(build3DesKey(PASSWORD_CRYPT_KEY), Algorithm);
				Cipher c1 = Cipher.getInstance(Algorithm);
				c1.init(Cipher.DECRYPT_MODE, deskey);    //初始化为解密模式
				return c1.doFinal(src);
			}catch(java.security.NoSuchAlgorithmException e1){
				e1.printStackTrace();
			} catch (NoSuchPaddingException e) {
				e.printStackTrace();
			} catch (InvalidKeyException e) {
				e.printStackTrace();
			} catch (IllegalBlockSizeException e) {
				e.printStackTrace();
			} catch (BadPaddingException e) {
				e.printStackTrace();
			} catch (UnsupportedEncodingException e) {
				e.printStackTrace();
			}
			return null;
		}
		
		/**
		 * @desc 根据字符串生成密钥字节数组 
		 * @param keyStr 密钥字符串
		 * @return
		 * @throws UnsupportedEncodingException
		 */
		public static byte[] build3DesKey(String keyStr) throws UnsupportedEncodingException{
			byte[] key = new byte[24];    //声明一个24位的字节数组，默认里面都是0
			byte[] temp = keyStr.getBytes("UTF-8");    //将字符串转成字节数组
			
			if(key.length > temp.length){
				//如果temp不够24位，则拷贝temp数组整个长度的内容到key数组中
				System.arraycopy(temp, 0, key, 0, temp.length);
			}else{
				//如果temp大于24位，则拷贝temp数组24个长度的内容到key数组中
				System.arraycopy(temp, 0, key, 0, key.length);
			}
			return key;
		}
		
		/**
		 * @desc 测试
		 */
		@Test
		public void test(){
			String msg = "rick'Zone";
			System.out.println("明文："+msg);
			//加密
			byte[] secretArr = encryptMode(msg.getBytes());
			System.out.println("密文："+Base64.encode(secretArr));
			//解密
			byte[] myMsgArr = decryptMode(secretArr);
			System.out.println("解密后："+new String(myMsgArr));
		}
	}

3DES的密钥必须是24位的byte数组，随便拿一个String.getBytes()是不行的，会报如下错误：
	
	java.security.InvalidKeyException: Invalid key length: 10 bytes

解决的办法有很多：  
1、按密钥固定长度重新定义字符串；  
2、先把字符串用Base64或者MD5加密，然后截取固定长度的字符转成byte数组；  
3、字符串转成Byte数组，针对该数组进行修改，若长度过长则只截取一部分，若长度不够则补零。  
加密的结果的编码方式要一致，从byte数组转成字符串，一般有两种方式，base64处理和十六进制处理。

#### AES ####

AES 在密码学中是高级加密标准（Advanced Encryption Standard ）的缩写。AES 作为新一代的数据加密标准汇聚了强安全性、高性能、高效率、易用和灵活等优点。

	import java.security.Security;
	import javax.crypto.Cipher;
	import javax.crypto.KeyGenerator;
	import javax.crypto.SecretKey;
	import org.junit.Test;
	import com.thinkgem.jeesite.common.utils.Base64;
	
	public class Aes {

		@Test
		public void test() throws Exception{
			//KeyGenerator提供对称密钥生成器的功能，支持各种算法 
		    KeyGenerator keygen; 
		    //SecretKey 负责保存对称密钥  
		    SecretKey deskey;  
		    //Cipher 负责完成加密或解密工作 
		    Cipher c;         
		    Security.addProvider(new com.sun.crypto.provider.SunJCE()); 
		    // 实例化支持AES 算法的密钥生成器，算法名称用AES 
		    keygen = KeyGenerator.getInstance("AES"); 
		    // 生成密钥 
		    deskey = keygen.generateKey(); 
		    // 生成Cipher 对象，指定其支持AES 算法 
		    c = Cipher.getInstance("AES");                  
		    String msg = "rick_zone"; 
		    System.out.println("明文是：" + msg); 
		    // 根据密钥，对Cipher 对象进行初始化,ENCRYPT_MODE表示加密模式 
		    c.init(Cipher.ENCRYPT_MODE, deskey); 
		    byte[] src = msg.getBytes(); 
		    // 加密，结果保存进enc 
		    byte[] enc = c.doFinal(src); 
		    System.out.println("密文是：" + Base64.encode(enc));   
		    // 根据密钥，对Cipher 对象进行初始化,ENCRYPT_MODE表示加密模式 
		    c.init(Cipher.DECRYPT_MODE, deskey); 
		    // 解密，结果保存进dec 
		    byte[] dec = c.doFinal(enc); 
		    System.out.println("解密后的结果是：" + new String(dec));   
		}
	}

运行结果：

	明文是：rick_zone
	密文是：hpff0pxvbDt+ECtyPYiBKg==
	解密后的结果是：rick_zone

### 非对称加密 ###

非对称加密算法需要两个密钥：公开密钥（publickey）和私有密钥（privatekey）。每个人拥有这两个密钥，公开密钥对外公开，私有密钥不公开。如果用公开密钥对数据进行加密，只有相对应的私有密钥才能解密；如果私有密钥对数据进行加密，那么只有用对应的公开密钥才能解密。  
非对称加密加密算法的基本过程是：

1. 通信前，接收方随机生成公钥，发送给发送方，自己保留私钥。
2. 发送方利用接收方的公钥加密明文，使其变为密文。
3. 接收方收到密文后，使用自己的密钥解密密文。

非对称加密算法的保密性比较好，它消除了最终用户交换密钥的需要，但加密和解密花费时间长、速度慢，它不适合于对文件加密而适用于对少量数据进行加密。  

#### RSA ####

RSA是第一个既能用于数据加密也能用于数字签名的算法。它易于理解和操作，也很流行。这这种加密算法的特点主要是密钥的变化，上面的对称加密算法中只有一个密钥。相当于只有一把钥匙，如果这把钥匙丢了，数据也就不安全了。RSA同时有两把钥匙，公钥与私钥。同时支持数字签名。数字签名的意义在于，对传输过来的数据进行校验。确保数据在传输工程中不被修改。  
RSA流程分析：

1、乙方生成一对密钥（公钥和私钥）并将公钥向其它方公开；  
2、得到该公钥的甲方使用该密钥对机密信息进行加密后再发送给乙方；  
3、乙方再用自己保存的另一把专用密钥（私钥）对加密后的信息进行解密；  
4、乙方只能用其专用密钥（私钥）解密由对应的公钥加密后的信息；  
5、在传输过程中，即使攻击者截获了传输的密文，并得到了乙的公钥，也无法破解密文，因为只有乙的私钥才能解密密文；

	import java.security.Key;
	import java.security.KeyFactory;
	import java.security.KeyPair;
	import java.security.KeyPairGenerator;
	import java.security.NoSuchAlgorithmException;
	import java.security.PrivateKey;
	import java.security.PublicKey;
	import java.security.Signature;
	import java.security.interfaces.RSAPrivateKey;
	import java.security.interfaces.RSAPublicKey;
	import java.security.spec.PKCS8EncodedKeySpec;
	import java.security.spec.X509EncodedKeySpec;
	import java.util.Map;
	import javax.crypto.Cipher;
	import com.google.common.collect.Maps;
	import sun.misc.BASE64Decoder;
	import sun.misc.BASE64Encoder;

	/**
	 * @desc RSA加密解密
	 */
	public class RSACoder {

		//定义加密方式
		private final static String KEY_RSA = "RSA";  
		//定义签名算法
		private final static String KEY_RSA_SIGNATURE = "MD5withRSA";
	    //定义公钥算法
		private final static String KEY_RSA_PUBLICKEY = "RSAPublicKey"; 
	    //定义私钥算法
		private final static String KEY_RSA_PRIVATEKEY = "RSAPrivateKey";   
	    
	    /**
	     * @desc 初始化密钥
	     * @return
	     */
	    public static Map<String, Object> init(){
	    	Map<String, Object> map = null;  
	        try {  
	            KeyPairGenerator generator = KeyPairGenerator.getInstance(KEY_RSA);  
	            generator.initialize(1024);  
	            KeyPair keyPair = generator.generateKeyPair();  
	            // 公钥  
	            RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();  
	            // 私钥  
	            RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();  
	            // 将密钥封装为map  
	            map = Maps.newHashMap();  
	            map.put(KEY_RSA_PUBLICKEY, publicKey);  
	            map.put(KEY_RSA_PRIVATEKEY, privateKey);  
	        } catch (NoSuchAlgorithmException e) {  
	            e.printStackTrace();  
	        }  
	        return map; 
	    }
	    
	    /**
	     * @desc 用私钥对信息生成数字签名 
	     * @param data 加密数据 
	     * @param privateKey 私钥 
	     * @return
	     */
	    public static String sign(byte[] data,String privateKey){
	    	String str = "";  
	        try {  
	            // 解密由base64编码的私钥  
	            byte[] bytes = decryptBase64(privateKey);  
	            // 构造PKCS8EncodedKeySpec对象  
	            PKCS8EncodedKeySpec pkcs = new PKCS8EncodedKeySpec(bytes);  
	            // 指定的加密算法  
	            KeyFactory factory = KeyFactory.getInstance(KEY_RSA);  
	            // 取私钥对象  
	            PrivateKey key = factory.generatePrivate(pkcs);  
	            // 用私钥对信息生成数字签名  
	            Signature signature = Signature.getInstance(KEY_RSA_SIGNATURE);  
	            signature.initSign(key);  
	            signature.update(data);  
	            str = encryptBase64(signature.sign());  
	        } catch (Exception e) {  
	            e.printStackTrace();  
	        }  
	        return str; 
	    }
	    
	    /**
	     * @desc 校验数字签名 
	     * @param data 加密数据 
	     * @param publicKey 公钥
	     * @param sign 数字签名
	     * @return 校验成功返回true，失败返回false 
	     */
	    public static boolean verify(byte[] data, String publicKey, String sign) {  
	        boolean flag = false;  
	        try {  
	            // 解密由base64编码的公钥  
	            byte[] bytes = decryptBase64(publicKey);  
	            // 构造X509EncodedKeySpec对象  
	            X509EncodedKeySpec keySpec = new X509EncodedKeySpec(bytes);  
	            // 指定的加密算法  
	            KeyFactory factory = KeyFactory.getInstance(KEY_RSA);  
	            // 取公钥对象  
	            PublicKey key = factory.generatePublic(keySpec);  
	            // 用公钥验证数字签名  
	            Signature signature = Signature.getInstance(KEY_RSA_SIGNATURE);  
	            signature.initVerify(key);  
	            signature.update(data);  
	            flag = signature.verify(decryptBase64(sign));  
	        } catch (Exception e) {  
	            e.printStackTrace();  
	        }  
	        return flag;  
	    } 
	    
	    /**
	     * @desc 私钥解密 
	     * @param data 加密数据
	     * @param key 私钥
	     * @return
	     */
	    public static byte[] decryptByPrivateKey(byte[] data, String key) {  
	        byte[] result = null;  
	        try {  
	            // 对私钥解密  
	            byte[] bytes = decryptBase64(key);  
	            // 取得私钥  
	            PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(bytes);  
	            KeyFactory factory = KeyFactory.getInstance(KEY_RSA);  
	            PrivateKey privateKey = factory.generatePrivate(keySpec);  
	            // 对数据解密  
	            Cipher cipher = Cipher.getInstance(factory.getAlgorithm());  
	            cipher.init(Cipher.DECRYPT_MODE, privateKey);  
	            result = cipher.doFinal(data);  
	        } catch (Exception e) {  
	            e.printStackTrace();  
	        }  
	        return result;  
	    }
	    
	    /**
	     * @desc 公钥解密
	     * @param data 加密数据
	     * @param key 公钥
	     * @return
	     */
	    public static byte[] decryptByPublicKey(byte[] data, String key) {  
	        byte[] result = null;  
	        try {  
	            // 对公钥解密  
	            byte[] bytes = decryptBase64(key);  
	            // 取得公钥  
	            X509EncodedKeySpec keySpec = new X509EncodedKeySpec(bytes);  
	            KeyFactory factory = KeyFactory.getInstance(KEY_RSA);  
	            PublicKey publicKey = factory.generatePublic(keySpec);  
	            // 对数据解密  
	            Cipher cipher = Cipher.getInstance(factory.getAlgorithm());  
	            cipher.init(Cipher.DECRYPT_MODE, publicKey);  
	            result = cipher.doFinal(data);  
	        } catch (Exception e) {  
	            e.printStackTrace();  
	        }  
	        return result;  
	    } 
	    
	    /**
	     * @desc 公钥加密
	     * @param data 待加密数据
	     * @param key 公钥
	     * @return
	     */
	    public static byte[] encryptByPublicKey(byte[] data, String key) {  
	        byte[] result = null;  
	        try {  
	            byte[] bytes = decryptBase64(key);  
	            // 取得公钥  
	            X509EncodedKeySpec keySpec = new X509EncodedKeySpec(bytes);  
	            KeyFactory factory = KeyFactory.getInstance(KEY_RSA);  
	            PublicKey publicKey = factory.generatePublic(keySpec);  
	            // 对数据加密  
	            Cipher cipher = Cipher.getInstance(factory.getAlgorithm());  
	            cipher.init(Cipher.ENCRYPT_MODE, publicKey);  
	            result = cipher.doFinal(data);  
	        } catch (Exception e) {  
	            e.printStackTrace();  
	        }  
	        return result;  
	    } 
	    
	    /**
	     * @desc 私钥加密
	     * @param data 待加密数据
	     * @param key 私钥
	     * @return
	     */
	    public static byte[] encryptByPrivateKey(byte[] data, String key) {  
	        byte[] result = null;  
	        try {  
	            byte[] bytes = decryptBase64(key);  
	            // 取得私钥  
	            PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(bytes);  
	            KeyFactory factory = KeyFactory.getInstance(KEY_RSA);  
	            PrivateKey privateKey = factory.generatePrivate(keySpec);  
	            // 对数据加密  
	            Cipher cipher = Cipher.getInstance(factory.getAlgorithm());  
	            cipher.init(Cipher.ENCRYPT_MODE, privateKey);  
	            result = cipher.doFinal(data);  
	        } catch (Exception e) {  
	            e.printStackTrace();  
	        }  
	        return result;  
	    } 
	    
	    /**
	     * @desc 获取公钥
	     * @param map
	     * @return
	     */
	    public static String getPublicKey(Map<String, Object> map) {  
	        String str = "";  
	        try {  
	            Key key = (Key) map.get(KEY_RSA_PUBLICKEY);  
	            str = encryptBase64(key.getEncoded());  
	        } catch (Exception e) {  
	            e.printStackTrace();  
	        }  
	        return str;  
	    } 
	    
	    /**
	     * @desc 获取私钥
	     * @param map
	     * @return
	     */
	    public static String getPrivateKey(Map<String, Object> map) {  
	        String str = "";  
	        try {  
	            Key key = (Key) map.get(KEY_RSA_PRIVATEKEY);  
	            str = encryptBase64(key.getEncoded());  
	        } catch (Exception e) {  
	            e.printStackTrace();  
	        }  
	        return str;  
	    }  
	    
	    /**
	     * @desc BASE64解密 
	     * @param key
	     * @return
	     * @throws Exception
	     */
	    public static byte[] decryptBase64(String key) throws Exception {  
	        return (new BASE64Decoder()).decodeBuffer(key);  
	    }  
	      
	    /**
	     * @desc BASE64加密 
	     * @param key
	     * @return
	     * @throws Exception
	     */
	    public static String encryptBase64(byte[] key) throws Exception {  
	        return (new BASE64Encoder()).encodeBuffer(key);  
	    }  
	
	    /**
	     * @desc 测试
	     */
	    public static void main(String[] args) {  
	        String privateKey = "";  
	        String publicKey = "";  
	        // 生成公钥私钥  
	        Map<String, Object> map = init();  
	        publicKey = getPublicKey(map);  
	        privateKey = getPrivateKey(map);  
	        System.out.println("公钥: \n\r" + publicKey);  
	        System.out.println("私钥： \n\r" + privateKey);  
	        System.out.println("公钥加密--------私钥解密");  
	        String word = "你好，世界！";  
	        byte[] encWord = encryptByPublicKey(word.getBytes(), publicKey);  
	        String decWord = new String(decryptByPrivateKey(encWord, privateKey));  
	        System.out.println("加密前: " + word + "\n\r" + "解密后: " + decWord);  
	        System.out.println("私钥加密--------公钥解密");  
	        String english = "Hello, World!";  
	        byte[] encEnglish = encryptByPrivateKey(english.getBytes(), privateKey);  
	        String decEnglish = new String(decryptByPublicKey(encEnglish, publicKey));  
	        System.out.println("加密前: " + english + "\n\r" + "解密后: " + decEnglish);  
	        System.out.println("私钥签名——公钥验证签名");  
	        // 产生签名  
	        String sign = sign(encEnglish, privateKey);  
	        System.out.println("签名:\r" + sign);  
	        // 验证签名  
	        boolean status = verify(encEnglish, publicKey, sign);  
	        System.out.println("状态:\r" + status);  
	    }
	}

测试结果：

	公钥: 

	MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDQblxGcna68ErW1gy02oGZsCvuddliTp8HMXJh
	k8Fgfy337dMb293uHNYmUmhsP0Jlkku1+5H53fb2iP9lYlZbCATjtKw9pSWeabMYizSoV1vRXvHc
	ts3vGn3efeJUf6IU7pAFihe9zmrCWuNdNfSp+ZrgYUNKVYOB+6S1ktPSsQIDAQAB
	
	私钥： 
	
	MIICeAIBADANBgkqhkiG9w0BAQEFAASCAmIwggJeAgEAAoGBANBuXEZydrrwStbWDLTagZmwK+51
	2WJOnwcxcmGTwWB/Lfft0xvb3e4c1iZSaGw/QmWSS7X7kfnd9vaI/2ViVlsIBOO0rD2lJZ5psxiL
	NKhXW9Fe8dy2ze8afd594lR/ohTukAWKF73OasJa41019Kn5muBhQ0pVg4H7pLWS09KxAgMBAAEC
	gYEAiPV7vO7KBdyP0MumXdCXCJ4yv+bAiVCQPvHG70df8kCfvCKSbePz2Nsp/NR5uWd4AbY3+cTt
	DvtDpIwxBjWv98CswHgwqKEq6hLleCNiC2FSikXywJ0RkU6IPx1L0d4P5e9YhYwU4xADk0M8n3KD
	rzzP1NLdoNJ4/DQk5jsQUIECQQD6F4HCLxIJJtZb/lOlaegYB+8teXIrXrRdhcmZrBuD75Jow/lO
	26AqF6QRwKGQmFSGljUCPRjSsvkRifsUGZ3VAkEA1VrmXUuolCCTRzSVqxfxi0nsOZKg83n8rO7e
	mIfuQn/WdyicxxX65WQTo9PqNaFgVkBwYmLcPY+IN9qPUDmjbQJAetK0nWN0nh3+xKtA9Unv/G1Z
	H6I3Welm909PVTNbSA6OfvlQJVRjcoitwTIzpsnJKVf3rKPI3yGahOyY7KQwCQJBAKnTbya4AHnX
	7CNzoebMakHbF6NEKcVkRlJI2PpEyMw6AbZbp195CXrqTA/NsNH7oDlHla1az8BYra7307eiCYkC
	QQDl1M+re4CgRJCy/mCdoymR108+BVlABVIF6Y7ihGEYLiznwRXyz/vxgu2YlOR/DLNIS4tgGY9v
	o6D1xDF39H2N
	
	公钥加密--------私钥解密
	加密前: 你好，世界！
	
	解密后: 你好，世界！
	私钥加密--------公钥解密
	加密前: Hello, World!
	
	解密后: Hello, World!
	私钥签名——公钥验证签名
	签名:
	kOWUL2Rvu7VGStpo4Qh+a9tNeASXdk6cxHXYUG4ExQQpiKD0ApPkby7yfrLX7qPgwmK5UGnDw9CW
	w1LbgvEObc9/HY0zdgWbh8t67tQz6xVcVPT3fNxpcnUi4e/Pe0+TdISQ/xmqh/S9tHncCMAvB3OJ
	+eULz1x3fE0RjIqhGVw=
	
	状态:
	true

看似很复杂的过程，用一句话就可以描述：使用公钥加密、私钥解密，完成了乙方到甲方的一次数据传递，通过私钥加密、公钥解密，同时通过私钥签名、公钥验证签名，完成了一次甲方到乙方的数据传递与验证，两次数据传递完成一整套的数据交互。

# 总结 #
---
本文介绍了常用的各种加密方法，在这里只介绍如何使用，不做深入探究，如果想深入了解，也可以查其他书籍资料，以及参考中的博客文章。里面还包括其他的未提到的加密算法。  
在后面的博客中介绍base64编码和非对称加密在实际项目中的应用。

# 参考 #
---
[java加密技术](http://blog.csdn.net/happylee6688/article/category/2902645)

