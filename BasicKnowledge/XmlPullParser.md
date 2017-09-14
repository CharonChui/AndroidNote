XmlPullParser
===

```java
public class PersonService {
    /**
     * 接收一个包含XML文件的输入流, 解析出XML中的Person对象, 装入一个List返回
     * @param in    包含XML数据的输入流
     * @return        包含Person对象的List集合
     */
    public List<Person> getPersons(InputStream in) throws Exception {
        //1.获取xml文件
        InputStream is = PersonService.class.getClassLoader().getResourceAsStream();
        //2.获取解析器(Android中提供了方便的方法就是使用Android中的工具类Xml)
        XmlPullParser parser = Xml.newPullParser();        
        //3.解析器解析xml文件
        parser.setInput(in, "UTF-8");                    

        List<Person> persons = new ArrayList<Person>();
        Person p = null;
        //4.循环解析
        for (int type = parser.getEventType(); type != XmlPullParser.END_DOCUMENT; type = parser.next()) {    // 循环解析
            if (type == XmlPullParser.START_TAG) {                // 判断如果遇到开始标签事件
                if ("person".equals(parser.getName())) {        // 标签名为person
                    p = new Person();                            // 创建Person对象
                    String id = parser.getAttributeValue(0);    // 获取属性
                    p.setId(Integer.parseInt(id));                // 设置ID
                    persons.add(p);                                // 把Person对象装入集合
                } else if ("name".equals(parser.getName())) {    // 标签名为name
                    String name = parser.nextText();            // 获取下一个文本
                    p.setName(name);                            // 设置name
                } else if ("age".equals(parser.getName())) {    // 标签名为age
                    String age = parser.nextText();                // 获取下一个文本
                    p.setAge(Integer.parseInt(age));            // 设置age
                }
            }
        }
        return persons;
    }

    /**
    *将数据写入到Xml文件中.
    *@param out    输出到要被写入数据的Xml文件的输出流//就相当于 OutputStream os = new FileOutputStream("a.xml");
    */
    public void writePersons(List<Book> Books, OutputStream out) throws Exception {

        //1.获得XmlSerializer(Xml序列化工具)(通过Android中的工具类Xml得到)
        XmlSerializer serializer = Xml.newSerializer();    
        //2.设置输出流(明确要将数据写入那个xml文件中)
        serializer.setOutput(out, "UTF-8");                
        //3.写入开始文档
        serializer.startDocument("UTF-8", true);
        //4.开始标签
        serializer.startTag(null, "bookstore");
        //5.循环遍历
        for (Book p : books) {
            //6.开始标签
            serializer.startTag(null, "book");
            //7.给这个标签设置属性
            serializer.attribute(null, "id", book.getId().toString());
            //8.子标签
            serializer.startTag(null, "name");
            //9.设置子标签的内容
            serializer.text(book.getName());
            //10.子标签结束
            serializer.endTag(null, "name");
            //11.标签结束
            serializer.endTag(null, "person");
        }
        //12.根标签结束
        serializer.endTag(null, "persons");
        //13.文档结束
        serializer.endDocument();
    }
}
```

---
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 