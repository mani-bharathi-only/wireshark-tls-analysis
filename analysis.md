Introduction
For this project I decided to look into how encrypted network traffic actually works by capturing and analyzing real HTTPS packets using Wireshark. The whole point was to get a better understanding of what happens behind the scenes when you open a secure website, specifically how the Transport Layer Security protocol handles the connection setup. To generate the traffic I just opened a browser and visited a few secure websites while Wireshark was running in the background capturing everything.
Tools and Setup
The main tool I used was Wireshark along with a regular web browser running on Windows. The capture was saved in pcapng format and the network interface was WiFi. All the traffic I captured was HTTPS which is encrypted using TLS, so it was a good dataset to work with for this kind of analysis.
During the session I was just doing normal stuff like loading webpages and accessing secure sites to generate enough HTTPS requests for analysis.
Protocols I Noticed
While going through the capture I came across a few different protocols working together. DNS was handling domain name lookups, TCP was managing the underlying connection, and TLS along with HTTPS were doing the actual secure communication. It was interesting to see how all of these layer on top of each other just to load a single webpage.
Breaking Down the TLS Handshake
Step 1 – Client Hello
The whole handshake kicks off with the browser sending a Client Hello message to the server. Inside this packet you can see a list of TLS versions the client supports, the cipher suites it can work with, a randomly generated value, and some extensions. One of the most useful extensions here is Server Name Indication (SNI) which tells the server which domain the client is trying to reach. This matters when multiple websites share the same IP address.
Step 2 – Server Hello
After receiving the Client Hello the server fires back with its own Server Hello. This packet includes the TLS version the server decided to use, the cipher suite it picked from the client's list, and its own random value. Basically the server is saying "here is what we are going to use for this session."
Step 3 – Certificate Exchange
Next the server sends over its digital certificate. Looking at this in Wireshark you can see the domain name, who issued the certificate, and the public key. The client uses this to verify that it is actually talking to the real server and not some imposter. This step is pretty important for preventing man in the middle attacks.
Step 4 – Key Exchange
Once the certificate checks out the client and server go through a key exchange to come up with a shared secret that neither side directly sends over the network. The capture showed ECDHE being used here which is a pretty common choice these days because it supports forward secrecy. RSA is another option but it is becoming less common.
Step 5 – Everything Gets Encrypted
After the handshake wraps up all the actual communication switches to encrypted mode. In Wireshark this shows up as TLS Application Data and you cannot read any of the contents. That is the whole point. The encryption protects the confidentiality of what is being sent and also makes sure the data cannot be tampered with in transit.
Key Observations
One thing I noticed is that the TCP three-way handshake always happens first before any TLS packets appear. The TLS handshake only starts once TCP has established the connection. After the TLS handshake finishes everything becomes TLS Application Data so you can see the structure of the communication but not the actual content. It is also clear that HTTPS is really just HTTP running on top of TLS which is why the two are so closely connected.
Final Thoughts
Going through this capture gave me a much clearer picture of how secure web connections actually get established. The TLS handshake is more involved than I initially thought, especially when you look at how the key exchange works without ever directly transmitting the secret. Even though the encrypted data itself is unreadable, just analyzing the handshake reveals a lot about how modern web security is structured. It was a good hands on way to see something that usually just happens invisibly in the background.
