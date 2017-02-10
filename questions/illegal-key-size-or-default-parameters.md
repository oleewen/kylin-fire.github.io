# Java加密抛出java.security.InvalidKeyException: Illegal key size or default parameters解决方法

## 来自stackoverflow的帖子
[http://stackoverflow.com/questions/6481627/java-security-illegal-key-size-or-default-parameters](http://stackoverflow.com/questions/6481627/java-security-illegal-key-size-or-default-parameters)

## 原理

因美国出口限制，Sun通过权限文件（local_policy.jar、US_export_policy.jar）对加密算法做了相应限制。

限制的加密算法会存在以下问题：
- 密钥长度上不能满足需求（如：java.security.InvalidKeyException: Illegal key size or default parameters）；
- 部分算法未能支持，如MD4、SHA-224等算法；
- API不够方便；一些常用Base64编码转换、十六进制编码转换等工具未能提供。

## 解决办法

### 1.下载非限制权限文件

下载以下各版本的非限制权限文件:

[Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files 6 Download](http://www.oracle.com/technetwork/java/javase/downloads/jce-6-download-429243.html)

[Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files 7 Download](http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html)

[Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files 8 Download](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)

### 2.替换jre下的安全文件

解压下载zip文件，替换到${JAVA_HOME}/jre/lib/security/
    
  Mac电脑：JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_73.jdk/Contents/Home/
    
  Win电脑：JAVA_HOME=c:\Program Files\java\jdk1.8.0_73.jdk\
