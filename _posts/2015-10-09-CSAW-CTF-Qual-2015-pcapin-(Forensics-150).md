---
title: CSAW CTF Qualifications 2015 - pcapin (Forensics 150)
homedisplay: featimg
author: anselm
description: CSAW CTF Qualifications 2015 (Forensics 150) - pcapin
tags: [CTF, Forensics, CSAW, PCAP]
category: [CTF, Forensics, PCAP]
--- 
<h1>pcapin</h1>
We have extracted a pcap file from a network where attackers were present. We know they were using some kind of file transfer protocol on TCP port 7179. We're not sure what file or files were transferred and we need you to investigate. We do not believe any strong cryptography was employed.

Hint: The file you are looking for is a png <a href="/write-ups/resources/wu/csaw2015/pcapin_73c7fb6024b5e6eec22f5a7dcf2f5d82.pcap">pcapin_73c7fb6024b5e6eec22f5a7dcf2f5d82.pcap</a>
<hr/>

<p>
This challenge has surprisingly few solves. 41 out of 1000+ registered teams. We are given a pcap file with a few request/response from a client to a server and we have to figure out what is being sent.
</p>

<p>Analyzing the flow of the data being transferred, we can infer that two request are sent to the server and after each request, the server will send a series of packets as response.
</p>
<img src="/write-ups/img/res/wu/csaw2015/pcapin-tcpstream.png" width="30%" height="30%">

<h2>Request 1</h2>
<pre>
00:08:00:00:07:31:f9:e9
</pre>

<h2>Request 2</h2>
<pre>
00:3a:00:00:15:02:f9:e9:8f:95:88:9e:c7:89:87:9e:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9
</pre>
<p>
We see that request 1 is 8 bytes in length and coincidentally, there is also 00:08 in the data. Looking at the second request, we also see 00:3a in the same place. 3a is 58 in decimal and this correspond to the size of the request. Looking at the responses, we are also able to divide it into "packets" based on the size in the first 2 bytes. Another thing to note is that there appears to be an overwhelming amount of "f9:e9" in both the requests and in the first set of response from the server. This would infer that "f9:e9" is either the padding or the key since when XORed with null, it remains the same. We are able to verify that that is true by xoring with each of the packets in the first set of responses.
</p>

<pre>
document.pdf
sample.tif
outfile.dat
grey_no_firewall.zip
flag.png
resume.pdf
god.pcapng
malware.exe
</pre>
<h2>code</h2>
{% highlight python %}
for cipher in response1:
	key = request1.split(":")
	key = key[6:]
	
	cipher = cipher.split(":")
	cipher = cipher[13:]

	ans = ""
	for i,c in enumerate(cipher):
		xor = (hexor(key[i % len(key)],c))
		ans += xor
	print ans[6:-4].decode("hex")
{% endhighlight %}

<p>
In the output, we noticed that there was a file named "flag.png". We can also see that the last 50 bytes of request 2 is identical to the section of cipher text that gets translated into "flag.png".
</p>
<img src="/write-ups/img/res/wu/csaw2015/pcapin-flagcipher.png" width="30%" height="30%">

<P>
Thinking that we solved it, we processed all packets in response 2 the same way as how we solve response 1 but it resulted in gibberish data. Since we know that we need a png file, we then tried to "guess" the key by taking the data segment of the first packet in the response and xoring the first 8 bytes with the png header. Getting a series of "3f:50" back, we used it as a key but it again resulted in gibberish data. Undaunted, we again look deeper into the responses and noticed that they all have the same size. However, it is unlikely that the original data can be divided that nicely so it would probably be padded at the end. Looking at the last packet of response 2, we can see a series of "6c:4c" but that is also not the correct key. We then look for differences between packets in response 1 versus those in response 2 and noticed something unusual. In response 1, byte 3,4 are both "00:00" in all instances but in response 2, they are all different. 
</p>
<img src="/write-ups/img/res/wu/csaw2015/pcapin-byte34.png" width="30%" height="30%">
<p>
Looking deeper, we realize that f9+45+1 = 13f -> 3f and e9+67=150 -> 50. Again verifying that with the last packet, f9+72+1=16c -> 6c and e9+63=14c -> 4c. Finally, using this new information, we found the flag!.
</p>
<img src="/write-ups/img/res/wu/csaw2015/pcapin-flag.png">
<p>A s1mp!3_n37w0rk_c4@113nge that was not simple at all!</p>


<h2>code</h2>
{% highlight python %}
for cipher in response2:
    cipher = cipher.split(":")

    key = ["f9","e9"]
    k1 = int(key[0],16)
    k2 = int(key[1],16)

    a1 = int(cipher[2],16)
    a2 = int(cipher[3],16)

    k1 = k1+a1+1
    k2 = k2+a2

    key = [hex(k2)[-2:],hex(k1)[-2:]]

    cipher = cipher[12:]

    ans = ""
    for i,c in enumerate(cipher):
        xor = (hexor(key[i % len(key)],c))
        ans += xor

    ans2 += ans

import binascii
hb=binascii.a2b_hex(ans2)
file = open("flag.png","wb")
file.write(hb)
file.close()
{% endhighlight %}

