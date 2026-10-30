NetworkWalks B084 – Week 1: Cybersecurity Lab Setup

Lab setup and Wireshark packet and voice inspection.

This repository documents my Week 1 work for the NetworkWalks B084 cybersecurity course. It covers building a Kali Linux lab environment and using Wireshark to inspect captured network traffic, from plain-text web and FTP logins to a recorded VoIP call.

Note: All captures were made in a closed practice lab. Nothing here was run against real systems, and the techniques should only ever be used on networks you own or have written permission to test.

Lab environment
Item	Details
Operating system	Kali Linux (Offensive Security)
Main tool	Wireshark (capturing on the WiFi interface)
Supporting tools	Terminal, Firefox, Chrome

Kali Linux lab desktop

What I practised
Capturing live traffic and reading it with display filters
Following TCP streams to read application data
Spotting scans and brute-force attempts from traffic patterns
Recovering a call's codec and audio from RTP packets
Linking each weakness to a practical fix
Labs and findings
1. Port scanning
Pattern: one host (192.168.10.66) sent many connection attempts to different ports on 192.168.10.20 within a fraction of a second, all from the same source port.
Evidence: a long run of [RST, ACK] replies means the probed ports were closed.
Filter used: tcp.flags.reset==1 && tcp.flags.ack==1
Conclusion: a TCP SYN (half-open) port scan, which is reconnaissance.
2. Follow TCP Stream (hidden note in a web page)
Filter used: http.request.uri == "/staff.html", then Follow → TCP Stream (stream 7).
Finding: the server returned HTTP/1.1 200 OK, and the page contained a note hidden in an HTML comment. Browsers don't display comments, but anyone capturing the traffic can read them.
Lesson: never leave internal notes or credentials in page source.
3. HTTP plain-text credentials
Filter used: http.request.method == "POST"
Finding: two POST /login requests were visible in clear text.
Fix: use HTTPS so the sign-in is encrypted in transit.
4. FTP brute-force attack
Filters used: ftp.request.command == "USER", "PASS", and ftp.response.code == 530
Finding: 192.168.10.66 tried 25 passwords against the admin account on the FTP server at 192.168.10.20. 24 attempts returned 530 Login incorrect and the last one succeeded with 230.
Conclusion: a brute-force attack. FTP sends credentials in plain text.
Fixes: account lockout or rate limiting, strong passwords, and SFTP or FTPS instead of FTP.
5. VoIP call inspection (SIP and RTP)
SIP set up the call. The INVITE came from extension 101 (Ali Raza) to 202 (Sara), and the call lasted about 25 seconds (INVITE, 180 Ringing, 200 OK, ACK, BYE).
RTP carried the voice itself as a steady stream of small packets in both directions.
Codec: G.711 PCMU (payload type 0), 480-byte payloads.
Tools used: Telephony → VoIP Calls, then RTP Player to play the audio back.
Fix: encrypt calls with SRTP (and SIP over TLS) so captured voice can't be replayed.
RTP Player (two streams)	RTP packet details
RTP Player showing both directions of the call	RTP packet showing payload type PCMU

The RTP Player shows the two directions of the call (192.168.10.45:16384 ⇄ 192.168.10.52:18000) taking turns speaking.

Useful Wireshark filters
Goal	Filter
Closed-port replies	tcp.flags.reset==1 && tcp.flags.ack==1
Open-port replies	tcp.flags.syn==1 && tcp.flags.ack==1
Web form submissions	http.request.method == "POST"
FTP usernames and passwords	ftp.request.command == "USER" || ftp.request.command == "PASS"
Failed FTP logins	ftp.response.code == 530
Successful FTP login	ftp.response.code == 230
SIP call setup	sip.Method == "INVITE"
Voice packets	rtp
Everything in one TCP stream	tcp.stream eq 7
Key takeaways
Anything sent unencrypted (HTTP, FTP, plain SIP/RTP) can be read by anyone who can capture it.
Attacks leave patterns: many ports from one host is a scan, and many failed logins from one host is brute force.
Encryption (HTTPS, SFTP, SRTP) is the first fix for most of these weaknesses.
Repository contents
.
├── README.md
└── images/
    ├── kali-lab-desktop.png
    ├── rtp-player-streams.png
    └── rtp-packet-details.png
Author

cartermenlor-wq – Computer Science (Networking), building toward a career in network security and cybersecurity.
