RMB大小写转换
=========

```java
public class RenMingBi {
    private static final char[] data = new char[] { '零', '壹', '贰', '叁', '肆', '伍', '陆', '柒', '捌', '玖' };
    private static final char[] units = new char[] { '元', '拾', '佰', '仟', '万', '拾', '佰', '仟', '亿' };

    public static void main(String[] args) {
        System.out.println(convert(11));
    }

    public static String convert(int money) {
        StringBuffer sbf = new StringBuffer();
        int unit = 0;
        while (money != 0) {
            sbf.insert(0, units[unit++]);
            System.out.println(sbf.toString());
            int number = money % 10;
            sbf.insert(0, data[number]);
            money /= 10;
        }
        return sbf.toString();
    }
}
```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
