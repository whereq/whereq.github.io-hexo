---
title: Best Practices of Using curl
date: 2024-11-03 11:26:54
categories:
- Best Practices
- Network
- CLI
tags:
- Best Practices
- Network
- CLI
---

- [Introduction](#introduction)
- [Basic Usage](#basic-usage)
  - [ Check if an URL is Alive](#-check-if-an-url-is-alive)
  - [ Download a File](#-download-a-file)
- [HTTP Methods](#http-methods)
  - [ HTTP GET](#-http-get)
  - [ HTTP POST with JSON Payload](#-http-post-with-json-payload)
- [Authentication](#authentication)
  - [ HTTP Basic Authentication](#-http-basic-authentication)
  - [ Bearer Token Authentication](#-bearer-token-authentication)
- [HTTPS and Certificates](#https-and-certificates)
  - [ HTTPS with Certificate](#-https-with-certificate)
  - [ Ignore HTTPS Certificate](#-ignore-https-certificate)
- [Advanced Scenarios](#advanced-scenarios)
  - [ Custom Headers](#-custom-headers)
  - [ Follow Redirects](#-follow-redirects)
  - [ Verbose Output](#-verbose-output)
  - [ Timeouts](#-timeouts)
- [Conclusion](#conclusion)
- [References](#references)

---

<a name="introduction"></a>
## Introduction

`curl` is a powerful command-line tool used for transferring data with URLs. It supports a wide range of protocols, including HTTP, HTTPS, FTP, and more. This article provides a comprehensive guide to the best practices of using `curl`, covering various scenarios such as checking if an URL is alive, downloading files, making HTTP GET/POST requests, handling authentication, and dealing with HTTPS certificates.

---

<a name="basic-usage"></a>
## Basic Usage

### <a name="check-if-an-url-is-alive"></a> Check if an URL is Alive

To check if an URL is alive, you can use the `-I` (or `--head`) option to send a HEAD request:

```bash
curl -I https://example.com
```

This command will return the HTTP headers without downloading the content, allowing you to quickly check if the URL is accessible.

### <a name="download-a-file"></a> Download a File

To download a file from a URL, simply use the `curl` command followed by the URL:

```bash
curl -O https://example.com/file.zip
```

The `-O` option saves the file with the same name as the remote file. If you want to save the file with a different name, use the `-o` option:

```bash
curl -o myfile.zip https://example.com/file.zip
```

---

<a name="http-methods"></a>
## HTTP Methods

### <a name="http-get"></a> HTTP GET

To make an HTTP GET request, use the `curl` command followed by the URL:

```bash
curl https://example.com/api/resource
```

### <a name="http-post-with-json-payload"></a> HTTP POST with JSON Payload

To make an HTTP POST request with a JSON payload, use the `-X POST` option and specify the JSON data using the `-d` option:

```bash
curl -X POST -H "Content-Type: application/json" -d '{"key1":"value1", "key2":"value2"}' https://example.com/api/resource
```

---

<a name="authentication"></a>
## Authentication

### <a name="http-basic-authentication"></a> HTTP Basic Authentication

To use HTTP Basic Authentication, use the `-u` option to specify the username and password:

```bash
curl -u username:password https://example.com/api/resource
```

### <a name="bearer-token-authentication"></a> Bearer Token Authentication

To use Bearer Token Authentication, use the `-H` option to set the `Authorization` header:

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" https://example.com/api/resource
```

---

<a name="https-and-certificates"></a>
## HTTPS and Certificates

### <a name="https-with-certificate"></a> HTTPS with Certificate

To use HTTPS with a client certificate, use the `--cert` and `--key` options:

```bash
curl --cert client.crt --key client.key https://example.com/api/resource
```

### <a name="ignore-https-certificate"></a> Ignore HTTPS Certificate

To ignore HTTPS certificate validation (not recommended for production), use the `-k` (or `--insecure`) option:

```bash
curl -k https://example.com/api/resource
```

---

<a name="advanced-scenarios"></a>
## Advanced Scenarios

### <a name="custom-headers"></a> Custom Headers

To send custom headers, use the `-H` option:

```bash
curl -H "X-Custom-Header: value" https://example.com/api/resource
```

### <a name="follow-redirects"></a> Follow Redirects

To follow HTTP redirects, use the `-L` (or `--location`) option:

```bash
curl -L https://example.com/redirect
```

### <a name="verbose-output"></a> Verbose Output

To get verbose output, use the `-v` (or `--verbose`) option:

```bash
curl -v https://example.com/api/resource
```

### <a name="timeouts"></a> Timeouts

To set a timeout for the request, use the `--connect-timeout` and `--max-time` options:

```bash
curl --connect-timeout 5 --max-time 10 https://example.com/api/resource
```

---

<a name="conclusion"></a>
## Conclusion

`curl` is a versatile and powerful tool for interacting with web services and downloading files from the command line. By mastering the various options and scenarios covered in this article, you can effectively use `curl` for a wide range of tasks, from simple URL checks to complex HTTP requests with authentication and custom headers.

---

## References

1. [curl Documentation](https://curl.se/docs/)
2. [curl Man Page](https://curl.se/docs/manpage.html)
3. [HTTP Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
4. [HTTP Authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)
5. [HTTPS](https://en.wikipedia.org/wiki/HTTPS)

By following these best practices, you can leverage `curl` to its full potential and efficiently manage your web interactions from the command line.