<h2>Raw Data</h2>
<pre>request1 = "00:08:00:00:07:31:f9:e9"
response1 = ["00:44:00:00:07:32:00:01:00:00:00:00:00:25:f2:a9:8d:96:8a:8c:84:9c:87:8d:c7:89:8d:9f:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:60:00",
"00:44:00:00:07:32:00:01:00:00:00:00:00:00:28:a9:9a:98:84:89:85:9c:c7:8d:80:9f:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:60:00",
"00:44:00:00:07:32:00:01:00:00:00:00:00:00:15:c1:86:8c:9d:9f:80:95:8c:d7:8d:98:9d:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:60:00",
"00:44:00:00:07:32:00:01:00:00:00:00:00:05:36:0a:8e:8b:8c:80:b6:97:86:a6:8f:90:9b:9c:9e:98:85:95:c7:83:80:89:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:60:00",
"00:44:00:00:07:32:00:01:00:00:00:00:00:00:15:c1:8f:95:88:9e:c7:89:87:9e:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:60:00",
"00:44:00:00:07:32:00:01:00:00:00:00:00:02:21:d9:9b:9c:9a:8c:84:9c:c7:89:8d:9f:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:60:00",
"00:44:00:00:07:32:00:01:00:00:00:00:00:00:6f:00:8e:96:8d:d7:99:9a:88:89:87:9e:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:60:00",
"00:44:00:00:07:32:00:01:00:00:00:00:00:00:7a:00:84:98:85:8e:88:8b:8c:d7:8c:81:8c:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:60:00"]

