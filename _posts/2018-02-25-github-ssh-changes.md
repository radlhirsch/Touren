---
layout: post
title:  "Github increases SSH requirements"
date:   2018-02-25 21:00:00
excerpt: "Github has removed weak cryptographic standards after announcement."
image:
thumb: /assets/img/thumbs/github_logo_288x174.jpg
tags: [github, git, ssh, development]
categories: [posts, development]
comments: true
lang: en
ref: github-ssh-changes
---

Today I received the following error message when accessing my Github repository on a Windows computer:

>Couldn't agree a key exchange algorithm (available: curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521)

After a short research I discovered the reason for this:

The following cryptographic standards considered vulnerable are no longer supported since 22nd Farbruar 2018:

* TLSv1/TLSv1.1: This applies to all HTTPS connections, including web, API, and Git connections to https://github.com and https://api.github.com.
* diffie-hellman-group1-sha1: This applies to all SSH connections to github.com
* diffie-hellman-group14-sha1: This applies to all SSH connections to github.com

If you have problems with authentication at github, you should update your git client software! 

See also:[Weak cryptographic standards removed](https://github.com/blog/2507-weak-cryptographic-standards-removed) 