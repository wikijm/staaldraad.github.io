---
layout: post
title:  "Huawei Quidway Password Extraction"
date:   2015-06-17 20:33:00
categories: huawei security snmp
---
In the past our favourite hardware vendor to pick on was Cisco these days however, there is a new kid on the block - Huawei. We all know about the dangers of SNMP and default community strings, think Cisco and tftp. Seems like Huawei suffers from similar fail. Like all routers, switches, servers, ect out there, Huawei devices can be managed through SNMP. And just like other devices in the wild, SNMP is mostly configured with the community strings public and private as defaults. Using snmpwalk standard configuration information like routing tables and VLAN configurations can be extracted. 

The Huawei Quidway firewall/router devices have an interesting flaw though, in the `upper` MIB values there lies gold. The most interesting MIB value can be found at  _1.3.6.1.4.1.2011.5.2.1.10._. 

![Huawei SNMPwalk]({{ site.url }}/assets/huawei_snmp.png)

It isn’t immediately evident, but those SNMP oids return the usernames of local users as Hex-STRINGs and the passwords for those users as clear-text STRINGs. The most interesting part here is that those passwords will be returned in clear-text, even when they have been configured as `cipher`. The usernames and passwords are used to administer the Quidway firewall either through SSH, Telnet or the Web interface. Simply query SNMP and retrieve valid usernames and passwords, log in with SSH or the Web admin interface, and achieve complete device compromise. 

I've written a small script to extract these values automatically. The tool wraps around snmpwalk, so configure the script to include the path to snmpwalk, or add snmpwalk to your PATH. Once set up, the tool will query the target host for the usernames and passwords, convert the usernames to strings and print these out with the corresponding password. Testing this on an internal showed that 9/10 of the different Huawei devices deployed on the target network would reveal the username and password through SNMP. The usernames returned will also include the login domain, by default this will be username@local.domain but could also include RADIUS users and other users. The tool will by default include the user domain in the output. To log in through SSH, FTP, web interface, simply drop the @local.domain portion of the output.

![Huawei extract passwords with SNMP]({{ site.url }}/assets/huawei_password_snmp.png)

The tool can be found here: 
{% gist staaldraad/bacb4bf0adc7b78c73eb %}


Happy router/firewall hunting!

As an added bonus, a script to decrypt passwords stored in Huawei config files (older configs used a known key and DES encryption). [Huawei Decrypt]({https://gist.github.com/staaldraad/605a5e40abaaa5915bc7})

