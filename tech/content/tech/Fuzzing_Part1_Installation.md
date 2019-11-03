+++
date = "2016-12-09T19:48:31-05:00"
title = "Welcome"
draft = true

+++

# Fuzzing (Part 1): Installation

This will be part 1 in a series on fuzzing, or automated vulnerability/penetration testing. Fuzzing has been shown to be a powerful tool for redteamers, and although it is still in its infancy, it is very clear that it will be the future of penetration testing. Popular tools such as Burp suite include a basic fuzzer for common vulnerabilities, and are often minimally employed in current testing implementations.

I'll be focusing on the most popular fuzzer currently in use, [American Fuzzy Lop](http://lcamtuf.coredump.cx/afl/) (AFL), and 2 semi-popular but largely outdated fuzzers/scanners, being [skipfish](https://code.google.com/archive/p/skipfish/) and [w3af](http://w3af.org).

#### Installing AFL

AFL was by far the easiest to install, as it has a lot of online community support, and AFL is still updated relatively often. The author is actually on twitter which is cool, and in general his website is a great read. He's done several projects, and also wrote Skipfish, which is discussed later in this post.

AFL runs on linux systems, as well as Mac OS X, BSD, and Solaris. There is also a [fork for Windows](https://github.com/ivanfratric/winafl), which is good for testing Windows Applications (*duh!*).

After you download AFL, you can install using

```
dpkg -i my.deb
```

