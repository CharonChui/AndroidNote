MD5加密
===

`MD5`是一种不可逆的加密算法只能将原文加密，不能讲密文再还原去，原来把加密后将这个数组通过`Base64`给变成字符串，
这样是不严格的业界标准的做法是对其加密之后用每个字节`&15`然后就能得到一个`int`型的值，再将这个`int`型的值变成16进制的字符串.虽然MD5不可逆，
但是网上出现了将常用的数字用`md5`加密之后通过数据库查询，所以`MD5`简单的情况下仍然可以查出来，一般可以对其多加密几次或者`&15`之后再和别的数运算等，
这称之为*加盐*.
 
```java
public class MD5Utils {
    /**
     * md5加密的工具方法
     */
    public static String encode(String password){
        try {
            MessageDigest digest = MessageDigest.getInstance("md5");
            byte[] result = digest.digest(password.getBytes());
            StringBuilder sb = new StringBuilder();//有的数很小还不到10所以得到16进制的字符串有一个
                                                   //的情况，这里对于小于10的值前面加上0
            //16进制的方式  把结果集byte数组 打印出来
            for(byte b :result){
                int number = (b&0xff);//加盐.
                String str =Integer.toHexString(number);
                if(str.length()==1){
                    sb.append("0");
                }
                sb.append(str);
            }
            return sb.toString();
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
            return "";
        }
    }
}
```

----
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 