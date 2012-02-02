---
layout: post
author: corey
title: Engine Yard rrdtool Error
tags:
- Technology
- Web
---
If you are trying to use [rrdtool][1] on an [Engine Yard][2] instance and
your Ruby version is 1.8.7, you might get this error:

{% highlight irb %}
irb(main):001:0> require 'RRD'
LoadError: librrd.so.2: cannot open shared object file: No such file or directory - /usr/lib/ruby/site_ruby/1.8/i686-linux/RRD.so
from /usr/lib/ruby/site_ruby/1.8/i686-linux/RRD.so
from (irb):1
from :0
{% endhighlight %}

It's sort of confusing because it looks like RRD.so doesn't exist, but it does:

{% highlight bash %}
~ $ ls /usr/lib/ruby/site_ruby/1.8/i686-linux/RRD.so
/usr/lib/ruby/site_ruby/1.8/i686-linux/RRD.so
{% endhighlight %}

What is happening is that RRD.so is linked to a missing shared library:

{% highlight bash %}
/usr/lib/ruby/site_ruby/1.8/i686-linux $ ldd -v RRD.so
linux-gate.so.1 => (0xb7f23000)
libruby18.so.1.8 => /usr/lib/libruby18.so.1.8 (0xb7e3f000)
librrd.so.2 => not found
libdl.so.2 => /lib/libdl.so.2 (0xb7e34000)
libcrypt.so.1 => /lib/libcrypt.so.1 (0xb7e05000)
libm.so.6 => /lib/libm.so.6 (0xb7ddf000)
libc.so.6 => /lib/libc.so.6 (0xb7cab000)
librt.so.1 => /lib/librt.so.1 (0xb7ca2000)
/lib/ld-linux.so.2 (0x80000000)
libpthread.so.0 => /lib/libpthread.so.0 (0xb7c8a000)
{% endhighlight %}

Which brings up the question where does RRD.so come from?

{% highlight bash %}
~ $ equery belongs RRD.so
[ Searching for file(s) RRD.so in *... ]
net-analyzer/rrdtool-1.4.5 (/usr/lib/ruby/site_ruby/1.8/i686-linux/RRD.so)
{% endhighlight %}

However the package net-analyzer/rrdtool only provides librrd.so.4 not librrd.so.2:

{% highlight bash %}
~ $ equery belongs librrd.so
[ Searching for file(s) librrd.so in *... ]
net-analyzer/rrdtool-1.4.5 (/usr/lib/librrd.so -> librrd.so.4.1.4)
{% endhighlight %}

That seems mean that Engine Yard is pre-building the packages they provide
and the current rrdtool package was built incorrectly.

I opened a support issue with Engine Yard about this and have been
going back and forth with them about it. Finally I got this from them:

> "It looks like there was an issue with compilation when we upgraded RRDTool
> to 1.4.5 on Ruby 1.8.7. If you need to use it, you shouldn't have a problem
> with 1.9.2 or 1.9.3. As of right now, we'll have to look into adding support
> on 1.8.7 but there's no timeframe."

A few days later that was followed up with:

> "Moving forward, our developers are going to focus on more pressing issues
> right now than edge cases."
>
> "I will make sure that we Roadmap this since we are still supporting 1.8.7
> but no idea on a time frame."

So the rrdtool package they provide is broken for Ruby 1.8.7 and they probably
won't fix it any time soon.

The only workaround I could think of was to have a chef recipe to remove
the offending RRD.so file and then install the librrd gem which will build
a RRD.so file that is useable. I tested this on a staging environment and
everything worked as expected.

However, rather than deploying this workaround we decided to go in a different
direction to capture and report the metrics we where interested in for our
application.

[1]: http://oss.oetiker.ch/rrdtool/
[2]: http://engineyard.com
