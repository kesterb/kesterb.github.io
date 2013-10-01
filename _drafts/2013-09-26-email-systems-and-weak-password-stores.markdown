---
layout: post
title: "Email systems and weak password stores"
date: 2013-09-24
category: tech
tags: security weak passwords email 
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

This tells me that the passwords are encoded in [Base64](http://en.wikipedia.org/wiki/Base64) format.  

{% highlight text %}
user1 a4c9d893ead4e6d9a1
user2 a1d7d38bd8dea5
user3 a098d387c3c6e7e8e78fe2d6
{% endhighlight %}

[Pwderr]: /images/mail-system-1.png "Password error message"