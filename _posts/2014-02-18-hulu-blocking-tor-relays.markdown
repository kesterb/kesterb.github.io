---
layout: post
title: "Hulu blocking Tor relays"
date:  2014-02-18
category: tech
tags: tech privacy tor hulu streaming video
---

![Based on your IP address, we noticed you are trying to access Hulu through an anonymous proxy tool.][HuluMsg]

I sat down to watch a show using Hulu Plus on my Roku the other day and was presented with this message whenever I tried to stream any videos from them.  It took me a few minutes to figure out what was going on here.  I run a non-exit [TOR Relay][1] and as such anyone can connect to the TOR network and see that my IP is one of the relays.

I contacted Hulu Support and let them know I was running an exit only relay.  They responded the next business day with the following message.

>Hi Brandon,
>
>Thank you for contacting us. I'm sorry about the anonymous proxy message you're seeing. A proxy is a server that acts as an intermediary between your computer and the Internet. It’s possible to use an anonymous proxy and not realize it.
>
>However, after further investigation, I've confirmed that your IP address ( X.X.X.X ) was incorrectly categorized to be an anonymous proxy. I have submitted a request to approve your IP address for use and you should be up and running in about 3-5 days. If the error appears after this time definitely reach out to us and we'll be happy to dig a bit deeper.

>Once again, I'm very sorry for the inconvenience. If anything else comes up, please don't hesitate to write back.
>
>Thanks,
>Dave
>Hulu Support

My issue was resolved a few days later like they promised, but it did lead me to think that this was a bit of a lazy way to block potential pirates.  For starters they could have actually connected to the network and seen that I was not an exit relay.  Another piece of info they could have used would be to match my connecting IP address to my billing address and see that they were geographically very close.

Of course none of this really touches on the obvious point.  Who the hell would want to watch HD video over TOR?

[HuluMsg]: /images/20140218-hulu-tor-2.png
[1]: https://www.eff.org/torchallenge/what-is-tor/ "What is TOR?"