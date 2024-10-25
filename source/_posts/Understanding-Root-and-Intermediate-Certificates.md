---
title: Understanding Root and Intermediate Certificates
date: 2024-10-23 09:35:02
categories:
- SSL
tags:
- SSL
---

- [Understanding Root and Intermediate Certificates](#understanding-root-and-intermediate-certificates)
  - [1. What are Certificates?](#1-what-are-certificates)
  - [Key Points:](#key-points)
  - [2. Introduction to Certificate Authority (CA)](#2-introduction-to-certificate-authority-ca)
  - [Key Points:](#key-points-1)
  - [3. Root Certificates](#3-root-certificates)
  - [Key Points:](#key-points-2)
  - [4. Intermediate Certificates](#4-intermediate-certificates)
  - [Key Points:](#key-points-3)
  - [5. Chain of Trust](#5-chain-of-trust)
  - [Key Points:](#key-points-4)
  - [6. Why are Intermediate Certificates Used?](#6-why-are-intermediate-certificates-used)
  - [Key Points:](#key-points-5)
  - [7. Sample Certificate Chain](#7-sample-certificate-chain)
  - [Key Points:](#key-points-6)
  - [8. Diagram: Certificate Chain](#8-diagram-certificate-chain)
  - [Key Points:](#key-points-7)
  - [9. Python Code Example for Validating Certificates](#9-python-code-example-for-validating-certificates)
    - [Python Code to Validate a Certificate Chain](#python-code-to-validate-a-certificate-chain)
  - [Key Points:](#key-points-8)
  - [10. Conclusion](#10-conclusion)
  - [Key Takeaways:](#key-takeaways)

---

## Understanding Root and Intermediate Certificates

---

### 1. What are Certificates?
<a name="1-what-are-certificates"></a>

Certificates are digital documents that verify the identity of an entity (such as a website) and establish secure communication between servers and clients. A certificate ensures the authenticity of the entity claiming to be what it is.

### Key Points:
- **Digital Signature**: Certificates are digitally signed by a Certificate Authority (CA) to ensure their authenticity.
- **Public Key Infrastructure (PKI)**: Certificates are a fundamental part of the PKI, which is used to secure online communications.
- **Types of Certificates**: There are various types of certificates, including SSL/TLS certificates, code signing certificates, and email certificates.

---

### 2. Introduction to Certificate Authority (CA)
<a name="2-introduction-to-certificate-authority-ca"></a>

A **Certificate Authority (CA)** is a trusted entity responsible for issuing digital certificates. These certificates bind a public key with an entity's identity, like a domain name or a company. The CA plays a crucial role in the **Public Key Infrastructure (PKI)** system, which helps to secure online communications.

### Key Points:
- **Issuing Certificates**: CAs issue certificates after verifying the identity of the entity requesting the certificate.
- **Trust Model**: Browsers and operating systems come pre-installed with a list of trusted Root CAs.
- **Revocation**: CAs can revoke certificates if they are compromised or no longer valid.

---

### 3. Root Certificates
<a name="3-root-certificates"></a>

A **Root Certificate** is the foundation of a trust chain. It is issued by a **Root Certificate Authority (Root CA)**, which is a trusted entity pre-installed in web browsers and operating systems. All certificates trust a Root CA, and this trust is propagated through the chain of certificates.

### Key Points:
- **Self-Signed**: Root Certificates are self-signed.
- **Ultimate Trust Anchor**: They act as the ultimate trust anchor.
- **Reputation**: Root CAs are only issued to very reputable organizations, and compromising a root certificate would cause massive security breaches.

---

### 4. Intermediate Certificates
<a name="4-intermediate-certificates"></a>

An **Intermediate Certificate** is issued by a Root CA or another Intermediate CA. Intermediate certificates act as **middle layers** in the trust chain, allowing the Root CA to delegate the task of issuing end-user certificates.

### Key Points:
- **Delegation**: Intermediate Certificates are used to spread the trust of a Root Certificate.
- **Security**: They ensure that if an Intermediate CA is compromised, the damage is limited to certificates issued by that Intermediate CA, not the root.
- **Hierarchy**: Intermediate Certificates are usually signed by the Root CA or another trusted intermediate CA.

---

### 5. Chain of Trust
<a name="5-chain-of-trust"></a>

The **Chain of Trust** is the hierarchy of certificates starting from the **Root Certificate** at the bottom(root), followed by **Intermediate Certificates**, and finally the **End-Entity Certificate** (also known as the **Leaf Certificate**), which is the certificate for the actual entity (e.g., a website).

|![Chain of Trust](images/Understanding-Root-and-Intermediate-Certificates/Chain_of_trust_v2.svg)|
|-|
|[Fanyangxi – CC BY-SA 4.0](https://commons.wikimedia.org/wiki/File:Chain_of_trust_v2.svg)|

### Key Points:
- **Hierarchy**: The chain of trust ensures that each certificate in the chain is signed by the next certificate up the chain.
- **Validation**: When a client connects to a server, it validates the entire certificate chain up to the trusted Root CA.
- **Security**: The chain of trust ensures that only trusted certificates are used for secure communications.

---

### 6. Why are Intermediate Certificates Used?
<a name="6-why-are-intermediate-certificates-used"></a>

Intermediate certificates are used for several reasons:
1. **Security**: If a root certificate is compromised, all certificates under it are compromised. Intermediate certificates limit the scope of a breach.
2. **Delegation**: Root certificates delegate the issuance of certificates to intermediates, spreading out the load and responsibility.
3. **Certificate Revocation**: If an Intermediate CA is compromised, only certificates issued by that intermediate can be revoked without affecting the root.

### Key Points:
- **Scalability**: Intermediate CAs allow the Root CA to scale its operations by delegating certificate issuance.
- **Flexibility**: Intermediate CAs can be managed independently, allowing for more granular control over certificate issuance and revocation.
- **Risk Mitigation**: Intermediate CAs reduce the risk of a single point of failure by distributing trust across multiple entities.

---

### 7. Sample Certificate Chain
<a name="7-sample-certificate-chain"></a>

Let's assume we are visiting a secure website. The browser will validate the certificate using a chain similar to the following:

1. **Root Certificate**: Trusted by browsers.
2. **Intermediate Certificate**: Signed by the Root CA.
3. **Leaf Certificate**: Issued for the website and signed by the Intermediate CA.

### Key Points:
- **Validation Process**: The browser validates the entire chain, starting from the Leaf Certificate and moving up to the Root Certificate.
- **Trust Check**: If any certificate in the chain is not trusted, the connection will fail.
- **Chain Length**: The length of the chain can vary depending on the number of intermediate CAs involved.

---

### 8. Diagram: Certificate Chain
<a name="8-diagram-certificate-chain"></a>

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

### Key Points:
- **Hierarchy**: The diagram shows the hierarchical structure of the certificate chain.
- **Trust Propagation**: Trust is propagated from the Root CA down to the End-Entity Certificate.
- **Validation Path**: The browser follows this path to validate the certificate chain.

---

### 9. Python Code Example for Validating Certificates
<a name="9-python-code-example-for-validating-certificates"></a>

We can use the `ssl` and `cryptography` libraries in Python to validate and inspect the certificate chain. Below is an example that uses Python to verify the SSL certificate chain of a given website.

#### Python Code to Validate a Certificate Chain
<a name="python-code-to-validate-a-certificate-chain"></a>

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

### Key Points:
- **SSL Context**: The `ssl.create_default_context()` function creates a default SSL context with secure settings.
- **Socket Connection**: The `wrap_socket` method wraps a standard socket in an SSL socket, allowing secure communication.
- **Certificate Chain**: The `getpeercert` method retrieves the certificate chain from the server.
- **Print Certificates**: The certificates in the chain are printed in PEM format for inspection.

---

### 10. Conclusion
<a name="10-conclusion"></a>

Root and Intermediate certificates are crucial to securing communications and establishing trust on the internet. By using a layered approach with Intermediate CAs, Root CAs can safely distribute trust across various entities while minimizing risk in case of a security breach.

### Key Takeaways:
- **Trust Model**: The chain of trust ensures that only trusted certificates are used for secure communications.
- **Security**: Intermediate certificates limit the scope of a breach, ensuring that if an Intermediate CA is compromised, only certificates issued by that intermediate are affected.
- **Scalability**: Intermediate CAs allow the Root CA to scale its operations by delegating certificate issuance.

By mastering these concepts, you can better understand how certificates work and how they are used to secure online communications.
