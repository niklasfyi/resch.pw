---
title: "Cryptpad Uberspace Guide"
date: 2019-07-08T14:43:46+02:00
lastmod: 2020-10-29T10:16:00+1:00
tags:
 - uberspace
 - linux
 - shared-hosting
 - cryptpad
 - guide
description: "Cryptpad is a Zero Knowledge realtime collaborative editor. It is based on Node.js and comes with encryption. It relies on the ChainPad. In this guide you will learn how to set it up on an uberspace!"
---

# Disclaimer

This guide was written for [UberLab](https://lab.uberspace.de/guide_cryptpad.html) by me. This is just a copy but maybe it is more up-to-date due to the fact that is's easier for me to update this site, than the UberLab. I am trying to keep both side up-to-date.

# Cryptpad

[Cryptpad](https://cryptpad.fr/) is a Zero Knowledge realtime collaborative editor. It is based on [Node.js](https://nodejs.org/en/) and comes with encryption. It relies on the [ChainPad](https://github.com/xwiki-contrib/chainpad/).

---

# Note

For this guide you need some tools:
 - [web backends](https://manual.uberspace.de/web-backends.html)
 - [Node.js](https://manual.uberspace.de/lang-nodejs.html) and its package manager [npm](https://manual.uberspace.de/lang-nodejs.html#npm)
 - [supervisord](https://manual.uberspace.de/daemons-supervisord.html)
 - [domains](https://manual.uberspace.de/web-domains.html)

# Prerequisites

Set up the backends:

{{< highlight bash>}}
[isabell@stardust ~]$ uberspace web backend set / --http --port 3000
Set backend for / to port 3000; please make sure something is listening!You can always check the status of your backend using "uberspace web backend list".
{{< / highlight >}}

Now let's get started with Cryptpad.

We're using [Node.js](https://manual.uberspace.de/lang-nodejs.html) in the stable version 10:

{{< highlight bash>}}
[isabell@stardust ~]$ uberspace tools version use node 10
Selected Node.js version 10
{{< / highlight >}}

We also need [Bower](https://bower.io/):

{{< highlight bash>}}
[isabell@stardust ~]$ npm install -g bower
{{< / highlight >}}

# Installation

Start with cloning the Cryptpad source code from [Github](https://github.com/xwiki-labs/cryptpad) and be sure to replace the branch 2.21.0 with the current release number from the [feed](https://github.com/xwiki-labs/cryptpad/releases):

{{< highlight bash>}}
[isabell@stardust ~]$ git clone --branch 2.21.0 https://github.com/xwiki-labs/cryptpad.git
~/cryptpad
Cloning into '~/cryptpad'...
remote: Enumerating objects: 172, done.
remote: Counting objects: 100% (172/172), done.
remote: Compressing objects: 100% (105/105), done.
remote: Total 43165 (delta 99), reused 109 (delta 67), pack-reused 42993
Receiving objects: 100% (43165/43165), 85.51 MiB | 4.81 MiB/s, done.
Resolving deltas: 100% (30989/30989), done.
Note: checking out '135182ea0a3500d27afe0146c94e112e1726ae7e'.

You are in 'detached HEAD' state. You can look around, make experimentalchanges and commit them, and you can discard any commits you make in thisstate without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you maydo so (now or later) by using -b with the checkout command again. Example:

git checkout -b

Checking out files: 100% (4319/4319), done.
{{< / highlight >}}

Now we need to install some dependencies:

{{< highlight bash>}}
[isabell@stardust ~]$ cd ~/cryptpad
[isabell@stardust cryptpad]$ npm install
(...)
added 168 packages from 186 contributors and audited 311 packages in 14.352s
found 0 vulnerabilities
[isabell@stardust cryptpad]$ bower install
(...)
bower install       open-sans-fontface#1.4.2
bower install       jquery#2.1.4
bower install       bootstrap#4.3.1
(...)
{{< / highlight >}}

# Configuration

## Copy example configuration

{{< highlight bash>}}
[isabell@stardust cryptpad]$ cp config/config.example.js config/config.js
{{< / highlight >}}

Edit `config/config.js` and change the value of the variable `_domain` to your domain, like so:

{{< highlight javascript>}}
/*
globals module
*/
var _domain = 'https://isabell.uber.space/';
{{< / highlight >}}

## Setup daemon

Create `~/etc/services.d/cryptpad.ini` with the following content:

{{< highlight ini>}}
[program:cryptpad]
directory=%(ENV_HOME)s/cryptpad
command=node server
autorestart=yes
{{< / highlight >}}

Now let's start the service:

After creating the configuration, tell [supervisord](https://manual.uberspace.de/daemons-supervisord.html) to refresh its configuration and start the service:

{{< highlight bash>}}
[isabell@stardust ~]$ supervisorctl reread
SERVICE: available
[isabell@stardust ~]$ supervisorctl update
SERVICE: added process group
[isabell@stardust ~]$ supervisorctl status
cryptpad                            RUNNING   pid 26020, uptime 0:03:14
{{< / highlight >}}

# Screenshot

{{< figure src="./images/screenshot.png" width="100%" >}}

# Customization

For any further configuration or customization you should have a look at the [Cryptpad Wiki](https://github.com/xwiki-labs/cryptpad/wiki/).

# Updates

*Check the update [feed](https://github.com/xwiki-labs/cryptpad/releases) regularly to stay informed about the newest version*

If there is a new version available, you can get the code using git. Replace the version number ``2.19.0`` with the latest version number you got from the release [feed](https://github.com/xwiki-labs/cryptpad/releases):

{% highlight shell %}
[isabell@stardust ~]$ cd ~/cryptpad
[isabell@stardust cryptpad]$ git pull origin 2.19.0
From https://github.com/xwiki-labs/cryptpad
 * tag                 2.19.0     -> FETCH_HEAD
Already up to date.
[isabell@stardust cryptpad]$
{% endhighlight%}

Then you need to restart the service, so the new code is used by the webserver:

{{< highlight bash>}}
[isabell@stardust cryptpad]$ supervisorctl restart cryptpad
[isabell@stardust cryptpad]$
{{< / highlight >}}

----

Last test: with Cryptpad 2.24.0 and Uberspace 7.3.3.0
