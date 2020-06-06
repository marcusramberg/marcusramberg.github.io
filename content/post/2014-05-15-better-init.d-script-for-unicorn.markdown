---
date: "2014-05-15T00:00:00Z"
published: true
tags:
  - unicorn
  - ruby
  - sysadmin
title: Improved init.d script for unicorn
---

I went looking for a init.d script for Unicorn today, and Google seemed to
give me various links to [this one](https://gist.github.com/jaygooby/504875) -
It's a nice and simple script, but it has a couple of issues. For one, it's
missing the LSB header, and more importantly it is missing status.

This does not make Ansible very happy, as it is using the status to check if
the service is running before enforcing state=stopped or state=running. I spent
some time tracking this down, so I wanted to share a fixed version with LSB
headers and status implemented, in case someone else has the same problem.

{% gist anonymous/e3110d827cad050ea749 %}
