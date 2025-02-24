# DNS reverse proxy #

[![Build Status](https://api.travis-ci.org/StalkR/dns-reverse-proxy.png?branch=master)](https://travis-ci.org/StalkR/dns-reverse-proxy) [![Godoc](https://godoc.org/github.com/StalkR/dns-reverse-proxy?status.png)](https://godoc.org/github.com/StalkR/dns-reverse-proxy)

A DNS reverse proxy to route queries to different DNS servers.
To illustrate, imagine an HTTP reverse proxy but for DNS.

It listens on both TCP/UDP IPv4/IPv6 on specified port.
Since the upstream servers will not see the real client IPs but the proxy,
you can specify a list of IPs allowed to transfer (AXFR/IXFR).

Example:

    $ go run dns_reverse_proxy.go -address :53 \
        -default 8.8.8.8:53 \
        -route .example.com.=8.8.4.4:53 \
        -route .example2.com.=8.8.4.4:53,1.1.1.1:53 \
        -route .example3.com.=https://dns.alidns.com \
        -allow-transfer 1.2.3.4,::1

A query for `example.net` or `example.com` will go to `8.8.8.8:53`, the default.
However, a query for `subdomain.example.com` will go to `8.8.4.4:53`. `.example3.com` will go to `https://dns.alidns.com` use `DNS over HTTPS`. `-default`
is optional - if it is not given then the server will return a failure for
queries for domains where a route has not been given.

It generate logs in [passivedns](https://github.com/gamelinux/passivedns) style:
```
1644906118.454894||10.0.1.27:52639||114.114.114.114:53||IN||gsp64-ssl.ls.apple.com.||CNAME||gsp64-ssl.ls-apple.com.akadns.net.||2431||1
1644906118.454932||10.0.1.27:52639||114.114.114.114:53||IN||gsp64-ssl.ls-apple.com.akadns.net.||A||17.142.171.5||69||1
1644906118.454942||10.0.1.27:52639||114.114.114.114:53||IN||gsp64-ssl.ls-apple.com.akadns.net.||A||17.142.171.6||69||1
1644906118.454948||10.0.1.27:52639||114.114.114.114:53||IN||gsp64-ssl.ls-apple.com.akadns.net.||A||17.142.171.7||69||1
```

# Setup #

Install go package, create Debian package, install:

    $ go get -u github.com/miekg/dns
    $ go get -u github.com/StalkR/dns-reverse-proxy
    $ cd $GOPATH/src/github.com/StalkR/dns-reverse-proxy
    $ fakeroot debian/rules clean binary
    $ sudo dpkg -i ../dns-reverse-proxy_1-1_amd64.deb

Configure in `/etc/default/dns-reverse-proxy` and start with `/etc/init.d/dns-reverse-proxy start`.

<!--
Alternatively with debuild:
  rm -f ../dns-reverse-proxy_*
Build unsigned:
  debuild --preserve-envvar PATH --preserve-envvar GOPATH -us -uc
Build with signed dsc and changes:
  debuild --preserve-envvar PATH --preserve-envvar GOPATH
Debuild asks for the orig tarball, you can proceed (y) or create it with:
  tar zcf ../dns-reverse-proxy_1.orig.tar.gz --exclude debian --exclude .git --exclude .gitignore .
-->

# License #

[Apache License, version 2.0](http://www.apache.org/licenses/LICENSE-2.0).

# Thanks #

- the powerful Go [dns](https://github.com/miekg/dns) library by [Miek Gieben](https://github.com/miekg)

# Bugs, feature requests, questions #

Create a [new issue](https://github.com/StalkR/dns-reverse-proxy/issues/new).
