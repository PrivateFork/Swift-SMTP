# Kitura-SMTP

Swift package for sending emails to an SMTP server.

Dependencies: [BlueSocket](https://github.com/IBM-Swift/BlueSocket.git), [BlueSSLService](https://github.com/IBM-Swift/BlueSSLService), [BlueCryptor](https://github.com/IBM-Swift/BlueCryptor.git), [LoggerAPI](https://github.com/IBM-Swift/LoggerAPI)

## Features

- Connect securely through SSL/TLS if available
- Authenticate with CRAM-MD5, LOGIN, PLAIN, or XOAUTH2
- Send emails with local file, HTML, and raw data attachments
- Add custom headers
- Asynchronous

## Usage

Use the `SMTP` struct as a handle to an SMTP server:

```swift
import KituraSMTP

let smtp = SMTP(hostname: "smtp.gmail.com",             // SMTP server address
                user: "user@gmail.com",                 // username to login 
                password: "password")                   // password to login

/* Additional parameters available for further customization */
```

### Send email

Create a `Mail` object and use your `smtp` handle to send it. To set the sender and receiver of an email, use the `User` struct:

```swift
let from = User(name: "Dr. Light", email: "drlight@gmail.com")
let to = User(name: "Megaman", email: "megaman@gmail.com")

let mail = Mail(from: from,
                to: [to],
                subject: "Humans and robots living together in harmony and equality.",
                text: "That was my ultimate wish.")

smtp.send(mail) { (err) in
    if let err = err {
        print(err)
    }
}
```

Add Cc and Bcc:

```swift
let roll = User(name: "Roll", email: "roll@gmail.com")
let zero = User(name: "Zero", email: "zero@gmail.com")

let mail = Mail(from: from,
                to: [to],
                cc: [roll],
                bcc: [zero],
                subject: "Humans and robots living together in harmony and equality.",
                text: "That was my ultimate wish.")

smtp.send(mail)

```

### Send attachments

Create an `Attachment`, attach it to your `Mail`, and send it through the `smtp` handle. Here's an example of how you can send the three supported types of attachments--a local file, HTML, and raw data:

```swift
let fileAttachment = Attachment(filePath: "~/img.png",
                                // You can add "CONTENT-ID" to reference this in another attachment
                                additionalHeaders: ["CONTENT-ID": "img001"])

let htmlAttachment = Attachment(htmlContent: "<html>Here's an image: <img src=\"cid:img001\"/></html>", 
                                related: [fileAttachment]) // to reference `fileAttachment`

let data = "{\"key\": \"hello world\"}".data(using: .utf8)!
let dataAttachment = Attachment(data: data, 
                                mime: "application/json", 
                                name: "file.json",
                                inline: false) // send as a standalone attachment

let mail = Mail(from: from, 
                to: [to], 
                subject: "Check out this image and JSON file!", 
                attachments: [htmlAttachment, dataAttachment]) // attachments we created earlier

smtp.send(mail)

/* Each type of attachment has additional parameters for further customization */
```

### Send multiple mails

```swift
let mail1: Mail = //...
let mail2: Mail = //...

smtp.send([mail1, mail2], 
        progress: { (mail, error) in
            // this optional callback gets called after each mail is sent
            // `mail` is the attempted mail, `error` is the error if one occured
        },
        completion: { (sent, failed) in
            // this optional callback gets called after all the mails have been sent
            // `sent` is an array of the successfully sent mails
            // `failed` is an array of (Mail, Error)--the failed mails and their corresponding errors
        }
)
```

## Acknowledgements

`Kitura-SMTP` was inspired by [Hedwig](https://github.com/onevcat/Hedwig) and [Perfect-SMTP](https://github.com/PerfectlySoft/Perfect-SMTP), two Swift libraries that can also be used to send emails to an SMTP server.

## License

Apache 2.0