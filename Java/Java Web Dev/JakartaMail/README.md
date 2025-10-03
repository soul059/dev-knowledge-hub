# Jakarta Mail

Jakarta Mail (formerly JavaMail) is a Jakarta EE API used for sending and receiving email. It provides a platform-independent and protocol-independent framework to build mail and messaging applications.

## Key Concepts

*   **`Session`**: Represents a mail session. It's the entry point to the API, used to get `Transport` and `Store` objects and to authenticate.
*   **`Message`**: Represents an email message. The `javax.mail.internet.MimeMessage` class is the most commonly used implementation.
*   **`Address`**: Represents an email address. The `javax.mail.internet.InternetAddress` class is used for email addresses.
*   **`Transport`**: An abstract class that models a message transport protocol (like SMTP).
*   **`Store`**: An abstract class that models a message storage protocol (like POP3 or IMAP).

## Maven Dependencies

To use Jakarta Mail, you need to add the following dependencies to your `pom.xml`.

```xml
<!-- Jakarta Mail API -->
<dependency>
    <groupId>jakarta.mail</groupId>
    <artifactId>jakarta.mail-api</artifactId>
    <version>2.1.0</version>
</dependency>

<!-- Jakarta Mail Implementation -->
<dependency>
    <groupId>org.eclipse.angus</groupId>
    <artifactId>angus-mail</artifactId>
    <version>1.0.0</version>
</dependency>
```

## Sending Email (SMTP)

To send an email, you need to configure an SMTP server.

### Steps to Send an Email:

1.  **Get a `Session` object**: Configure properties for the SMTP server (host, port, authentication).
2.  **Create a `MimeMessage` object**: This will be the email message.
3.  **Set the sender and recipients**: Use `setFrom()` and `addRecipient()` with `Message.RecipientType.TO`, `CC`, or `BCC`.
4.  **Set the subject and content**: Use `setSubject()` and `setText()` or `setContent()` for multipart messages.
5.  **Send the message**: Use `Transport.send(message)`.

### Example: Sending a Simple Text Email

```java
import jakarta.mail.*;
import jakarta.mail.internet.*;
import java.util.Properties;

public class EmailSender {

    public static void main(String[] args) {

        final String username = "your-email@example.com";
        final String password = "your-password";
        final String host = "smtp.example.com";

        Properties props = new Properties();
        props.put("mail.smtp.auth", "true");
        props.put("mail.smtp.starttls.enable", "true");
        props.put("mail.smtp.host", host);
        props.put("mail.smtp.port", "587");

        Session session = Session.getInstance(props, new Authenticator() {
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication(username, password);
            }
        });

        try {
            Message message = new MimeMessage(session);
            message.setFrom(new InternetAddress("from-email@example.com"));
            message.setRecipients(Message.RecipientType.TO, InternetAddress.parse("to-email@example.com"));
            message.setSubject("Testing Jakarta Mail");
            message.setText("This is a test email sent using Jakarta Mail.");

            Transport.send(message);

            System.out.println("Email sent successfully!");

        } catch (MessagingException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## Reading Email (IMAP/POP3)

To read emails, you connect to a `Store` (like an IMAP or POP3 server), get a `Folder` (like the inbox), and then fetch the messages.

### Example: Reading from INBOX

```java
// ... (session properties for IMAP server)

Session session = Session.getDefaultInstance(props);

try {
    Store store = session.getStore("imaps");
    store.connect("imap.example.com", "your-email@example.com", "your-password");

    Folder inbox = store.getFolder("INBOX");
    inbox.open(Folder.READ_ONLY);

    Message[] messages = inbox.getMessages();
    for (Message message : messages) {
        System.out.println("Subject: " + message.getSubject());
        System.out.println("From: " + message.getFrom()[0]);
        // ... process message content
    }

    inbox.close(false);
    store.close();

} catch (Exception e) {
    e.printStackTrace();
}
```