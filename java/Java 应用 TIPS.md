# Java 应用 TIPS

聚合 Java 语言中的一些冷门场景的解决方案

## json

### 自定义 json 序列化与反序列化

首先看一个 json，如下：

```json
{
    "name": "zhangsan",
    "remark": {
        "date": "2019-05-23",
        "message": "over due"
    }
}
```

现在，我们想要 remark 节点的值，也是一个 json，但现在我希望用字符串的方式进行接收，在代码里，我们会用对象接收，会出现转化 Error。那我们怎么实现呢？使用自定义序列化的方式。

```java
@JsonSerialize(using = StringSerializer.class)
@JsonDeserialize(using = StringDeserializer.class)
private String remark;
...
public static class StringSerializer extends JsonSerializer<String> {
    public StringSerializer() {
    }

    @Override
    public void serialize(String value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeString(value);
    }
}

public static class StringDeserializer extends JsonDeserializer<String>{
    public StringDeserializer() {
    }

    @Override
    public String deserialize(JsonParser p, DeserializationContext ctxt) throws IOException {
        return p.getValueAsString();
    }
}
```

这边需要注意下方的序列化与反序列化类都要是静态类，否则是不生效的。

> @JsonSerialize，@JsonDeserialize 还有很多参数内容，按需查询

## 日期

### Java 8 日期格式

Java 7 中的 SimpleDateFormat 是非线程安全的，常见开发者使用 static 方式进行声明 sdf，在大数据量场景下，会报错。

在 Java 8 中存在 DateTimeFormatter 承担日期格式转换职责，我们举个例子看一下

```java
DateTimeFormatter dtf = DateTimeFormatter.ISO_DATE_TIME;
LocalDateTime ldt = LocalDateTime.now();

String strDate = ldt.format(dtf);
System.out.println(strDate);

DateTimeFormatter dtf2 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String strDate2 = dtf2.format(ldt);
System.out.println(strDate2);
LocalDateTime newDate = ldt.parse(strDate2, dtf2);
System.out.println(newDate);
```

