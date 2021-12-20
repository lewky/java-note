<!--
date: 2021-12-08T22:34:12+08:00
lastmod: 2021-12-20T22:34:12+08:00
-->
## 建造者模式（Builder Pattern）

建造者模式通过引入一个Builder类来创建一个复杂的大对象，对客户端屏蔽创建复杂大对象的具体细节。这个复杂对象由许多其他的对象构成，通常很多工具类都会使用到建造者模式来创建复杂的对象。

## 样例代码

比如创建一封邮件，需要设置发送方，接收方，邮件主题、内容，发送日期等。

```java
@Data
public class Email {

    private String from;
    private List<Recipient> recipients;
    private String subject;
    private String content;
    private LocalDate sentDate;

}

@Data
public class Recipient {

    private RecipientType type;
    private String internetAddress;

}

public enum RecipientType {

    TO,CC
}

public class EmailBuilder {

    public Email buildEmail() {
        final Email email = new Email();
        email.setFrom("from");

        final List<Recipient> recipients = new ArrayList<>();
        final Recipient to = new Recipient();
        // build to
        to.setType(RecipientType.TO);
        to.setInternetAddress("to");
        recipients.add(to);
        // build cc
        final Recipient cc = new Recipient();
        to.setType(RecipientType.CC);
        to.setInternetAddress("cc");
        recipients.add(cc);
        email.setRecipients(recipients);

        email.setSubject("subject");
        email.setContent("content");
        email.setSentDate(LocalDate.now());

        return email;
    }
}
```

这里为了减少setter和getter代码的书写，使用了`@Data`注解。

## 参考链接

* [建造者模式](https://www.runoob.com/design-pattern/builder-pattern.html)