---
layout: post
title: "Hardening Grub"
date:  2008-11-30
category: tech
tags: grub linux security systems administration hardening
---

Limiting permissions is an extremely important part of being a systems administrator.  Restricting access to boot up options inside of GRUB is no different.  That is why understanding the password and lock directives inside of the menu.lst file are so important.  Allowing users to boot just any old option from the menu.lst file can be dangerous.  Epically when one of the options is to drop to user into a single user mode with no password on the root account.

Enter the password directive to save us.  The password directive is very simple to understand.  Here is the definition from the GRUB manual:

{% highlight sh %}
password [--md5] passwd [new-config-file]
{% endhighlight %}

I will break these down for you.  The flag –md5 tells GRUB that the following text is actually encrypted using md5crypt.  In order to encrypt using md5crypt you will want to open up a terminal session in your running \*nix environment then type the command **grub**.  Once inside of the grub program (your command prompt will turn into: grub>)  you will want to type **md5crypt**.  Enter your password and then copy the output after “Encrypted:”.  That is your encrypted password.

Now for the cool part.  Adding a password will harden GRUB by not allowing users to change the boot options.  If you want to only give your users one or two boot options, but also have the option for administrators to boot into recovery mode, then you will want ot use the optional [new-config-file] parameter.  This will let you specify a new menu.lst file (such as /boot/grub/menu-admin.lst) that requires the password to be put in before the options are shown.

Another option is to use the lock directive.  The lock directive will prevent execution past the lock keyword unless the correct password is put in.  For example:

{% highlight sh %}
title This entry is too dangerous to be executed by normal users
lock
root (hd0,1)
kernel /no-security-os
{% endhighlight %}

Before the user can execute root(hd0,1) they will be required to put in the correct password.