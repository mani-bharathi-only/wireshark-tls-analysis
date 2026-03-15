# TLS Encrypted Traffic Analysis

## Overview

For this project I captured and analyzed HTTPS network traffic using Wireshark to get a better understanding of how TLS actually works in practice. The idea was pretty simple — open a browser, visit some secure websites, and capture the packets while everything's happening. From there I went through the capture file to look at how the TLS handshake plays out and what kind of information is actually visible even in encrypted traffic.

It was honestly more interesting than I expected because even though the actual data is encrypted, the handshake itself exposes a lot about how the connection gets set up.

---

## Tools Used

- Wireshark (for packet capture and analysis)
- Chrome browser (to generate HTTPS traffic)
- Windows 10

---

## Capture Details

The traffic was captured using Wireshark on the WiFi interface and saved as a `.pcapng` file. To generate traffic I just browsed around a few HTTPS sites — nothing special, mostly just loading pages to get a decent mix of packets.

**Traffic types captured:**
- HTTPS / TLS encrypted communication
- DNS lookups (domain resolution before connecting)
- TCP handshakes

---

## Protocols in the Capture

Looking through the capture, four main protocols showed up consistently:

- **DNS** — resolves domain names to IPs before the connection even starts
- **TCP** — the underlying transport layer, does the three-way handshake first
- **TLS** — handles the encryption negotiation
- **HTTPS** — the application layer running on top of TLS

It's worth noting that TLS doesn't just appear out of nowhere — TCP has to finish its own handshake before TLS can even start. That ordering is pretty visible in Wireshark if you filter by a specific IP.

---

## TLS Handshake Breakdown

This was the most interesting part to analyze. The TLS handshake is basically the client and server figuring out how they're going to encrypt their conversation before they actually say anything private.

### Client Hello

The handshake kicks off with the client sending a **Client Hello** to the server. Inside this packet you can see:

- The TLS versions the client supports
- A list of cipher suites (basically the encryption algorithms it knows how to use)
- A randomly generated value
- Extensions including **SNI (Server Name Indication)** — this is actually interesting because SNI reveals the domain name the client is trying to reach, even though everything after this is encrypted

### Server Hello

The server replies with its own **Server Hello**, which narrows things down:

- Which TLS version they'll actually use
- The specific cipher suite chosen from the client's list
- The server's random value

At this point both sides have agreed on the encryption parameters. The server picks whatever it supports from the client's list — usually the strongest option.

### Certificate Exchange

Next the server sends over its **digital certificate**. In Wireshark this shows up as the `Certificate` message. From the packet I could see:

- The domain name the cert is issued for
- Who issued it (the Certificate Authority)
- The public key

The certificate is how the client confirms it's talking to the real server and not some impostor. It's signed by a trusted CA so the browser can verify it.

### Key Exchange

After the certificate gets verified, the client and server do a **key exchange** to generate a shared session key. Most modern connections use **ECDHE (Elliptic Curve Diffie-Hellman Ephemeral)** for this — it's popular because it provides forward secrecy, meaning even if the private key gets compromised later, past sessions can't be decrypted.

### Finished / Encrypted Communication

Once the key exchange is done both sides send a `Finished` message to confirm the handshake went okay, and then everything switches over to encrypted application data. In Wireshark this just shows up as:
```
TLS Application Data
```

You can see the packets flying back and forth but the actual content is unreadable. That's the whole point.

---

## What I Noticed

A few things stood out while going through the capture:

- The TCP three-way handshake (SYN, SYN-ACK, ACK) always happens *before* TLS starts — it's easy to miss if you're just filtering for TLS
- SNI in the Client Hello leaks the domain name — so even with TLS, someone watching the traffic can see *which* site you're connecting to, just not what you're doing there
- After the handshake everything is Application Data — there's no way to read the actual HTTP requests or responses without the session keys
- TLS 1.3 handshakes are noticeably shorter than older versions — fewer round trips

---

## Conclusion

Going through this capture made the TLS handshake feel a lot more concrete than just reading about it. The process of negotiating cipher suites, exchanging certificates, and generating session keys is all right there in the packets. Even though the encrypted payload is unreadable, the handshake itself gives a lot of useful information about how the secure channel gets established.

The main takeaway for me was how layered the whole thing is — DNS, then TCP, then TLS, then finally HTTPS application data. Each layer depends on the one below it, and Wireshark makes it easy to see all of that happening in sequence.
