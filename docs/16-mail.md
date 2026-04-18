[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">

# Mail

Modul untuk mengirim email.

``` java
public interface MailHandler {
	void send(MailObject mailObject) throws Exception;	
	void send(MailObject mailObject, boolean asynchronous) throws Exception;
}

public class MailObject {
	private InternetAddress from;	
	private InternetAddress[] to;	
	private InternetAddress[] cc;	
	private InternetAddress[] bcc;	
	private String subject;	
	private String plainText;
	private String htmlText;
	private Inline[] inline;
	private Attachment[] attachment;
	private boolean multipart = false;
	private String encoding = "UTF-8";
}
```

## Bean

``` java
@Singleton
MailHandler mailHandler(TaskHandler taskHandler) {
    return new MailHandlerImpl()
    .setTaskHandler(taskHandler)
    .setMailProperties(appProperties.getMail());
}
```

## Contoh Properties

``` md
mail:
    host: smtp.gmail.com
    port: 587
    username: "<username>"
    password: "<password>"
    properties:
        "mail.smtp.host": "smtp.gmail.com"
        "mail.smtp.ssl.trust": "smtp.gmail.com"
        "mail.smtp.port": "587"
        "mail.smtp.auth": "true"
        "mail.smtp.starttls.enable": "true"
        "mail.imap.ssl.enable": "true"
        "mail.transport.protocol": "smtp"
        "mail.debug": "true"
        "mail.smtp.ssl.protocols": "TLSv1.2"
```

##

[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">
