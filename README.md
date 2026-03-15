# Wireshark TLS Encryption Analysis

This project analyzes TLS handshake packets captured using Wireshark.

## Topics Covered
- TCP 3-way handshake
- TLS Client Hello
- TLS Server Hello
- Certificate inspection
- Encrypted traffic analysis


## DNS Query

DNS resolves a domain name into an IP address.
![DNS Query](Screenshots/dns_query.png)

---

## TLS Client Hello

The client starts the TLS handshake by sending supported encryption methods.

![TLS Client Hello](Screenshots/tls_client_hello.png)

---

## TLS Server Hello

The server responds by selecting a cipher suite.

![TLS Server Hello](Screenshots/tls_server_hello.png)

---

## TLS Certificate

The server sends its digital certificate for authentication.

![TLS Certificate](Screenshots/tls_certificate.png)

---

## Encrypted TLS Traffic

After the handshake, all communication becomes encrypted.

![Encrypted TLS](Screenshots/tls_encrypted_data.png)
