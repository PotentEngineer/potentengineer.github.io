---
title: Pretty permalinks using IIS 8/IIS 8.5
description: ""
date: 2024-02-09T02:32:58.987Z
preview: ""
tags:
    - wordpress
    - iis
    - blog
categories: []
draft: true
---
When I threw this blog together, I had chose one of the default Permalinks options of:

http://domain/index.php/%postname%

I wanted to get that index.php out of there and that is where my journey began.

I am running WordPress on IIS 8.5 and Windows Server 2012 R2. First, I just went in and attempted to edit the Permalinks under settings, but when I went to save I got the below error.

<img src="{{ site.baseurl }}/assets/images/htaccess_writable.png">

<!--more-->