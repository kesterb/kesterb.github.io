---
layout: post
title: "Weak password stores"
date: 2013-10-01
category: tech
tags: security weak password stores email
---

As a sysadmin I come into contact with a lot of email systems. Being a curious person I tend to want to know how the systems work under the hood. Some times the password limitations of the system give a way clues about the password system. Other times looking at the actual password store reveals much. In two of the email systems I have managed there were glaring issues with the password management. I'm not going to name these email systems outright, but you may be able to figure out what they are from the clues provided.

The first mail system I will refer to as Mail1.  One of the things that made me raise an eyebrow was the password policy for the system.  When updating a password on Mail1 I was presented with the following error message.

![Passwords must be between 4 and 15 characters.][Pwderr]

This is a dead giveaway that the system isn't hashing passwords.  We can probably tell that the passwords will be encrypted, encoded, or stored in plain text on disk.  I took a look at the password store and found passwords were stored in a format like this.

{% highlight text %}
user1 pMnYk+rU5tmh
user2 odfTi9jepQ==
user3 oJjTh8PG5+jnj+LW
{% endhighlight %}

This tells me that the passwords are encoded in [Base64][1] format.  I decided that the next thing to do would be to set try setting the passwords to something I knew so that I could try some analysis on the  stored values.  I set the passwords for the users to these plaintext values.

{% highlight text %}
user1 Password1
user2 Password2
user3 LongPa55w0rd123
{% endhighlight %}

Here are those values from the account store.

{% highlight text %}
user1 pMnYk+rU5tmh
user2 pMnYk+rU5tmi
user3 oNfTh8PGqarnUOLWoJWY
{% endhighlight %}

The fact that the stored versions of the passwords for user1 and user2 were so similar confirmed to me that the passwords were not being encrypted.  If these were encrypted I would have expected a huge difference in the values of user1 and user2.  Since they are almost identical except for the last character I went with the assumption that these passwords were put through a static process before being stored on disk.  My next step was to remove the base64 encoding on the passwords and see if they would decode directly to ASCII values.  Unfortunately they did not and the output was unreadable, so I converted the base64 values to hex values using the following command.

{% highlight bash %}
echo -n "pMnYk+rU5tmh" | base64 -d | xxd -p
echo -n "pMnYk+rU5tmi" | base64 -d | xxd -p
echo -n "oNfTh8PGqarnUOLWoJWY" | base64 -d | xxd -p
{% endhighlight %}

This gave the following outputs.

{% highlight text %}
a4c9d893ead4e6d9a1
a4c9d893ead4e6d9a2
a0d7d387c3c6a9aae750e2d6a09598
{% endhighlight %}

I then converted my plain text passwords to hex values so that I could compare the two.

{% highlight bash %}
echo -n "Password1" | base64 -d | xxd -p
echo -n "Password2" | base64 -d | xxd -p
echo -n "LongPa55w0rd123" | base64 -d | xxd -p
{% endhighlight %}

And the outputs.

{% highlight text %}
50617373776f726431
50617373776f726432
4c6f6e675061353577307264313233
{% endhighlight %}

While comparing the hex strings to each other I noticed that some constant was probably being applied to each password before being stored.  I started with the assumption that the encoded password was being stored in [8-bit ASCII][2] and the constant was the same format.  My next step would be to see if I could find what the constant is, so subtracting the first byte of the plain text password from the encoded password should give me the first byte of the constant.  I should be able to get the second byte of the constant by subtracting the second byte of the plain text password from the encoded password, and so on.

Here is the math from the first password.

{% highlight text %}
0xa4 - 0x50 = 0x54 = T
0xc9 - 0x61 = 0x68 = h
0xd8 - 0x73 = 0x65 = e
0x93 - 0x73 = 0x20 = ' '
0xea - 0x77 = 0x73 = s
0xd4 - 0x6f = 0x65 = e
0xe6 - 0x72 = 0x74 = t
0xd9 - 0x64 = 0x75 = u
0xa1 - 0x31 = 0x70 = p
{% endhighlight %}

This was looking very promising, so I applied the same process for the longest password that was allowed by the system in order to get the entire constant.

{% highlight text %}
0xa0 - 0x4c = 0x54 = T
0xd7 - 0x6f = 0x68 = h
0xd3 - 0x6e = 0x65 = e
0x87 - 0x67 = 0x20 = ' '
0xc3 - 0x50 = 0x73 = s
0xc6 - 0x61 = 0x65 = e
0xa9 - 0x35 = 0x74 = t
0xaa - 0x35 = 0x75 = u
0xe7 - 0x77 = 0x70 = p
0x50 - 0x30 = 0x20 = ' '
0xe2 - 0x72 = 0x70 = p
0xd6 - 0x64 = 0x72 = r
0xa0 - 0x31 = 0x6f = o
0x95 - 0x32 = 0x63 = c
0x98 - 0x33 = 0x65 = e
{% endhighlight %}

And just like that we have our constant "The setup proce".  From there I wrote a python function to automate the process.  This function is Python 3 compliant.  If you are having issues with it in Python 2, try stripping off the `.decode('ascii')` functions.

{% highlight python %}
import binascii
...
def decodePass(passBase64):
	passConst = "The setup proce"
	passPlain = ""
	passHex = binascii.b2a_hex(binascii.a2b_base64(passBase64))
	passConstHex = binascii.b2a_hex(passConst.encode('ascii')).decode('ascii')
	for i in range(0, len(passHex), 2):
		passPlain = passPlain + binascii.a2b_hex(hex(int(passHex[i:i+2], 16) - int(passConstHex[i:i+2], 16))[2:]).decode('ascii')
	return passPlain
{% endhighlight %}

Part two of this post will be coming out soon which will describe the password decoding process for the second mail system.

[Pwderr]: /images/mail-system-1.png "Password error message"

[1]: http://en.wikipedia.org/wiki/Base64 "Wikipedia article on Base64"
[2]: http://www.asciitable.com/ "ASCII Table"
