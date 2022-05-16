---
title: "Build your own HTTP server in Java in less than one hour (only GET method)"
date: 2020-03-30T15:00:00Z
cover: "cover.png"
useRelativeCover: true
draft: false
description: "Every time you visit a website your web browser uses the HTTP protocol to communicate with web server and fetch the page's content. Also, when you are implementing backend app and you have to communicate with other backend app - 80% (or more) of cases you will use the HTTP."
---

Post was originally posted on [my old dev.to blog](https://dev.to/mateuszjarzyna/build-your-own-http-server-in-java-in-less-than-one-hour-only-get-method-2k02)

## One of the most frequency used protocol in the whole Internet \*

**\* In OSI model, layer 7**

Every time you visit a website your web browser uses the HTTP protocol to communicate with web server and fetch the page's content. Also, when you are implementing backend app and you have to communicate with other backend app - 80% (or more) of cases you will use the HTTP.

Long story short - when you want to be a good software developer, you have to know how the HTTP protocol works. And wiring the HTTP server is pretty good way to understood, I think.
What a web browser sends to the web server?

Good question. Of course, you can use "developer tools", let's do it.

##  What a web browser sends to the web server? 

Good question. Of course, you can use "developer tools", let's do it.

{{< image src="images/developer-tools.png" alt="Developer tools" position="center" style="border-radius: 8px;" >}}

Hmm... but what now? What exactly it means? We can see some URL, some method, some status, version (huh?), headers, and other stuff. Useful? Yes, but only to analyze the web app, when something is wrong. We still don't know how HTTP works.

###  Wireshark, my old friend 

The source of truth. Wireshark is application to analyze network traffic. You can use it to see each packet that is sent by your (or to your) PC.

{{< image src="images/wireshark.png" alt="Wireshark" position="center" style="border-radius: 8px;" >}}

But to be honest - if you know how to use Wireshark - you probably know how HTTP and TCP works. It's pretty advanced program.

### You are right - the specification

Every good (I mean - used by more that 5 peoples) protocols should have specification. In this case it's called [RFC](https://en.wikipedia.org/wiki/Request_for_Comments). But don't lie - you will never read this, it's too long - https://tools.ietf.org/html/rfc2616 .

### Just run the server and test

Joke? No. Probably you have installed on your PC very powerful tool called netcat, it's pretty advanced tool.
One of the netcat features is TCP server. You can run the netcat to listen on specific port and print every thing what it gets. Netcat is a command line app.

```bash
nc -v -l -p 8080
```

Netcat (`nc`), please, listen (`-l`) on port `8080` (`-p 8080`) and print everything (`-v`).

Now open web browser and enter `http://localhost:8080/`. Your browser will send the HTTP request to the server runned by netcat. Of course `nc` will print the whole request and ignore it, browser will wait for the response (will timeout soon). To kill `nc` press `ctrl+c`.

[![asciicast](https://asciinema.org/a/313443.svg)](https://asciinema.org/a/313443)

So, finally, we have an HTTP request!

```
GET / HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:74.0) Gecko/20100101 Firefox/74.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Cookie: JSESSIONID=D3AF43EBFC0C9D92AD9C37823C4BB299
Upgrade-Insecure-Requests: 1
```

As you can see - it fully texts protocol. No bits to analyze, just plain text.

## HTTP request

It may be a little confusing. Maybe `nc` parses the request before printing? HTTP protocol should be complicated, where is the sequence of 0 and 1? There aren't any. HTTP is really very simple text protocol. There is only one, little trap (I will explain it at the end of this section).

We can split the request to the 4 main parts:

```
GET / HTTP/1.1
```

This is the *main* request.

`GET` - this is the HTTP method. Probably you know [there are a lot of methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods).
`GET` means `give me`

`/` - resource. `/` means *default one*.
When you will open `localhost:8080/my_gf_nudes.html`, the resource will be `/my_gf_nudes.html`

`HTTP/1.1` - HTTP version. There are few versions, 1.1 is commonly used.

```
Host: localhost:8080
```

Host. One server can host many domains, using this field, the browser says which domain it wants exactly

```
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:74.0) Gecko/20100101 Firefox/74.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Cookie: JSESSIONID=D3AF43EBFC0C9D92AD9C37823C4BB299
Upgrade-Insecure-Requests: 1
```

Headers. In short: some additional informations. But I'm sure that you know what headers are :)

```

```

Surprise - empty line. It means: end of the request. In general - empty line in HTTP means end of section.

### The trap

In HTTP every new line separator is a Window's new line. `\r\n` not `\n`. Remember.

### Response

Ok. We have a request. How does response look like? Send a request to any server and see, there is nothing simpler.
On your laptop you can find another very useful tool - telnet. Using `telenet` you can open TCP connection, write something to server and print the response.
Try to do it yourself. Run `telnet google.com 80` (80 is the default HTTP port) and type request manually (you know how it should look like). To close connection press `ctrl+]` then type `quit`.

[![asciicast](https://asciinema.org/a/313466.svg)](https://asciinema.org/a/313466)

OK. We have a response.

```
HTTP/1.1 301 Moved Permanently
Location: http://www.google.com/
Content-Type: text/html; charset=UTF-8
Date: Wed, 25 Mar 2020 18:53:12 GMT
Expires: Fri, 24 Apr 2020 18:53:12 GMT
Cache-Control: public, max-age=2592000
Server: gws
Content-Length: 219
X-XSS-Protection: 0
X-Frame-Options: SAMEORIGIN

<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```

We can split it to 4 sections

```
HTTP/1.1 301 Moved Permanently
```

`HTTP/1.1` - version
`301` - [status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status). I believe you are familiar with that
`Moved Permanently` - human-readable status code

```
Location: http://www.google.com/
Content-Type: text/html; charset=UTF-8
Date: Wed, 25 Mar 2020 18:53:12 GMT
Expires: Fri, 24 Apr 2020 18:53:12 GMT
Cache-Control: public, max-age=2592000
Server: gws
Content-Length: 219
X-XSS-Protection: 0
X-Frame-Options: SAMEORIGIN
```

Headers

```

```

Empty line, it means that the content will be sent in next section.

```
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```

Content, HTML or binary or something

```

```

Empty line, means end of request.

**REMEMBER: each new line is `\r\n`**

## Time for programming!

We know how request look like, we know how response look like, it's time to implement our server.

### What we expect

We want to get a very simple thing - to display an HTML page and a picture in a browser.
Let's prepare two HTMLs files and one picture

```
❯ pwd
/tmp/www

❯ ls
gallery.html index.html   me.jpg

❯ cat index.html
<html>
  <header>
    <title>My homepage!</title>
  </header>
  <body>
    <h1>Welcome!</h1>
    <p><a href="gallery.html">Here</a> you can look at my pictures</p>
  </body>
</html>

❯ cat gallery.html
<html>
  <head>
    <title>Gallery</title>
  </head>
  <body>
    <h1>My sexi photos<h1>
    <img src="me.jpg" />
  </body>
</html>
```

### The plan

Plan is very simple:

0. Open TCP socket and listen
1. Accept the client and read request
2. Parse the request
3. Find requested resource on disk
4. Send the response
5. Test

### Open TCP socket

In this article we will use `ServerSocket` class to handle TCP connection. As a homework you can reimplement the server to use the classes from the `nio` packages.

So, open your IDE and let's start.

```java
    public static void main( String[] args ) throws Exception {
        try (ServerSocket serverSocket = new ServerSocket(8080)) {
            while (true) {
                // implement client handler here
            }
        }
    }
```

I want to keep the code concise and clean - it's why I `throws Exception` instead of implementing good exception handling.
So as I told, we have to open socket on port 8080 (why not 80? Because to use low port you need root privileges).
We also need the infinity loop to 'pause the server'.

Use `telnet` to test the socket:

{{< image src="images/wireshark.png" alt="Wireshark" position="center" style="border-radius: 8px;" >}}

Perfect, works.

### Accept client connection

```java
        try (ServerSocket serverSocket = new ServerSocket(8080)) {
            while (true) {
                try (Socket client = serverSocket.accept()) {
                    handleClient(client);
                }
            }
        }
```        

To accept connection from client we have to call **blocking** `accept()` method. Java program will wait for a client on that line.

Time to implement the client handler:

```java
    private static void handleClient(Socket client) throws IOException {
        System.out.println("Debug: got new client " + client.toString());
        BufferedReader br = new BufferedReader(new InputStreamReader(client.getInputStream()));

        StringBuilder requestBuilder = new StringBuilder();
        String line;
        while (!(line = br.readLine()).isBlank()) {
            requestBuilder.append(line + "\r\n");
        }

        String request = requestBuilder.toString();
        System.out.println(request);
    }
```

We have to read the request. How? Just read the input stream from the client's socket. In Java it's not so simple, that's why I made this ugly line

```java
new BufferedReader(new InputStreamReader(client.getInputStream()));
```

Well, Java.

Request ends with one empty line (`\r\n`), remember? Client will send empty line, but imputStream will be still open, we have to read it until one, empty line arrives.

Run the server, go to `http://localhost:8080/` and observe logs:

{{< image src="images/logs.png" alt="Logs" position="center" style="border-radius: 8px;" >}}

It works! We can log the whole request!

### Parse the request

Parsing the request is realy simple, I don't think there's any need for further explanation

```java
        String request = requestBuilder.toString();
        String[] requestsLines = request.split("\r\n");
        String[] requestLine = requestsLines[0].split(" ");
        String method = requestLine[0];
        String path = requestLine[1];
        String version = requestLine[2];
        String host = requestsLines[1].split(" ")[1];

        List<String> headers = new ArrayList<>();
        for (int h = 2; h < requestsLines.length; h++) {
            String header = requestsLines[h];
            headers.add(header);
        }

        String accessLog = String.format("Client %s, method %s, path %s, version %s, host %s, headers %s",
                client.toString(), method, path, version, host, headers.toString());
        System.out.println(accessLog);
```

Just some splits. The only one thing that you may not understand is why we started the loop from 2? Because first line (index 0) is `GET / HTTP/1.1`, second line is host. The headers start with the third line of the request

### Send response

We will send the response to the client's output stream.

```java
        OutputStream clientOutput = client.getOutputStream();
        clientOutput.write("HTTP/1.1 200 OK\r\n".getBytes());
        clientOutput.write(("ContentType: text/html\r\n").getBytes());
        clientOutput.write("\r\n".getBytes());
        clientOutput.write("<b>It works!</b>".getBytes());
        clientOutput.write("\r\n\r\n".getBytes());
        clientOutput.flush();
        client.close();
```

Do you remember how response should look like?

```
version status code
headers
(empty line)
content
(empty line)
```

Don't forget to close the output stream.

{{< image src="images/it-works.png" alt="It works!" position="center" style="border-radius: 8px;" >}}

Wow, it's really works

### Find requested resource

We have to implement two methods first

```java
    private static String guessContentType(Path filePath) throws IOException {
        return Files.probeContentType(filePath);
    }

    private static Path getFilePath(String path) {
        if ("/".equals(path)) {
            path = "/index.html";
        }

        return Paths.get("/tmp/www", path);
    }
```

`guessContentType` - we have to tell to the browser what kind of content we are sending. It's callend `content type`. Fortunately, there are built-in mechanisms in Java for this. We don't have to make a big switch block.

`getFilePath` - Before we will return the file - we need to known it location.
This condition deserves attention

```java
        if ("/".equals(path)) {
            path = "/index.html";
        }
```

If user wants default resource then return `index.html`.

### Send the response

Do you remember the code that sends response to the user (block of clientOutput.write)? We need to move it to the method

    private static void sendResponse(Socket client, String status, String contentType, byte[] content) throws IOException {
        OutputStream clientOutput = client.getOutputStream();
        clientOutput.write(("HTTP/1.1 \r\n" + status).getBytes());
        clientOutput.write(("ContentType: " + contentType + "\r\n").getBytes());
        clientOutput.write("\r\n".getBytes());
        clientOutput.write(content);
        clientOutput.write("\r\n\r\n".getBytes());
        clientOutput.flush();
        client.close();
    }

Ok, finally we can return the file

        Path filePath = getFilePath(path);
        if (Files.exists(filePath)) {
            // file exist
            String contentType = guessContentType(filePath);
            sendResponse(client, "200 OK", contentType, Files.readAllBytes(filePath));
        } else {
            // 404
            byte[] notFoundContent = "<h1>Not found :(</h1>".getBytes();
            sendResponse(client, "404 Not Found", "text/html", notFoundContent);
        }

It works! 

{{< image src="images/it-works-2.png" alt="It works!" position="center" style="border-radius: 8px;" >}}

Finally, we can see html page served by our web server!

## Homework

0. Make it multi-thread.
   1. Create thread pool
   2. Move handleClient method to separated class and run it in new thread
1. Rewrite it using non-blocking IO
2. Implement POST method
   1. Start netcat
   2. Send some HTML form
   3. Analyze request

## Full source code

```java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.List;

// Read the full article https://mateuszjarzyna.github.io/posts/build-your-own-http-server-in-java-in-less-than-one-hour-only-get-method/
public class Server {

    public static void main( String[] args ) throws Exception {
        try (ServerSocket serverSocket = new ServerSocket(8080)) {
            while (true) {
                try (Socket client = serverSocket.accept()) {
                    handleClient(client);
                }
            }
        }
    }

    private static void handleClient(Socket client) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(client.getInputStream()));

        StringBuilder requestBuilder = new StringBuilder();
        String line;
        while (!(line = br.readLine()).isBlank()) {
            requestBuilder.append(line + "\r\n");
        }

        String request = requestBuilder.toString();
        String[] requestsLines = request.split("\r\n");
        String[] requestLine = requestsLines[0].split(" ");
        String method = requestLine[0];
        String path = requestLine[1];
        String version = requestLine[2];
        String host = requestsLines[1].split(" ")[1];

        List<String> headers = new ArrayList<>();
        for (int h = 2; h < requestsLines.length; h++) {
            String header = requestsLines[h];
            headers.add(header);
        }

        String accessLog = String.format("Client %s, method %s, path %s, version %s, host %s, headers %s",
                client.toString(), method, path, version, host, headers.toString());
        System.out.println(accessLog);


        Path filePath = getFilePath(path);
        if (Files.exists(filePath)) {
            // file exist
            String contentType = guessContentType(filePath);
            sendResponse(client, "200 OK", contentType, Files.readAllBytes(filePath));
        } else {
            // 404
            byte[] notFoundContent = "<h1>Not found :(</h1>".getBytes();
            sendResponse(client, "404 Not Found", "text/html", notFoundContent);
        }

    }

    private static void sendResponse(Socket client, String status, String contentType, byte[] content) throws IOException {
        OutputStream clientOutput = client.getOutputStream();
        clientOutput.write(("HTTP/1.1 \r\n" + status).getBytes());
        clientOutput.write(("ContentType: " + contentType + "\r\n").getBytes());
        clientOutput.write("\r\n".getBytes());
        clientOutput.write(content);
        clientOutput.write("\r\n\r\n".getBytes());
        clientOutput.flush();
        client.close();
    }

    private static Path getFilePath(String path) {
        if ("/".equals(path)) {
            path = "/index.html";
        }

        return Paths.get("/tmp/www", path);
    }

    private static String guessContentType(Path filePath) throws IOException {
        return Files.probeContentType(filePath);
    }

}
```