request2 = "00:3a:00:00:15:02:f9:e9:8f:95:88:9e:c7:89:87:9e:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9:e9:f9"
response2 = ["00:d4:45:67:07:32:00:1c:00:00:00:00:d9:6f:1e:78:5d:35:4a:35:50:3f:50:32:19:77:14:6d:50:3f:51:78:50:3f:50:28:58:39:50:3f:50:a7:e0:b2:78:3f:50:3f:56:5d:1b:78:14:3f:af:3f:af:3f:af:9f:ed:98:c3:3f:50:2a:26:76:14:7e:04:47:cc:d2:cd:48:08:6b:87:89:90:40:63:cb:51:79:0a:2b:4b:2d:33:ef:40:ce:f9:7e:ff:9d:32:1e:4a:34:16:9c:96:2d:1b:b3:19:77:14:90:76:5d:28:7b:d9:1a:01:cb:4a:5d:d9:22:73:09:7c:67:ff:5d:47:b6:75:9f:72:fe:30:28:75:1d:70:ed:6b:7c:83:a6:a7:38:63:d8:5e:05:98:3f:d3:da:0d:41:8f:08:8f:d8:cc:06:ab:a3:e5:08:2b:92:e3:c9:0a:d4:3c:7a:da:97:78:3a:e5:9f:e4:93:dc:2a:6b:48:e2:5f:b3:79:a2:35:5b:86:46:23:1c:e4:e7:e1:fa:f2:75:d4:d4:d4:21:4e:68:b2",
"00:d4:23:c6:07:32:00:1c:00:01:00:00:b5:18:2f:3f:85:f7:16:fa:51:03:ba:d8:75:a0:d4:94:11:60:34:b9:1b:29:af:87:77:c5:7f:ee:12:ea:37:2c:d9:31:e1:b0:f5:c8:16:a1:fa:4c:bc:04:ea:d3:61:47:f0:a2:05:2c:05:13:74:93:b1:78:6c:9c:ac:41:20:96:0c:b9:1b:89:f9:d0:34:2e:15:42:90:a3:b2:26:b9:7e:ec:5e:32:c0:e8:c8:10:4a:da:13:f9:db:49:8d:bf:23:34:22:f0:e5:51:a1:1e:32:82:36:4c:36:90:22:99:9a:2b:1d:cf:33:3e:bd:00:b2:e0:6b:f1:03:af:6b:19:ab:33:e2:42:2a:43:71:c2:82:c6:ba:02:83:61:43:1c:9b:42:da:21:14:70:02:93:67:ab:49:92:b3:00:9e:71:75:e2:7c:4e:68:52:c7:1b:22:d7:eb:83:43:64:8b:be:8a:3b:b9:84:8c:f1:14:59:98:2a:ac:88:29:bb:8c:e7:99:da:9e:a9:89:39:88:de:22",
"00:d4:98:69:07:32:00:1c:00:02:00:00:6b:0b:a9:5b:9b:fe:6d:e4:de:d1:19:05:80:4d:97:77:e7:01:e3:d7:b1:54:9d:2f:34:db:3b:bb:7b:0b:cb:a8:c1:63:db:ce:9c:da:4d:8d:a4:15:d7:93:82:52:9e:9e:d2:39:25:7d:20:67:bc:cf:24:0d:68:57:bd:79:85:f1:33:fc:fc:21:29:2c:3c:f6:b6:76:62:e1:ab:e0:ff:5c:07:ba:46:1e:a9:73:55:64:d6:14:80:4a:94:14:67:ac:6c:ee:ba:78:d8:ca:7e:74:a0:5f:c9:1e:a1:67:77:a4:73:13:43:4c:8c:ce:88:20:57:03:1a:96:06:cd:1d:85:28:e8:e0:42:81:c1:aa:3c:bf:d1:a7:e7:82:09:b8:b2:fc:db:f1:4a:d6:96:8c:65:a6:f6:fa:05:45:e3:1b:db:ee:61:b4:09:e8:04:b9:37:60:2e:1d:8d:9c:94:56:c2:46:8c:1c:e8:3a:ba:eb:f5:9d:e0:35:6d:2c:28:e5:fd:1d:4b:b1:55:1e:cc:ee:6a",
"00:d4:48:73:07:32:00:1c:00:03:00:00:39:c9:04:61:45:a1:1f:00:c2:b9:a5:20:b4:12:5c:df:11:a2:43:94:f2:27:23:1a:44:14:0e:6b:7d:89:ca:d3:2f:a4:50:bb:bb:8c:2d:82:83:5d:17:eb:c8:fd:3e:21:ad:3e:59:84:ae:27:6e:21:b5:10:6e:31:2f:ab:82:fc:21:d7:bb:ec:85:f1:db:7f:fd:e3:04:75:34:82:e5:4f:47:5a:86:e9:4b:20:2d:fb:56:35:2e:36:b8:8a:d6:57:44:5b:46:30:a5:a4:11:cc:da:c5:e3:32:61:3f:41:b6:fa:78:88:8a:64:7c:96:8a:bc:36:38:66:ba:50:55:bc:83:3e:1f:8e:92:df:e6:d4:b7:e7:8e:99:85:5b:8b:cc:41:73:0e:19:ae:7b:84:9b:4d:72:27:74:5c:b3:95:8b:9e:10:bf:27:fd:29:45:49:38:70:0a:ad:85:41:dc:95:e4:33:e9:d4:73:7b:61:df:f9:99:eb:41:ec:af:fb:dd:2c:ad:8d:41:5c:f3:04:8e",
"00:d4:dc:51:07:32:00:1c:00:04:00:00:ea:88:87:0e:0e:a1:14:d6:01:99:a6:36:ec:2b:c1:63:84:09:d1:1e:d3:1e:72:60:27:eb:70:b5:21:cd:2c:9b:a7:2c:e6:a5:89:25:c8:4e:87:b0:37:d6:d1:b1:f5:70:6f:25:dc:19:a6:05:90:af:49:50:cd:3f:39:16:e4:05:9d:89:82:48:84:d8:40:85:27:12:3a:21:a9:45:a3:6a:52:c7:97:50:34:13:fe:13:3f:75:94:8b:53:0e:8d:f9:e7:f0:77:34:4d:db:cb:59:a4:cb:01:e4:41:04:1e:b4:35:ca:fa:bd:e6:ee:36:22:91:88:a7:5d:a6:4b:53:e0:4a:f6:b5:49:06:37:89:2f:c9:e7:4d:a3:9f:71:bd:d1:a3:6f:83:32:2f:c0:08:0a:e1:cd:93:7d:11:6b:06:ea:72:98:75:a9:9c:7b:f2:0f:a3:3d:4b:a7:e6:af:ca:56:ab:e8:04:0e:82:6f:5b:00:99:d1:b9:70:75:a9:5b:1d:c0:b0:b7:cc:33:29:ec:70",
"00:d4:5c:ff:07:32:00:1c:00:05:00:00:27:8c:48:88:5f:92:3c:02:2e:ac:12:34:0b:b0:6e:e7:63:5d:26:b5:2f:25:0c:d9:d7:08:60:da:03:a1:07:71:d3:79:67:a9:99:26:08:ab:26:cb:e9:0e:f0:4e:60:9e:31:0f:c3:b9:26:37:30:66:ac:98:26:ca:63:d8:ee:36:3f:9f:7b:c2:7c:c0:3a:9f:39:c7:69:8b:52:6b:9b:a8:78:c8:d5:2f:43:0f:db:56:6a:19:75:fc:18:cd:12:ce:7c:6b:96:9a:4a:1b:73:2e:13:f5:67:66:9e:27:49:bb:20:c7:fd:b4:89:8c:b8:02:ba:14:a7:59:e7:12:26:98:5c:26:9f:81:41:d4:7e:c2:7e:a4:03:f8:b5:22:15:ba:db:7a:92:a3:2a:8e:2d:d3:8e:55:ef:64:8c:90:e4:98:b7:ef:ba:3f:53:b4:3a:04:56:08:5c:5e:68:09:57:ad:46:9c:38:94:42:8b:53:9f:c5:7e:48:a5:53:b9:e2:50:18:3f:fc:5c:ed:49:9c:bb",
"00:d4:94:4a:07:32:00:1c:00:06:00:00:39:4e:ca:a5:64:e4:97:29:46:cf:66:fd:81:b8:09:fe:1c:bf:a2:87:40:68:e3:76:b0:81:e3:65:ef:d7:8b:18:c9:69:49:d2:6f:0b:05:97:0a:b7:4b:a3:68:c8:98:2f:70:bf:45:ff:f2:48:fe:03:b2:5d:95:ff:07:b2:8f:4c:4a:74:62:b9:5c:5c:d4:65:9c:47:a4:47:b7:d6:32:ce:55:e0:1d:67:ea:57:f7:ab:16:5f:68:cd:ff:ef:b2:d4:7f:2c:09:a2:ef:36:a2:e3:74:00:53:e6:53:4e:8a:66:5b:58:dc:55:74:e2:71:8c:34:41:af:4f:e3:4e:b3:3d:e2:5f:cb:20:6e:c5:d3:12:0a:41:87:33:4f:6d:35:71:cf:77:54:84:b8:04:83:e0:e3:0e:d1:1c:21:00:b5:09:20:fd:c4:a0:4c:63:eb:0f:be:0b:76:03:57:5c:25:1b:d1:55:04:e7:71:ed:28:95:ae:d0:c8:64:ee:35:7f:75:ca:e9:01:fd:40:3f:5d:5e",
"00:d4:58:ec:07:32:00:1c:00:07:00:00:55:e0:12:dd:2c:a1:03:7f:eb:ce:e4:d1:0a:85:78:91:60:31:12:68:46:e3:61:fe:59:37:ee:24:d5:47:68:14:40:a3:9f:1e:98:ff:27:48:da:c7:d4:2a:40:15:76:28:7d:5c:3c:87:3e:cb:a6:37:10:97:99:0f:67:16:ad:fa:88:ea:af:07:36:67:26:58:de:bb:3c:b3:14:d7:7e:05:44:4a:ce:71:c3:db:6d:a3:22:8d:59:a9:34:55:f1:94:13:2e:01:e9:a2:c7:a7:6f:ab:a0:51:c3:eb:6c:49:d9:5b:94:03:88:4f:81:be:85:83:4b:ae:87:43:15:cc:4b:d9:ca:e3:df:4d:8a:8d:52:19:76:c7:f6:73:f4:e9:1e:9e:b1:b4:08:cf:4f:0f:e6:74:15:52:5c:cf:89:fb:aa:38:65:3b:77:f3:a0:74:f9:fe:71:09:8e:4b:03:b5:27:c1:58:d0:49:da:4c:d1:b2:1e:dd:eb:50:b5:99:34:91:19:5d:dd:72:e0:79:5e:5c",
"00:d4:1f:29:07:32:00:1c:00:08:00:00:bf:42:31:0b:9b:69:fd:c8:91:e7:58:fa:0a:86:8e:55:5b:70:7b:94:e7:4d:85:71:29:3e:b9:b2:11:28:a3:a8:aa:67:eb:3c:6b:9c:97:19:4a:50:b7:7d:3b:0a:41:b3:fd:ff:b4:bf:0a:03:0a:11:d5:b7:d5:dc:e3:e7:f6:d0:b6:7f:77:18:72:7a:73:58:74:77:3c:de:30:3b:2a:0f:03:d8:be:50:81:9d:6a:b2:6a:df:a0:7c:f6:d2:76:95:f8:ce:9d:8d:01:3e:5a:16:1f:3c:2f:2d:86:fb:9a:11:bc:f5:ca:38:6a:08:a8:b9:52:3f:d1:26:3a:91:f2:dc:99:c0:e4:ca:5d:19:fe:17:1f:3c:fa:f1:43:57:bd:42:55:89:e0:41:fa:bc:59:03:c9:56:eb:ce:ad:a9:a7:af:f4:cb:84:34:76:85:2c:d4:9f:24:69:a9:a7:af:f4:78:48:83:a2:9d:b9:12:06:13:17:b8:83:88:5e:47:a6:11:2f:a8:8a:0c:21:7e:18:d8",
"00:d4:7c:cd:07:32:00:1c:00:09:00:00:d8:2b:87:9f:71:69:27:ec:2c:64:0f:03:9d:6f:11:39:25:0a:4a:4e:5b:a8:c8:6d:0f:2a:38:09:e6:26:2b:bc:ae:02:5a:6e:b9:a5:64:1e:6c:c6:97:d5:4d:83:a5:f8:1d:fa:d5:34:fc:bc:c5:ad:d1:91:53:57:9d:58:70:b2:7e:3e:2e:8a:81:e2:7c:a8:58:df:ba:93:03:0d:41:06:a8:49:28:3b:31:78:bf:87:04:bc:20:a3:a5:90:7a:57:4c:80:db:a0:4e:8e:26:01:58:6a:8f:11:78:7e:b0:07:d7:16:d6:e3:4b:24:9e:62:ca:40:c9:48:31:b9:29:b1:70:b4:34:a6:03:1d:83:58:f1:1d:74:65:0f:ea:39:10:78:52:90:40:20:10:bb:6d:c4:0e:4f:04:c4:b9:2a:57:5b:a2:9f:12:57:97:4a:8d:21:29:c6:08:51:6b:3b:1b:d9:8a:4b:41:fd:c0:d8:b3:ea:54:d7:c3:39:39:23:81:4c:5d:80:e0:b1:f9:a8:17",
"00:d4:58:ba:07:32:00:1c:00:0a:00:00:cf:3a:6b:fa:dd:af:fb:ef:d8:65:b0:b5:6e:71:82:77:a6:2f:9e:6f:61:31:c1:aa:40:fd:1c:3a:76:ee:9a:9f:36:0b:49:75:d1:6b:ba:6b:9a:68:4a:39:ee:fb:4d:ce:0f:fc:ad:2a:8d:0c:ef:0c:c2:73:4e:4f:bf:da:98:2a:33:9e:13:62:41:11:e1:ba:47:ba:ab:d2:10:f1:80:a3:82:73:9b:8a:7a:53:46:79:2b:f3:fd:0c:07:34:c6:63:c6:b6:eb:e0:61:90:eb:5d:ae:f7:43:ae:da:c4:ef:c9:25:cc:fb:de:fc:32:83:33:34:7d:a0:72:5d:a1:75:7f:a3:f4:2c:4c:ee:11:88:79:42:94:a5:a8:59:e6:d4:22:63:ec:ea:41:3e:5f:f5:3c:e2:53:4e:4f:bf:4a:7d:f5:2c:f2:db:63:12:e3:2f:bb:7c:cd:3d:4f:98:80:f1:2b:0f:e0:2f:3b:e0:79:88:ac:7b:ea:32:9c:4c:ec:a8:12:48:44:8e:ca:96:3b:b1:02",
"00:d4:d7:ab:07:32:00:1c:00:0b:00:00:8a:b2:45:f4:26:33:f6:3d:93:de:52:af:64:b1:b0:0c:2e:00:43:02:07:13:36:f3:1a:7e:ce:14:13:7f:3a:eb:29:c6:d4:92:df:9a:5a:2e:62:4a:64:56:f9:b5:f1:54:63:55:9d:d1:c8:6b:ea:6c:5a:b5:c0:f9:80:2d:28:a4:bf:d1:6b:cd:a1:7b:56:63:d3:42:73:f9:bf:ff:32:76:13:d6:4b:ff:c9:fb:88:aa:3c:dd:17:65:f8:0b:c0:49:68:38:de:14:f8:b5:f9:fc:08:66:e2:3e:ca:eb:db:49:54:7c:6f:26:e5:b8:43:5f:79:da:82:d1:04:42:0b:7e:c5:67:50:05:c0:5f:b8:8e:5a:35:85:f2:44:c2:1f:e0:a0:e1:74:56:bb:6f:94:f1:6e:27:f9:db:1f:5b:0a:0a:23:38:6a:2f:90:cc:a9:5b:11:68:ad:f6:c3:7e:72:0c:02:fc:21:c9:85:3c:9c:ef:ed:43:df:06:3b:b2:ba:40:e4:f7:b4:51:d5:0c:2f:da",
"00:d4:41:f2:07:32:00:1c:00:0c:00:00:1d:e5:1f:f3:53:1a:e6:40:d1:4a:8d:4e:cf:31:de:64:f6:67:93:e2:38:fc:23:76:46:71:68:bd:d6:90:67:ec:32:f3:93:3b:e1:8d:b2:78:b2:62:c2:e4:74:61:de:7b:13:c9:3e:5f:5d:be:ba:15:4a:3b:2b:b5:66:06:bd:1d:fd:79:60:f9:39:d9:81:cc:68:81:0f:5d:95:6d:9c:3c:e9:49:a9:b3:8b:95:e3:ed:16:a1:9e:f0:7d:76:5a:c9:51:b7:ee:f1:dc:f5:12:b0:cc:f2:4c:f2:5f:0d:40:35:c6:19:f5:12:d2:cc:c5:06:83:de:36:b6:7e:6f:d1:bb:1f:e3:43:d4:fd:77:1b:c8:38:b4:5a:c9:a2:31:fb:6d:0e:78:26:29:cf:6f:3a:79:ae:be:b1:e2:7d:94:7c:1c:c7:a0:76:0f:ab:91:b8:5e:54:24:b0:57:60:fa:0c:2c:80:0e:be:38:ab:d0:0f:32:f3:b4:e6:6f:7e:17:0e:7a:f5:ac:28:db:0a:18:c5:07",
"00:d4:1e:fb:07:32:00:1c:00:0d:00:00:9e:5c:95:71:cd:18:bd:61:9d:60:c8:40:24:91:ef:0f:90:6e:fb:0d:ff:ff:28:b9:0d:4b:6e:fa:34:ba:e1:18:7d:8d:9c:18:5e:88:15:d8:0b:a7:97:23:da:06:d7:91:60:b7:62:15:4f:e8:7f:8d:b0:52:e7:2b:d7:aa:17:ea:cc:48:2a:95:b7:8f:ca:69:ca:22:7e:ad:9f:ef:ee:d3:7d:3c:41:e9:b0:01:b7:30:b3:0a:cc:2f:f0:21:1d:e1:68:e8:12:de:97:e9:86:52:2e:d2:1c:e6:57:d7:dc:a3:85:1b:6a:d2:08:b1:48:0a:c1:46:0a:fd:61:c3:b7:33:f3:8d:03:bf:b4:30:7a:43:92:2c:db:38:64:8d:25:d9:7c:51:c0:04:1f:ed:b3:22:50:76:49:cd:9c:70:2f:96:07:df:e5:e8:fc:2e:48:5a:5b:b3:c6:f8:24:19:26:7b:86:f0:06:fc:60:ff:6c:09:be:ad:6d:4a:de:28:1b:70:3f:8e:d9:b9:45:fc:2f:7c",
"00:d4:a9:e3:07:32:00:1c:00:0e:00:00:38:4f:14:32:cd:8d:e2:47:db:b5:5e:8c:5f:02:63:04:0b:78:ba:d5:d9:57:26:c6:9c:38:f5:fa:d1:be:9c:54:46:3c:62:1e:98:6c:06:04:66:6e:5d:de:06:31:7e:ea:d3:dd:64:52:32:20:59:65:97:c6:b8:66:e7:c5:02:e7:e0:b5:7f:8c:e0:af:77:e2:4f:5b:16:6c:43:73:04:eb:52:6b:29:6c:21:eb:81:72:78:0f:a5:c0:a3:0c:99:f8:81:52:c6:76:fd:26:8e:82:08:b2:b8:22:55:e7:4e:81:e6:a9:89:f7:d8:ca:06:d9:70:32:b1:58:be:9c:f5:3a:cd:70:6a:d2:9a:ba:8b:72:d9:39:36:c5:ae:c7:c8:f3:25:7c:8e:b4:fe:5d:a7:68:da:a3:f2:bc:fe:e7:af:f1:2c:ca:23:52:2c:7a:7f:e3:b5:81:ee:47:10:9a:2c:9c:52:00:76:32:9c:6a:d2:d2:39:89:d1:65:42:c6:93:fb:d4:2e:60:0f:f4:12:a6:b6",
"00:d4:e1:46:07:32:00:1c:00:0f:00:00:51:88:89:7b:80:7c:08:f3:a0:1f:f7:83:8c:b7:b5:39:5a:3d:3d:52:3f:21:86:b7:7a:68:05:e3:b7:a2:2e:da:b7:c3:36:aa:57:22:5d:61:17:e2:7a:ea:25:0e:08:31:c9:96:2f:21:5a:32:8d:ae:b4:0f:83:f7:53:8d:81:1f:6f:84:b0:d6:91:65:ad:7c:76:ce:9e:d4:31:db:df:e4:21:d5:33:54:a7:db:3f:67:fd:00:de:2a:6f:22:f7:c0:c7:30:24:b8:25:4e:a0:b0:72:89:b4:e2:76:c6:32:b3:43:b8:ac:45:5d:27:7b:d4:20:ce:05:f4:fc:13:ff:6b:ad:6a:7a:1e:a9:7c:01:82:0d:a3:98:31:b0:52:dc:3d:2e:4b:89:6f:0c:39:51:86:95:cb:91:aa:0c:86:f4:6e:0c:fc:10:44:da:20:d8:08:f8:08:bc:cd:c1:35:83:05:92:c0:aa:79:79:f4:ff:0f:16:3d:89:7d:ed:64:30:e4:56:89:08:2c:57:18:fd:49",
"00:d4:00:7c:07:32:00:1c:00:10:00:00:3b:15:d8:bd:95:58:20:b6:7a:c9:63:7f:27:7b:b2:48:00:8f:17:55:ff:a8:76:e3:a6:d4:3a:dc:97:3c:68:0e:a1:98:db:e3:56:bc:86:c3:8f:8b:22:43:39:34:f4:c5:9b:ea:7b:9c:1e:b8:26:fb:a8:64:c6:5f:b7:2a:1b:17:82:bd:19:88:57:15:12:14:29:36:cb:a7:91:07:a2:c5:e1:89:31:69:35:c7:f9:59:68:cc:73:ec:bd:a2:3d:f2:af:0b:51:50:f0:7d:2d:de:a7:4e:eb:f6:21:9c:0b:d4:35:c7:bc:5d:9e:01:f6:69:fa:75:92:7e:6c:2f:1f:79:22:0f:95:4b:0b:2a:e5:62:de:8d:e4:88:26:f3:15:bd:0c:c2:78:6f:44:f9:b0:62:ef:be:47:24:cd:19:61:9f:30:2e:c0:24:51:ca:56:99:29:05:9b:fa:19:43:05:fb:82:58:c6:d3:f5:33:9d:19:ca:45:9d:30:aa:f5:e5:9b:d8:81:6e:3c:70:30:a6:80",
"00:d4:62:c2:07:32:00:1c:00:11:00:00:fb:02:cd:fa:59:32:7e:63:01:76:b2:e5:79:ba:42:5f:df:2d:d9:7e:89:6c:3b:67:a4:42:1b:b7:6f:55:29:d2:b6:bf:f5:3e:89:0f:3c:70:4a:2a:d7:60:80:e1:16:29:79:fd:11:1c:96:02:ce:fd:d7:5f:08:72:52:f8:d4:23:fd:b0:77:95:1e:27:5c:b6:57:02:9e:19:f0:45:fe:02:48:2c:9c:eb:e1:e1:9b:29:64:ad:49:a9:40:f8:ce:39:9a:2f:77:64:9d:d9:2f:2c:49:9e:ae:ae:ce:6e:26:c2:c8:37:d0:27:39:8e:79:8c:7c:8f:c8:a2:fc:03:4a:6f:da:be:98:c2:39:16:95:01:14:b6:5c:b1:73:ed:63:49:a1:6a:2c:d8:4b:a2:66:6b:f7:86:19:19:cc:45:c1:49:b9:3f:c8:ae:a1:57:62:0b:2d:4a:1f:7d:53:b8:e2:5c:3d:32:74:92:59:fb:c1:52:4e:36:35:cd:32:06:07:ee:4b:a0:d2:ae:3f:16:c6:75",
"00:d4:08:54:07:32:00:1c:00:12:00:00:a8:08:f6:c6:bb:94:ab:c2:42:3e:fa:d4:f7:e7:da:3f:58:07:7d:f1:7b:8f:17:d6:26:bc:51:e8:7f:6d:69:8b:ab:b9:34:0b:c9:f6:cd:aa:cd:d9:e6:74:4b:2e:ce:f0:97:72:f1:d6:f9:86:e1:80:3f:20:8a:6c:28:e0:a7:97:6f:5a:65:b3:35:be:48:89:2b:ce:c8:f2:1d:74:c2:7c:9c:2a:48:cf:e3:bf:29:97:a9:d6:8b:1d:48:c4:a8:39:4a:82:cf:25:c5:f1:a3:e0:50:dd:47:49:4d:f5:56:81:17:ac:54:a7:31:e4:83:88:55:21:de:97:06:75:05:7b:cc:20:cd:fe:9f:6d:68:a2:06:33:08:97:7e:c8:18:f7:ed:5c:80:1a:e7:a9:2a:c7:f7:a7:88:a8:b1:45:e0:3a:c3:a1:50:59:3d:9e:67:b7:5f:d4:99:4b:27:95:ad:a8:c7:b7:03:c8:85:b3:13:5b:ca:ff:ca:76:4c:70:1f:37:16:89:bc:d1:9a:55:a7:f6",
"00:d4:27:f8:07:32:00:1c:00:13:00:00:48:08:4b:50:58:8d:bd:7f:b6:a6:43:b3:f3:2b:6a:ab:cb:5d:35:54:ad:dc:d9:61:48:f3:f2:17:76:69:c9:08:ca:22:1f:c2:3c:65:3e:9f:ec:b5:c6:0e:e1:80:57:73:7a:16:8f:7f:c5:54:40:16:0a:d6:0e:26:21:4e:2b:35:3f:ce:3d:38:e6:1a:da:9b:57:4a:26:6e:72:06:f2:94:8c:ba:d1:07:cb:75:12:d2:5b:b7:e7:7b:02:9b:a5:05:f3:28:78:3a:58:7d:cf:99:0b:8e:c3:20:2b:20:93:ce:3f:dc:64:fd:ae:74:61:8d:cb:97:9c:34:30:67:27:e4:7a:16:e2:c4:ab:85:2b:df:8a:63:3c:52:dd:55:0f:3d:0e:97:8d:e8:3a:57:97:ad:91:52:e2:eb:4a:25:73:f2:32:f0:f2:aa:ca:59:7e:8b:73:81:ca:56:0f:51:ab:62:88:77:39:c4:2a:a5:26:e5:81:4d:89:e9:26:ce:5e:7e:5c:2f:8b:a0:70:80:c0:26",
"00:d4:23:1b:07:32:00:1c:00:14:00:00:f8:e0:6d:cb:b4:3c:33:e2:fa:86:e5:c3:da:88:22:e7:6e:9f:2e:cb:bc:6b:eb:c3:4e:0e:4e:52:a7:67:3c:76:fe:41:0f:13:d2:65:1a:3d:68:d3:55:41:5e:37:d0:ef:da:50:4c:fd:70:79:20:bc:ce:51:f2:be:88:11:b6:ee:f6:99:7e:d5:35:02:78:bd:9f:93:ee:15:59:f6:49:5f:4e:97:f4:4d:fa:73:c6:19:4a:b0:5d:de:99:e0:ff:94:0c:11:c0:ea:f7:d2:95:b7:81:c6:50:b5:46:27:2f:9e:87:a8:9e:16:e6:dd:47:9a:bc:00:1b:42:c5:b0:cb:8a:cd:45:bc:6c:27:4d:fa:59:6e:b3:52:fc:aa:3f:23:22:bb:df:43:75:5f:45:ac:ec:bc:33:bc:65:e9:07:a1:d7:23:7a:85:51:82:d2:ce:88:e8:20:79:07:67:5e:47:a1:3b:e0:f0:a4:69:62:5c:a6:12:b4:af:36:77:cb:a9:b6:f6:23:cd:f4:ed:d2:c0:47",
"00:d4:e9:e8:07:32:00:1c:00:15:00:00:be:3e:6e:2c:86:9d:af:8f:ed:9b:c1:d0:58:e7:0e:2c:ee:4c:a8:a4:a5:a3:bc:87:5d:a8:9b:81:4e:51:25:8a:37:1b:20:2c:6c:6c:1b:db:4f:6a:5b:01:07:1e:2a:a1:4c:82:16:d5:bc:8b:08:57:f8:24:f2:c1:99:ab:98:42:68:4e:bc:66:e3:c3:76:b5:7c:e7:d2:d9:07:34:46:40:30:02:53:5f:16:c0:f3:fb:ec:88:c7:e3:e2:25:5c:30:a8:a2:6c:e9:8a:88:ba:1d:2c:28:fe:7b:c9:fa:c0:95:28:d1:8e:c9:9a:64:a5:22:05:b2:72:d3:46:ab:69:74:49:ab:6e:ca:82:1b:82:ce:f8:b3:83:b9:1b:02:22:04:7a:1e:a7:b0:b4:7b:f8:a8:a4:51:22:11:a1:b6:f7:4a:ef:7e:ea:74:d0:30:77:d2:af:03:91:47:77:45:29:e4:d4:82:36:0d:1f:20:65:d7:ab:95:3d:2b:f1:46:62:13:f3:16:25:a6:3b:6e:6b:92",
"00:d4:cd:e7:07:32:00:1c:00:16:00:00:eb:3c:37:58:8a:72:21:3b:28:a4:9c:8a:9c:7f:a2:20:de:44:76:88:c7:d5:e4:ed:22:a2:e2:05:b3:a5:28:3c:31:84:d0:33:ef:38:21:80:70:3b:4d:90:fb:6e:c4:fa:61:5f:5c:5b:cc:59:18:22:c8:de:ca:91:4a:92:63:6b:34:2a:d4:d0:b5:9e:5e:cd:8d:a9:ad:32:35:45:d5:8b:8e:73:58:f0:fc:ea:19:0b:1d:62:64:6b:dc:c4:ad:ba:e6:0a:4b:e0:14:9b:fa:a4:15:09:4d:23:1b:a3:fc:18:61:44:cf:78:2e:3d:69:20:e6:da:e0:c7:f3:84:93:dd:48:5e:b1:6f:7f:88:9e:86:51:97:1a:15:34:4a:e7:1f:35:20:77:74:b2:25:4a:95:cb:de:47:ab:0d:0d:c3:7e:4c:70:3d:2b:c8:6b:71:d1:9f:5a:36:4a:ca:a6:b8:a7:10:3a:34:a3:d0:41:79:7a:6a:f1:0c:0a:5d:58:67:a9:75:6f:74:43:de:aa:0a:fb",
"00:d4:43:8d:07:32:00:1c:00:17:00:00:a5:0b:16:cb:1a:d3:50:1b:64:2e:6d:f6:d6:d4:a5:0c:43:0c:37:62:39:72:7e:b1:79:d6:a5:9a:bc:82:33:50:0f:d2:eb:4a:7e:a1:4d:aa:47:8e:10:8c:93:cd:17:e3:1a:ef:f2:84:39:f8:d1:57:34:ce:30:b0:ae:8c:06:1c:d5:83:89:a3:86:a5:6e:a1:b1:b2:31:57:1c:b7:ff:ac:67:54:af:e4:5e:29:7c:db:0c:45:7e:7b:dc:0b:ae:e4:ac:af:e0:a8:33:61:24:2f:bd:81:ca:75:b9:f3:50:d7:a0:10:aa:07:01:5b:d3:8a:41:80:eb:a0:8f:e2:e3:16:5f:13:5f:5c:9a:bd:77:91:84:cb:60:d7:3b:56:d4:9a:80:be:71:56:1a:ce:b7:e9:df:28:68:bd:87:f7:1b:ee:22:81:3e:80:27:80:1b:ce:5e:1e:f5:83:25:9b:a6:c9:fb:0a:a6:6e:e0:0c:6f:24:6e:cd:10:ee:d0:b1:9c:e6:e1:cc:f5:3b:7f:d8:3b:4b",
"00:d4:0f:76:07:32:00:1c:00:18:00:00:e9:bf:33:90:60:96:ee:b7:e1:95:e5:7d:d6:ff:c0:75:dd:dd:8b:1d:f6:a0:76:0c:1a:4c:9d:4a:3d:ae:c0:16:72:b2:2a:4a:a0:fa:7e:4a:db:03:a3:0b:c6:05:e6:4b:de:74:cc:2f:ab:7f:29:ef:84:46:61:18:ed:69:f0:0b:32:d6:25:82:db:9d:4b:db:ec:ba:36:6d:32:44:30:6e:38:b5:78:45:8f:63:c0:b1:20:87:41:c4:d5:94:64:90:69:73:2b:9c:b8:bf:31:d8:dd:8e:b6:e0:3b:ef:bb:a9:f8:ae:d8:9c:0b:83:14:b2:29:b1:a8:e1:9e:cd:9e:8a:da:ab:84:9e:16:44:3a:45:96:c5:3b:c2:be:ca:5f:75:a4:c0:78:5c:c9:9a:d7:cc:3d:e5:44:3e:79:3f:7e:88:49:84:44:5e:f7:1e:43:a8:6b:87:df:cf:05:36:14:a0:cb:82:c4:3e:ac:42:0f:19:9e:2e:10:7b:70:38:4e:98:e1:37:82:d2:e8:ee:d4:a8",
"00:d4:25:5a:07:32:00:1c:00:19:00:00:00:e6:97:c2:fe:b9:9e:b0:55:bc:b8:ea:a0:c5:fe:64:bb:1a:45:2d:6c:3f:c3:89:8e:85:72:61:a3:df:19:40:94:a2:04:10:ed:1a:44:ac:37:e4:35:51:1e:a3:0b:7d:29:35:cc:80:7f:de:fd:6e:20:a5:7b:26:72:5f:04:a4:58:a2:20:70:0c:cb:8e:84:07:43:fe:95:a4:d8:5c:6c:65:3f:e3:dd:ac:0c:44:10:25:7d:66:19:ee:39:17:e9:ad:e6:90:64:5d:23:8e:94:97:04:f2:57:07:80:0d:82:9b:61:af:07:49:9a:01:c7:7c:4d:06:73:01:1d:64:31:1f:ff:81:aa:28:dd:85:08:13:a1:30:8f:90:b1:1e:33:4e:15:61:33:69:95:c4:76:2a:bb:a4:fb:23:76:2d:51:c4:c9:ee:6e:35:6b:e7:a2:f1:0d:03:4b:98:d2:2d:40:47:b0:a8:39:e5:ea:97:bc:48:a9:5f:21:8f:db:3a:ec:f3:2f:74:38:b2:d7:52:7e",
"00:d4:f9:2e:07:32:00:1c:00:1a:00:00:8e:0e:c8:3f:cb:06:fc:0a:66:30:11:c9:2e:c9:05:4f:6f:c2:78:d9:f4:2e:1d:76:95:5f:ab:cf:15:fc:09:37:ec:24:48:f2:5f:ca:6e:01:8e:17:5b:66:03:e5:49:5c:c6:65:72:06:c3:a0:d8:78:74:43:bc:d8:b1:d5:31:17:f3:14:a4:b3:8e:f8:ef:9c:70:06:f9:2e:17:0f:e5:1e:a0:71:76:f7:93:3e:64:d5:1a:ea:35:ef:3c:5d:76:46:5a:4e:66:5f:b0:6d:40:ef:3c:5a:83:ea:74:34:15:16:e6:11:4e:58:40:f8:e6:c2:ae:af:59:af:45:e1:69:72:96:0f:ec:3f:8e:64:3f:16:9c:b6:82:0b:ce:8b:e7:d3:98:c1:25:d9:eb:85:20:d2:56:bb:a9:ca:a1:9f:4e:10:0d:24:ed:96:62:de:be:df:6f:ae:02:bc:10:40:fc:8a:e6:4f:59:8c:ac:d4:65:ca:d4:4c:ec:bd:6f:8e:44:bc:3b:e5:05:60:bd:3f:7a:33",
"00:d4:72:63:07:32:00:1c:00:1b:00:00:a0:b5:8e:fa:2a:93:93:9d:93:da:c1:83:1a:c0:5c:d2:17:f4:d7:0f:2a:0e:0e:d2:00:4a:68:90:56:35:15:1d:30:b9:66:4d:ca:34:61:06:b7:13:84:b8:1f:15:bc:d6:d2:3b:d3:bb:a5:03:eb:4b:5a:1f:8c:b3:d3:a1:6d:4d:20:66:45:8d:0a:30:50:f5:f5:d5:78:de:fe:ce:17:97:ba:10:a6:82:e2:df:48:bc:a7:ac:80:9d:af:05:93:d6:79:77:42:70:6c:9e:8a:61:1e:5e:7e:a4:63:40:08:06:2a:2a:9b:1a:a1:74:98:d1:77:f2:6f:2a:44:57:a3:b8:48:fd:2e:b3:f2:c6:7b:aa:e7:cb:d2:16:a6:95:23:4e:7b:5b:a9:93:4c:da:ff:f3:25:f5:c9:5b:1e:6c:4c:6c:4c:25:09:22:08:c2:0e:0c:ce:6c:4c:6c:4c:6c:4c:6c:4c:6c:4c:6c:4c:6c:4c:6c:4c:6c:4c:6c:4c:6c:4c:6c:4c:6c:4c:6c:4c:6c:4c:6c"]
</pre>