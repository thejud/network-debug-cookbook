# network-debug-cookbook

Some tools and techniques for tcpdump, wireshark and others.

I don't use these very often, and so I have to spend time looking things up repeatedly.

Table of Contents
=================

<!--ts-->
   * [network-debug-cookbook](#network-debug-cookbook)
   * [Table of Contents](#table-of-contents)
      * [Overview](#overview)
      * [tcpdump basics](#tcpdump-basics)
         * [Basic http request debugging](#basic-http-request-debugging)
      * [tshark (and wireshark)](#tshark-and-wireshark)
         * [look at http request matching a pattern](#look-at-http-request-matching-a-pattern)
      * [Other resources](#other-resources)

<!-- Added by: jdagnall, at: Sat Feb 29 19:46:40 PST 2020 -->

<!--te-->

Rebuild the TOC with:

    gh-md-toc --insert README.md


See https://github.com/ekalinin/github-markdown-toc

## tshark (and wireshark)

tshark is a terminal version of wireshark. It provides a higher level interface
to looking at network traffic than tcpdump, and is probably a good place to
start for most things.

Be sure you know what interfaces are being used. In particular, if you are
running in a kubernetes environment, be aware of any pod security layers that
may accept encrypted traffic on one interface and forward unencrypted traffic
to another.

### look at http request matching a pattern

I wanted to look at the raw HTTP requests, and tshark makes that easy.

This command limits to the http protocol, and ensures the request uri matches the (regex) subscribe. 
It produces brief output.

    # tshark  -Y http.request.uri~health
    Capturing on 'eth0'
        7 0.098643771 10.200.103.88 → 10.22.201.225 HTTP 196 GET /healthz/ready HTTP/1.1
       56 2.098765732 10.200.103.88 → 10.22.201.225 HTTP 196 GET /healthz/ready HTTP/1.1
       84 4.098699160 10.200.103.88 → 10.22.201.225 HTTP 196 GET /healthz/ready HTTP/1.1
      111 6.098781966 10.200.103.88 → 10.22.201.225 HTTP 196 GET /healthz/ready HTTP/1.1
      131 8.098690081 10.200.103.88 → 10.22.201.225 HTTP 196 GET /healthz/ready HTTP/1.1


Add the `-O http` to get the raw request:

    # tshark -O http -Y http.request.uri~health

Add an input filter to limit what gets captured by destination port:

    # Filter the input to match only destination port 8443
    tshark -O http -Y http.request.uri~health dsp port 8443

## tcpdump basics

By default, tcpdump will only pick one interface. Be sure that you know which interface(s) are being
used by your application. In a recent case, in Kubernetes with envoy for security, the public (eth0) interface
was encrypted, but the loopback interface was unsecured. So I needed to start tcpdump with the `-i lo` option, e.g.

    # bind to the loopback interface
    tcpdump -i lo
    

Write output to a file in pcap format. This can be consumed later.

    tcpdump -w /tmp/out.pcap -i eth0

### Basic http request debugging

    tcpdump -i lo dst port 8443  

## Other resources


