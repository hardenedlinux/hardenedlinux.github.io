---
layout: post
title:  "STIG-4-Debian"
date:   2015-06-19 03:38:39
categories: jekyll update
---
By:n3o4po11o

##Why STIG?

STIGs is bring by a government agency called The Defense Information
System Agency(DISA), which is entity responsible for maintaining the security
posture of the Department of Defence(DOD) IT infrastructure. After we heard 
how the NSA fuck this world from Mr.Sn0wd3n.We will pay more attention about
how *they* do the defense.   

DoD use this Security Technical Implantation Guides to All DoD IT assets before
they allowed to operate.   

And the STIGs classification system based on Mission Assurance Catagory (I-III)
and Confidentiality Level (Public-Classified), giving you 9 different possible 
combinations of config requirements.    

##Why Debian?

In this scripts I use the Debian 8, Debian has a lot security mechanism, and 
some good feature, especially "ReproducibleBuilds". I use the STIG for Red hat
6 v1r7 to porting STIG for Debian 8.   

Debian always has active maintenance, and has good security in default-configuration   

##What's different?

In STIG for RHEL-06, there's some service doesn't exist in debian, or some command or
some purpose implement in different way.

You could find the porting-log in the repo [STIG-4-Debian](https://github.com/hardenedlinux/STIG-4-Debian)

But the general idea are all based on STIG For RHEL-06 v1r7

##TODO

There's a lot of TODO   
Because this version I release just a simple "POC", and just a pre-release version.
It doesn't even cover all the "check"

But I will release the first version of "full-check" version soon, and add Classification 
and Severity right after full-check, I think it will release in next month.


##Reference

[1]Difference between hardening guides (CIS, NSA, DISA)
http://security.stackexchange.com/questions/73164/difference-between-hardening-guides-cis-nsa-disa

[2]What Are “STIGs” and How Do They Impact Your Overall Security Program?
http://www.seguetech.com/blog/2013/05/06/stigs-impact-overall-security-program

[3]Beyond compliance: DISA STIGs’ role in cybersecurity
http://gcn.com/Articles/2015/05/14/DISA-STIG-compliance.aspx?Page=1

[4]Security Technical Implementation Guides (STIGs)
http://iase.disa.mil/stigs/Pages/index.aspx

[5]DISA RHEL 6 STIG V1 R7
http://iasecontent.disa.mil/stigs/zip/Apr2015/U_RedHat_6_V1R7_STIG.zip

[6]Defense Information Systems Agency
https://en.wikipedia.org/wiki/Defense_Information_Systems_Agency

[7]ReproducibleBuilds
https://wiki.debian.org/ReproducibleBuilds
