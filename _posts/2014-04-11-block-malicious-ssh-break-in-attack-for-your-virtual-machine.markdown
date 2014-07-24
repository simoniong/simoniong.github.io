---
layout: post
title: "block malicious ssh break-in attack for your virtual machine"
date: 2014-04-11 20:58:01 +0800
comments: true
categories: security
---

A few months ago, I setup my personal [GitLab](https://www.gitlab.com) in [DigitalOcean](https://www.digitalocean.com/), It serve as my personal "Github" to store my private gems, and libraries. I use so-called one key installer provided by DigitalOcean to setup this service, and it's quite nice, since I don't need to make my hand dirty to setup Gitlab form the very beginning.

It worked fine until a few days ago, It's found out that my GitLab service get stuck for a little bit, and I need to restart my virtual machine to resume the service. After research for some while, an abnormal malicious ssh break-in attack was found from my /var/log/auth.log

{% codeblock lang:bash /var/log/auth.log %}
Apr 10 05:34:32 do sshd[4846]: Failed password for root from 203.81.22.35 port 51847 ssh2
Apr 10 05:34:32 do sshd[4846]: Received disconnect from 203.81.22.35: 11: Bye Bye [preauth]
Apr 10 05:34:35 do sshd[4848]: Address 203.81.22.35 maps to mail.ckgsb.edu.cn, but this does not map back to the address - POSSIBLE BREAK-IN ATTEMPT!
Apr 10 05:34:35 do sshd[4848]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=203.81.22.35 user=root
Apr 10 05:34:38 do sshd[4848]: Failed password for root from 203.81.22.35 port 52189 ssh2
Apr 10 05:34:38 do sshd[4848]: Received disconnect from 203.81.22.35: 11: Bye Bye [preauth]
Apr 10 05:34:41 do sshd[4850]: Address 203.81.22.35 maps to mail.ckgsb.edu.cn, but this does not map back to the address - POSSIBLE BREAK-IN ATTEMPT!
{% endcodeblock %}

It seems the attacker use brute force to generate password token, and try to break into my virutual machine for every 3 sec. And this block my GitLab service too!

This kind of brute force attach can be easily resloved by Failtoban. Failtoban keep monitoring your service logs to identify malicious attack, and update firewall rules, reject the related Ip addresses for a specific amount of time. It work for apache, ssh, and other services.

And thanks to DigitalOcean's support [article](https://www.digitalocean.com/community/articles/how-to-protect-ssh-with-fail2ban-on-ubuntu-12-04), It's very easy to install & config failtoban too!

I will be appreciated if you have any other suggested tools about blocking malicious attack, please drop down your recommendation in the comment.
