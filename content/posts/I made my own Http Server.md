+++
title = "I made my own Http Server"
date = "2025-04-07"
draft = false
cover = "/images/cover/http-server.png"
#tags = [""]
showFullContent = false
hideComments = true

#for SEO
#keywords = [""]
#description = ""
+++
[Github](https://github.com/asdiAdi/codecrafters-http-server-csharp)

Challenge attempt for creating my own [HTTP server](https://app.codecrafters.io/courses/http-server/overview).

<!--more-->

{{< details summary="TLDR" >}}
- Used C#.
- Implemented `GET`, `POST`, `gzip compression`, etc.
{{< /details >}}

In this guided project, I will attempt to build my own HTTP server that's capable of handling simple GET/POST requests, serving files and handling multiple concurrent connections.
This is an attempt in the challenge "[Build your own HTTP server](https://app.codecrafters.io/courses/http-server/overview)" from codecrafters.
The project is designed to be refactored each step of the way. I will paste the code that passes the tests at each stage, but I won't update the pasted code after refactoring.

## Base
### Repository Setup
I will be using C# for this attempt.

### Bind to a port
Create a TCP server that listens on port 4221.
```c#
TcpListener server = new TcpListener(IPAddress.Any, 4221);
server.Start();
server.AcceptSocket(); // wait for client
```

### Respond with 200
Respond to an HTTP request with a 200 response.
An HTTP response is made up of three parts, each separated by a [CRLF](https://developer.mozilla.org/en-US/docs/Glossary/CRLF) (\r\n).
```c#
Socket socket = server.AcceptSocket();
socket.Send(Encoding.UTF8.GetBytes("HTTP/1.1 200 OK\r\n\r\n"));
```

### Extract URL path
Extract the URL path from an HTTP request, and respond with either a `200` or `404`, depending on the path.
```c#
// read incoming data
var responseBuffer = new byte[1024];
socket.Receive(responseBuffer);

var lines = ASCIIEncoding.UTF8.GetString(responseBuffer).Split("\r\n");
string requestLine = lines[0];
string[] requestLineParts = requestLine.Split(" ");
string httpMethod = requestLineParts[0];
string requestTarget = requestLineParts[1];
string httpVersion = requestLineParts[2];

var notFoundResponse = $"{httpVersion} 404 Not Found\r\n\r\n";
var okResponse = $"{httpVersion} 200 OK\r\n\r\n";

var response = notFoundResponse;
if (httpMethod == "GET" && requestTarget == "/")
{
    response = okResponse;
}

socket.Send(Encoding.UTF8.GetBytes(response));
```

### Respond with body
Implement the `/echo/{str}` endpoint, which accepts a string and returns it in the response body.
```c#
else if (httpMethod == "GET" && requestTarget.StartsWith("/echo/"))
{
    string message = requestTarget.Substring(6);
    statusLine = $"{httpVersion} 200 OK\r\n";
    headers.Add("Content-Type: text/plain");
    headers.Add($"Content-Length: {message.Length}");
    headers.Add("\r\n"); // end of header
    responseBody = message;
}

var response = statusLine + string.Join("\r\n", headers) + responseBody;
```

### Read header
Implement the `/user-agent` endpoint, which reads the [`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/User-Agent) request header and returns it in the response body.
```c#
string getHeaderValue(List<string> headers, string header)
{
    foreach (string str in headers)
    {
        if (str.Contains(header))
        {
            return str.Substring(header.Length + 2);
        }
    }
    return "";
}
...
else if (httpMethod == "GET" && requestTarget.StartsWith("/user-agent"))
{
    string message = getHeaderValue(requestHeaders, "User-Agent");
    statusLine = $"{httpVersion} 200 OK\r\n";
    responseHeaders.Add("Content-Type: text/plain");
    responseHeaders.Add($"Content-Length: {message.Length}");
    responseHeaders.Add("\r\n"); // end of header
    responseBody = message;
}
```

### Concurrent connections
Add support for concurrent connections.
```c#
TcpListener server = new TcpListener(IPAddress.Any, 4221);
server.Start();

while (true)
{
    TcpClient client = server.AcceptTcpClient();
    Task.Run(() => HandleClient(client));
}

async Task HandleClient(TcpClient client)
{
    NetworkStream  stream = client.GetStream();
    var responseBuffer = new byte[1024];
    stream.Read(responseBuffer, 0, responseBuffer.Length);
...
```

### Return a file
Implement the `/files/{filename}` endpoint, which returns a requested file to the client.
```c#
else if (httpMethod == "GET" && requestTarget.StartsWith("/files/"))
{
    // check if file exists
    string fileName = requestTarget.Substring(7);
    string directory = Environment.GetCommandLineArgs()[2];
    string pathFile = $"{directory}/{fileName}";
    
    if (File.Exists(pathFile)) {
        string message = File.ReadAllText(pathFile);
        statusLine = $"{httpVersion} 200 OK\r\n";
        responseHeaders.Add("Content-Type: application/octet-stream");
        responseHeaders.Add($"Content-Length: {message.Length}");
        responseHeaders.Add("\r\n"); // end of header
        responseBody = message;
    }
}
```

### Read request body
add support for the `POST` method of the `/files/{filename}` endpoint, which accepts text from the client and creates a new file with that text.
```c#
else if (httpMethod == "POST" && requestTarget.StartsWith("/files/"))
{
    // check if file exists
    string fileName = requestTarget.Substring(7);
    string directory = Environment.GetCommandLineArgs()[2];
    string pathFile = $"{directory}/{fileName}";
    File.WriteAllText(pathFile, requestBody.TrimEnd('\0'));

    statusLine = $"{httpVersion} 201 Created\r\n\r\n";
}
```

## HTTP Compression
### Compression headers
Add support for [compression](https://en.wikipedia.org/wiki/HTTP_compression)
Add support for the `Accept-Encoding` and `Content-Encoding` headers.
```c#
string encoding = getHeaderValue(requestHeaders, "Accept-Encoding");
if (responseHeaders.Count > 0 && encoding != null)
{
    if (encoding.ToLower() == "gzip")
    {
        responseHeaders.Insert(0, "Content-Encoding: gzip");
    }
}
```

### Multiple compression schemes
Add support for `Accept-Encoding` headers that contain multiple compression schemes.
```c#
if (encoding.ToLower().Split(", ").Contains("gzip"))
{
    responseHeaders.Insert(0, "Content-Encoding: gzip");
}
```

### Gzip compression
Add support for `gzip` compression.
```c#
if (encoding.ToLower().Split(", ").Contains("gzip"))
{
    responseHeaders.Insert(0, "Content-Encoding: gzip");
    byte[] gzipData = compressMessage(responseBody);
    var index = responseHeaders.FindIndex(0, (h) => h.Contains("Content-Length"));
    if (index != -1)
    {
        responseHeaders[index] = $"Content-Length: {gzipData.Length}";
    }

    responseBody = gzipData;
}
```