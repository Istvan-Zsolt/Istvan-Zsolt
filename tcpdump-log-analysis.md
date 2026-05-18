# tcpdump Traffic Log Analysis

## Website Compromise and Suspicious Redirect Investigation


## Purpose of This Analysis

This document breaks down the most important entries from the `tcpdump` traffic log used in this investigation. The aim is to show how the traffic developed over time, what each key section means and why the later change in destination was treated as suspicious.

The log points to a browsing session that begins with a connection to:

yummyrecipesforme.com

and later shifts to:

greatrecipesforme.com

That change is important because it matches the incident scenario, where users were redirected after interacting with a suspicious file download prompt.


# 1. DNS Request for the Original Website

14:18:32.192571 IP your.machine.52444 > dns.google.domain: 35084+ A? yummyrecipesforme.com. (24)
14:18:32.204388 IP dns.google.domain > your.machine.52444: 35084 1/0/0 A 203.0.113.22 (40)

## What this means

The first line shows the user machine sending a DNS request to find the IP address for:

yummyrecipesforme.com

The second line shows the DNS server replying with an IP address:

203.0.113.22

In simple terms, the computer first had to ask, “Where is this website located on the network?” before it could connect to it.

## Why it matters

This confirms that the browsing session originally began with the expected website, rather than the suspicious redirected domain.


# 2. TCP Connection Established with the Original Website

14:18:36.786501 IP your.machine.36086 > yummyrecipesforme.com.http: Flags [S], seq 2873951608, win 65495, length 0
14:18:36.786517 IP yummyrecipesforme.com.http > your.machine.36086: Flags [S.], seq 3984334959, ack 2873951609, win 65483, length 0
14:18:36.786529 IP your.machine.36086 > yummyrecipesforme.com.http: Flags [.], ack 1, win 512, length 0

## What this means

These lines show the start of a TCP connection between the user machine and the website. This is commonly known as the **TCP three-way handshake**:

1. `Flags [S]` — the user machine sends a SYN packet to begin the connection.
2. `Flags [S.]` — the website responds with SYN-ACK, confirming it received the request.
3. `Flags [.]` — the user machine sends an ACK, completing the connection setup.

## Why it matters

This proves that the user machine successfully established a connection with the original website before any later redirection occurred.


# 3. HTTP Request to the Original Website

14:18:36.786589 IP your.machine.36086 > yummyrecipesforme.com.http: Flags [P.], seq 1:74, ack 1, win 512, length 73: HTTP: GET / HTTP/1.1


## What this means

This entry shows the browser sending an HTTP GET request to the website. The request:

HTTP: GET / HTTP/1.1

means the browser was asking for the main page of the site.

## Why it matters

This confirms that the session involved ordinary web traffic over **HTTP**. It also supports the wider incident story that the suspicious activity started during normal browsing of the recipe website.


# 4. DNS Request for a Different Domain

14:20:32.192571 IP your.machine.52444 > dns.google.domain: 21899+ A? greatrecipesforme.com. (24)
14:20:32.204388 IP dns.google.domain > your.machine.52444: 21899 1/0/0 A 192.0.2.17 (40)


## What this means

Roughly two minutes after the first website connection, the user machine sends another DNS request. This time, it asks for the IP address of:

greatrecipesforme.com


The DNS server returns:

192.0.2.17


## Why it matters

This is one of the most important points in the log. The browsing session started with `yummyrecipesforme.com`, but the system later begins resolving a different domain. In the context of the incident scenario, this supports the possibility of an unwanted or malicious redirection.

On its own, a second DNS request is not automatically malicious. Users visit multiple websites all the time. However, when it appears during a scenario involving a suspicious download and browser redirection, it becomes significant evidence.


# 5. TCP Connection to the Redirected Website

14:25:29.576493 IP your.machine.56378 > greatrecipesforme.com.http: Flags [S], seq 1020702883, win 65495, length 0
14:25:29.576510 IP greatrecipesforme.com.http > your.machine.56378: Flags [S.], seq 1993648018, ack 1020702884, win 65483, length 0
14:25:29.576524 IP your.machine.56378 > greatrecipesforme.com.http: Flags [.], ack 1, win 512, length 0


## What this means

These entries show a new TCP connection being set up, this time between the user machine and:

greatrecipesforme.com


The same SYN, SYN-ACK and ACK pattern appears again, showing that a valid TCP session was established.

## Why it matters

This confirms that the system did not simply look up the new domain. It actually connected to it. That makes the redirection behaviour more meaningful from an investigation perspective.


# 6. HTTP Request to the Redirected Website


14:25:29.576590 IP your.machine.56378 > greatrecipesforme.com.http: Flags [P.], seq 1:74, ack 1, win 512, length 73: HTTP: GET / HTTP/1.1


## What this means

The browser sends another HTTP GET request, this time to the second domain:


greatrecipesforme.com


## Why it matters

This shows that after the new domain was resolved, the browser actively requested content from it. In the scenario, this aligns with the reported redirection to a fake or malicious website.

---

# 7. Timeline of the Key Events

| Approximate time | Event                                       |
| ---------------- | ------------------------------------------- |
| 14:18:32         | DNS lookup for `yummyrecipesforme.com`      |
| 14:18:36         | TCP connection opened to original website   |
| 14:18:36         | HTTP GET request sent to original website   |
| 14:20:32         | DNS lookup for `greatrecipesforme.com`      |
| 14:25:29         | TCP connection opened to redirected website |
| 14:25:29         | HTTP GET request sent to redirected website |

---

# 8. Main Analytical Conclusion

The `tcpdump` log shows a clear sequence:

1. The machine resolves and connects to `yummyrecipesforme.com`.
2. HTTP traffic confirms normal web browsing activity.
3. The machine later resolves `greatrecipesforme.com`.
4. A new HTTP connection is established with that second domain.

When interpreted alongside the incident scenario, the change from the original domain to the second domain supports the conclusion that the user was redirected during a potentially malicious web session.

The traffic log alone does not prove every part of the compromise. For example, it does not by itself confirm the exact contents of the downloaded file or fully prove how the website administrator account was accessed. However, it provides technical evidence that strongly supports the reported redirection behaviour and helps build the incident timeline.

---

# 9. What This Shows in a Cybersecurity Context

This exercise demonstrates why traffic logs matter in incident investigations. Even a short log can help an analyst:

* Reconstruct the order of events
* Identify which domains were contacted
* Confirm that HTTP sessions were established
* Spot unusual changes in destination
* Separate what is directly proven by the log from what is inferred from the wider scenario

That distinction is important. Good analysis does not overstate the evidence; it explains what the data shows and where further investigation would still be needed.

