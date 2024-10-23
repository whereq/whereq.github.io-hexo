---
title: Understanding Root and Intermediate Certificates
date: 2024-10-23 09:35:02
categories:
- SSL
tags:
- SSL
---

## Understanding Root and Intermediate Certificates

### Index
- [Understanding Root and Intermediate Certificates](#understanding-root-and-intermediate-certificates)
  - [Index](#index)
  - [1. What are Certificates?](#1-what-are-certificates)
  - [2. Introduction to Certificate Authority (CA)](#2-introduction-to-certificate-authority-ca)
  - [3. Root Certificates](#3-root-certificates)
  - [4. Intermediate Certificates](#4-intermediate-certificates)
  - [5. Chain of Trust](#5-chain-of-trust)
- [|Fanyangxi – CC BY-SA 4.0|](#fanyangxi--cc-by-sa-40)
  - [6. Why are Intermediate Certificates Used?](#6-why-are-intermediate-certificates-used)
  - [7. Sample Certificate Chain](#7-sample-certificate-chain)
  - [8. Diagram: Certificate Chain](#8-diagram-certificate-chain)
  - [9. Python Code Example for Validating Certificates](#9-python-code-example-for-validating-certificates)
    - [Python Code to Validate a Certificate Chain](#python-code-to-validate-a-certificate-chain)
  - [10. Conclusion](#10-conclusion)

---

### 1. What are Certificates?
Certificates are digital documents that verify the identity of an entity (such as a website) and establish secure communication between servers and clients. A certificate ensures the authenticity of the entity claiming to be what it is.

---

### 2. Introduction to Certificate Authority (CA)
A **Certificate Authority (CA)** is a trusted entity responsible for issuing digital certificates. These certificates bind a public key with an entity's identity, like a domain name or a company. The CA plays a crucial role in the **Public Key Infrastructure (PKI)** system, which helps to secure online communications.

---

### 3. Root Certificates
A **Root Certificate** is the foundation of a trust chain. It is issued by a **Root Certificate Authority (Root CA)**, which is a trusted entity pre-installed in web browsers and operating systems. All certificates trust a Root CA, and this trust is propagated through the chain of certificates.

**Key Points:**
- Root Certificates are self-signed.
- They act as the ultimate trust anchor.
- Root CAs are only issued to very reputable organizations, and compromising a root certificate would cause massive security breaches.

---

### 4. Intermediate Certificates
An **Intermediate Certificate** is issued by a Root CA or another Intermediate CA. Intermediate certificates act as **middle layers** in the trust chain, allowing the Root CA to delegate the task of issuing end-user certificates.

**Key Points:**
- Intermediate Certificates are used to spread the trust of a Root Certificate.
- They ensure that if an Intermediate CA is compromised, the damage is limited to certificates issued by that Intermediate CA, not the root.
- Intermediate Certificates are usually signed by the Root CA or another trusted intermediate CA.

---

### 5. Chain of Trust
The **Chain of Trust** is the hierarchy of certificates starting from the **Root Certificate** at the bottom(root), followed by **Intermediate Certificates**, and finally the **End-Entity Certificate** (also known as the **Leaf Certificate**), which is the certificate for the actual entity (e.g., a website).

|![Chain of Trust](images/Understanding-Root-and-Intermediate-Certificates/Chain_of_trust_v2.svg)|
|-|
|[Fanyangxi – CC BY-SA 4.0](https://commons.wikimedia.org/wiki/File:Chain_of_trust_v2.svg)|
---

### 6. Why are Intermediate Certificates Used?
Intermediate certificates are used for several reasons:
1. **Security:** If a root certificate is compromised, all certificates under it are compromised. Intermediate certificates limit the scope of a breach.
2. **Delegation:** Root certificates delegate the issuance of certificates to intermediates, spreading out the load and responsibility.
3. **Certificate Revocation:** If an Intermediate CA is compromised, only certificates issued by that intermediate can be revoked without affecting the root.

---

### 7. Sample Certificate Chain
Let's assume we are visiting a secure website. The browser will validate the certificate using a chain similar to the following:

1. **Root Certificate:** Trusted by browsers.
2. **Intermediate Certificate:** Signed by the Root CA.
3. **Leaf Certificate:** Issued for the website and signed by the Intermediate CA.

---

### 8. Diagram: Certificate Chain

Here's a visual diagram to explain the trust chain:

```plaintext
                  +--------------------------+
                  |       Root CA            |
                  |    (Trusted by OS)       |
                  +--------------------------+
                           |
                           v
                  +--------------------------+
                  |   Intermediate CA 1       |
                  |   (Issued by Root CA)     |
                  +--------------------------+
                           |
                           v
                  +--------------------------+
                  |   Intermediate CA 2       |
                  |  (Issued by Int. CA 1)    |
                  +--------------------------+
                           |
                           v
                  +--------------------------+
                  |    End-Entity Cert        |
                  | (Issued for example.com)  |
                  +--------------------------+
```

---

### 9. Python Code Example for Validating Certificates

We can use the `ssl` and `cryptography` libraries in Python to validate and inspect the certificate chain. Below is an example that uses Python to verify the SSL certificate chain of a given website.

#### Python Code to Validate a Certificate Chain

```python
import ssl
import socket
import OpenSSL

def get_certificate_chain(hostname, port=443):
    """Fetch and print the certificate chain for a given hostname."""
    context = ssl.create_default_context()
    conn = context.wrap_socket(
        socket.socket(socket.AF_INET),
        server_hostname=hostname,
    )
    conn.connect((hostname, port))
    
    # Get the certificate chain
    cert_bin = conn.getpeercert(True)
    cert = ssl.DER_cert_to_PEM_cert(cert_bin)
    x509 = OpenSSL.crypto.load_certificate(OpenSSL.crypto.FILETYPE_PEM, cert)
    
    # Print certificates in the chain
    chain = conn.getpeercert(True)
    print(f"Certificate chain for {hostname}:\n")
    
    for (idx, cert) in enumerate(chain):
        print(f"Certificate {idx + 1}:\n")
        print(OpenSSL.crypto.dump_certificate(OpenSSL.crypto.FILETYPE_PEM, cert).decode())
    
    conn.close()

# Example: get certificate chain for a website
get_certificate_chain('www.example.com')
```

---

### 10. Conclusion
Root and Intermediate certificates are crucial to securing communications and establishing trust on the internet. By using a layered approach with Intermediate CAs, Root CAs can safely distribute trust across various entities while minimizing risk in case of a security breach.

