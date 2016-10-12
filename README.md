# nginx-dynamic-tls-records-patch
Add TLS Dynamic Record Resizing to Nginx

From a424fefb0a638eb6d32756b5a0c471efc63e5384 Mon Sep 17 00:00:00 2001
From: Vlad Krasnov

Date: Sat, 9 Jan 2016 06:53:14 -0800

Subject: [PATCH] - Add TLS Dynamic Record Resizing



What we do now:

We use a static record size of 4K. This gives a good balance of latency and
throughput.



Optimize latency:

By initialy sending small (1 TCP segment) sized records, we are able to avoid
HoL blocking of the first byte. This means TTFB is sometime lower by a whole
RTT.



Optimizing throughput:

By sending increasingly larger records later in the connection, when HoL is not
a problem, we reduce the overhead of TLS record (29 bytes per record with
GCM/CHACHA-POLY).



Logic:

Start each connection with small records (1369 byte default, change with
ssl_dyn_rec_size_lo). After a given number of records (40, change with
ssl_dyn_rec_threshold) start sending larger records (4229, ssl_dyn_rec_size_hi).
Eventually after the same number of records, start sending the largest records
(ssl_buffer_size).

In case the connection idles for a given amount of time (1s,
ssl_dyn_rec_timeout), the process repeats itself (i.e. begin sending small
records again).



Upstream source:

https://github.com/cloudflare/sslconfig/blob/master/patches/nginx__dynamic_tls_records.patch
