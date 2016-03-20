---
layout: post
title: "ip2dec, dec2ip, ip2hex and hex2ip"
disqus: y
date: 2014-10-19 02:46:24 +0800
---

写程序时经常会因为偷懒在syslog里面直接打10进制或者16进制的ip地址，又或者是数据库中保存的不是ip字符串，当看这些数据时每次都需要手算一些。为了提高效率，写了四个脚本，放到PATH里面就可以执行了。

```sh
#!/bin/bash
ip2hex () {
	local a b c d ip=$@
		IFS=. read -r a b c d <<< "$ip"
		printf '0x%x\n' "$((a * 256 ** 3 + b * 256 ** 2 + c * 256 + d))"
}

ip2hex "$@"
```

```sh
#!/bin/bash
hex2ip () {
	local ip dec=$@
	desc=$(($desc))
		for e in {3..0}
	do
		((octet = dec / (256 ** e) ))
			((dec -= octet * 256 ** e))
			ip+=$delim$octet
			delim=.
			done
			printf '%s\n' "$ip"
}

hex2ip "$@"
```

```sh
#!/bin/bash
ip2dec () {
	local a b c d ip=$@
		IFS=. read -r a b c d <<< "$ip"
		printf '%d\n' "$((a * 256 ** 3 + b * 256 ** 2 + c * 256 + d))"
}

ip2dec "$@"
```

```sh
#!/bin/bash
dec2ip () {
	local ip dec=$@
		for e in {3..0}
	do
		((octet = dec / (256 ** e) ))
			((dec -= octet * 256 ** e))
			ip+=$delim$octet
			delim=.
			done
			printf '%s\n' "$ip"
}

dec2ip "$@"
```

测试

```sh
# ip2hex 1.2.3.4| xargs hex2ip | xargs ip2dec | xargs dec2ip
100.101.102.103
```
