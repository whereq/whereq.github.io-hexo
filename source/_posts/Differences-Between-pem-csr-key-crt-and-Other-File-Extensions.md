---
title: Differences Between pem csr key crt and Other File Extensions
date: 2024-10-23 10:54:02
categories:
- SSL
tags:
- SSL
---

# Differences Between `.pem`, `.csr`, `.key`, `.crt`, and Other File Extensions

### Index
- [Differences Between `.pem`, `.csr`, `.key`, `.crt`, and Other File Extensions](#differences-between-pem-csr-key-crt-and-other-file-extensions)
    - [Index](#index)
    - [1. Introduction](#1-introduction)
    - [2. What is a `.pem` File?](#2-what-is-a-pem-file)
    - [3. What is a `.csr` File?](#3-what-is-a-csr-file)
    - [4. What is a `.key` File?](#4-what-is-a-key-file)
    - [5. What is a `.crt` File?](#5-what-is-a-crt-file)
    - [6. Other Common Certificate Formats](#6-other-common-certificate-formats)
    - [7. File Extensions and Corresponding Commands](#7-file-extensions-and-corresponding-commands)
      - [Generate a Private Key (`.key`):](#generate-a-private-key-key)
      - [Generate a CSR (`.csr`):](#generate-a-csr-csr)
      - [View CSR Content:](#view-csr-content)
      - [Generate a Self-Signed Certificate (`.crt`):](#generate-a-self-signed-certificate-crt)
      - [Convert a PEM to DER:](#convert-a-pem-to-der)
      - [Convert a PEM to PFX:](#convert-a-pem-to-pfx)
      - [View Certificate Content (`.crt`, `.pem`):](#view-certificate-content-crt-pem)
    - [8. Diagram: Certificate Workflow](#8-diagram-certificate-workflow)
    - [9. Conclusion](#9-conclusion)

---

### 1. Introduction
When dealing with SSL/TLS certificates, you often encounter various file extensions like `.pem`, `.csr`, `.key`, and `.crt`. Each serves a different purpose in the certificate creation, signing, and installation process. Understanding these files and how they interrelate is essential for setting up secure communications.

---

### 2. What is a `.pem` File?

A **PEM** (Privacy Enhanced Mail) file is a container format used for encoding certificates, keys, and other related data in Base64. It is often used to store the root, intermediate, and server certificates in SSL communication.

**Characteristics:**
- Encoded in Base64 (ASCII-readable).
- Enclosed between `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----`.

PEM can store:
- Certificate authority (CA) certificates.
- Server certificates.
- Private keys.

**Example Content of a `.pem` file:**

```
-----BEGIN CERTIFICATE-----
MIIDdzCCAl+gAwIBAgIE...
-----END CERTIFICATE-----
```

---

### 3. What is a `.csr` File?

A **CSR** (Certificate Signing Request) is a file generated when applying for an SSL certificate. It contains information about the domain and organization and is submitted to the Certificate Authority (CA) to request a signed certificate.

**Key Points:**
- It includes public key information and information about the organization.
- The CA uses it to generate a signed certificate.

**Example Content of a `.csr` file:**

```
-----BEGIN CERTIFICATE REQUEST-----
MIICzDCCAbQCAQAw...
-----END CERTIFICATE REQUEST-----
```

---

### 4. What is a `.key` File?

A **KEY** file contains the private key used to encrypt and decrypt information. Private keys must remain confidential and should never be shared. They are used to secure communication between clients and servers, and they are essential for creating CSRs and decrypting SSL traffic.

**Key Points:**
- It is used to sign the CSR and decrypt the certificate.
- Stored in PEM format but holds private key data.

**Example Content of a `.key` file:**

```
-----BEGIN PRIVATE KEY-----
MIIEvAIBADANBgkq...
-----END PRIVATE KEY-----
```

---

### 5. What is a `.crt` File?

A **CRT** (Certificate) file contains the SSL certificate issued by a CA. This certificate is used to secure communication between a client and a server. It contains the public key of the entity and is signed by the Certificate Authority.

**Key Points:**
- Often used interchangeably with `.pem`.
- It may be in PEM or DER format.
- Typically contains the public key of the certificate holder.

**Example Content of a `.crt` file:**

```
-----BEGIN CERTIFICATE-----
MIIC+DCCAeCgAwIBAgIJALfZc1...
-----END CERTIFICATE-----
```

---

### 6. Other Common Certificate Formats

**DER (.der):**  
- Binary version of PEM.
- Commonly used in Windows.
- Can store certificates and private keys.

**PFX/PKCS12 (.pfx or .p12):**  
- Binary format for storing the certificate, intermediate certificates, and private key in one file.
- Often used in Windows IIS servers.

---

### 7. File Extensions and Corresponding Commands

Here are some common OpenSSL commands for generating, converting, and inspecting certificate files:

#### Generate a Private Key (`.key`):
```bash
openssl genrsa -out private.key 2048
```

#### Generate a CSR (`.csr`):
```bash
openssl req -new -key private.key -out request.csr
```

#### View CSR Content:
```bash
openssl req -in request.csr -noout -text
```

#### Generate a Self-Signed Certificate (`.crt`):
```bash
openssl req -x509 -new -nodes -key private.key -sha256 -days 365 -out certificate.crt
```

#### Convert a PEM to DER:
```bash
openssl x509 -outform der -in certificate.pem -out certificate.der
```

#### Convert a PEM to PFX:
```bash
openssl pkcs12 -export -out certificate.pfx -inkey private.key -in certificate.crt -certfile ca.crt
```

#### View Certificate Content (`.crt`, `.pem`):
```bash
openssl x509 -in certificate.crt -noout -text
```

---

### 8. Diagram: Certificate Workflow

Here's a diagram to illustrate the typical workflow from generating a private key to receiving a certificate from a Certificate Authority:

```plaintext
+----------------------+        +--------------------+        +--------------------+
|   Private Key (.key)  | -----> |   CSR (.csr)       | -----> |   Certificate (.crt)|
+----------------------+        +--------------------+        +--------------------+
       |                                      ^
       |                                      |
       |           Certificate Signing        |
       +--------------------------------------+
```

1. **Generate Private Key:** The `.key` file is generated first, which is necessary for creating a CSR.
2. **Generate CSR:** A `.csr` is generated using the private key and sent to the CA.
3. **Receive Certificate:** The CA signs the CSR and returns a `.crt` or `.pem` certificate file.

---

### 9. Conclusion
Understanding the differences between certificate-related files like `.pem`, `.csr`, `.key`, and `.crt` is essential for managing SSL certificates. Each of these files has a unique role in the certificate lifecycle, from generating requests to securing communications. These files work together to ensure the integrity and security of encrypted data transmissions over the web.