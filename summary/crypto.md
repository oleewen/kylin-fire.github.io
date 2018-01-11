# 常用算法

MD5、SHA、HMAC这三种加密算法，可谓是非可逆加密，就是不可解密的加密方法，我们称之为单向加密算法。我们通常只把他们作为加密的基础。单纯的以上三种的加密并不可靠。MD5、SHA以及HMAC是单向加密，任何数据加密后只会产生唯一的一个加密串，通常用来校验数据在传输过程中是否被修改。其中HMAC算法有一个密钥，增强了数据传输过程中的安全性，强化了算法外的不可控因素。

MD5

MD5 -- message-digest algorithm 5 （信息-摘要算法）缩写，广泛用于加密和解密技术，常用于文件校验。校验？不管文件多大，经过MD5后都能生成唯一的MD5值。好比现在的ISO校验，都是MD5校验。怎么用？当然是把ISO经过MD5后产生MD5的值。一般下载linux-ISO的朋友都见过下载链接旁边放着MD5的串。就是用来验证文件是否一致的。

       维基百科上有比较详细的算法介绍：http://zh.wikipedia.org/wiki/Md5，而百度百科也有类似的介绍：http://baike.baidu.com/view/7636.htm，有兴趣的同学可以仔细阅读。

Java语言中对于Md5已经有了比较完整的实现，如java.security中的MessageDigest实现，而apache开源组织也提供了一个包装时间DigestUtils（commons-codes：1.7）。

DigestUtils包括了常用的mdX算法，官方API：http://commons.apache.org/codec/apidocs/org/apache/commons/codec/digest/DigestUtils.html

 包装实现：



   public static String md5Hex(String str) {

      if (StringUtil.isBlank(str)) {

         return SignConsts.EMPTY;

      }

      // 直接是apache的库

      return DigestUtils.md5Hex(str);

   }





   public static String md5Hex(byte[] bytes) {

      if (bytes == null || bytes.length == 0) {

         return SignConsts.EMPTY;

      }

      // 直接是apache的库

      return DigestUtils.md5Hex(bytes);

   }





   public static byte[] md5(String str) {

      if (StringUtil.isBlank(str)) {

         return new byte[0];

      }

      // 直接是apache的库

      return DigestUtils.md5(str);

   }





   public static byte[] md5(byte[] bytes) {

      if (bytes == null || bytes.length == 0) {

         return new byte[0];

      }

      // 直接是apache的库

      return DigestUtils.md5(bytes);

   }



SHA

SHA(Secure Hash Algorithm，安全散列算法），数字签名等密码学应用中重要的工具，被广泛地应用于电子商务等信息安全领域。虽然，SHA与MD5通过碰撞法都被破解了，但是SHA仍然是公认的安全加密算法，较之MD5更为安全。

       维基百科上有比较详细的算法介绍：http://zh.wikipedia.org/wiki/SHA家族，而百度百科也有类似的介绍：http://baike.baidu.com/view/531723.htm ，有兴趣的同学可以仔细阅读。

Java语言中对于Md5已经有了比较完整的实现，如java.security中的MessageDigest实现，而apache开源组织也提供了一个包装时间DigestUtils（commons-codes：1.7）。

DigestUtils包括了常用的shaX算法，官方API：http://commons.apache.org/codec/apidocs/org/apache/commons/codec/digest/DigestUtils.html

 包装实现：



   public static String shaHex(String str) {

      if (StringUtil.isBlank(str)) {

         return SignConsts.EMPTY;

      }

      // 直接是apache的库

      return DigestUtils.shaHex(str);

   }





   public static String shaHex(byte[] bytes) {

      if (bytes == null || bytes.length == 0) {

         return SignConsts.EMPTY;

      }

      // 直接是apache的库

      return DigestUtils.shaHex(bytes);

   }





   public static byte[] sha(String str) {

      if (StringUtil.isBlank(str)) {

         return new byte[0];

      }

      // 直接是apache的库

      return DigestUtils.sha(str);

   }





   public static byte[] sha(byte[] bytes) {

      if (bytes == null || bytes.length == 0) {

         return new byte[0];

      }

      // 直接是apache的库

      return DigestUtils.sha(bytes);

   }



HMAC

HMAC(Hash Message Authentication Code，散列消息鉴别码，基于密钥的Hash算法的认证协议。消息鉴别码实现鉴别的原理是，用公开函数和密钥产生一个固定长度的值作为认证标识，用这个标识鉴别消息的完整性。使用一个密钥生成一个固定大小的小数据块，即MAC，并将其加入到消息中，然后传输。接收方利用与发送方共享的密钥进行鉴别认证等。

      维基百科上有比较详细的算法介绍：http://zh.wikipedia.org/wiki/Hmac ，而百度百科也有类似的介绍：http://baike.baidu.com/view/1136366.htm ，有兴趣的同学可以仔细阅读。

Java语言中对于Md5已经有了比较完整的实现，如java.security中的Mac实现，而apache开源组织暂时没有找到相关实现。



包装实现：






   public static Key hmacKey(String algorithm) {

      if (StringUtil.isBlank(algorithm)) {

         return null;

      }



      try {

         KeyGenerator keyGenerator = KeyGenerator.getInstance(algorithm);

         SecretKey secretKey = keyGenerator.generateKey();

         return secretKey;

      } catch (Exception e) {

         return null;

      }

   }





   public static String key2str(Key key) {

      if (key != null) {

         byte[] bytes = key.getEncoded();

         if (bytes != null && bytes.length > 0) {

            return base64EncodeStr(bytes);

         }

      }

      return SignConsts.EMPTY;

   }







   public static Key str2key(String key, String algorithm) {

      if (StringUtil.isBlank(key) || StringUtil.isBlank(algorithm)) {

         return null;

      }

      SecretKey secretKey = new SecretKeySpec(base64Decode(key), algorithm);

      return secretKey;

   }





   public static String hmacHex(String str, Key key) {

      return hmacHex(str.getBytes(), key);

   }





   public static String hmacHex(byte[] bytes, Key key) {

      return new String(Hex.encodeHex(hmac(bytes, key)));

   }





   public static byte[] hmac(String str, Key key) {

      return hmac(str.getBytes(), key);

   }





   public static byte[] hmac(byte[] bytes, Key key) {

      if (bytes == null || bytes.length == 0) {

         return new byte[0];

      }

      try {

         Mac mac = Mac.getInstance(key.getAlgorithm());

         mac.init(key);

         return mac.doFinal(bytes);

      } catch (Exception e) {

         return new byte[0];

      }

   }



参考文档

http://www.iteye.com/topic/1122076

http://security.group.iteye.com/group/wiki/1710-one-way-encryption-algorithm

http://aubdiy.blog.51cto.com/2978849/815656
