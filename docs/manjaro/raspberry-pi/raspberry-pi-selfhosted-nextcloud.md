---
title: 'Sharing Pi NextClound'
published: true
date: '03-10-2020 08:34'
publish_date: '03-10-2020 08:34'
taxonomy:
    category:
        - docs
    tag:
        - registrar
        - joker.com
        - nextcloud
        - domain
---

## Share your Pi NextCloud
When you have setup a NextCloud instance on your Pi - next step is sharing it with your family and friends. This topic is not on installing Pi or NextCloud or the internal configuration but the steps to take to share it using a public domain name.

This topic stems from the [Self hosted cloud service][4] topic and I thought - maybe a little write-up on the steps to share an instance using a domain-name.

Sure - you can do this using the public IP address of your internet connection - but several questions rise

* How many of your family and friends can remember your IP?
* And what if it changes?
* What is the bare minimal I need to do?
* What do I need to know?
* What must I prepare for?

## Registration
This is where you need a registrar.

A registrar is an entity supplying usage rights for internet domain names - e.g. [pacbang-linux.org][1] or [nix.dk][2]. 

One thing to remember - you don't buy a domain name - you buy the right to use it for period - usually a year - and the fee is paid up-front. If - subsequently - you don't pay - you loose that usage right. Many registrars allow for for multiple years upfront - some times at a discounted rate.

### Important
There is three contact levels for a given domain - ensure you are the owner contact - because this is where some shady resellers cheat - especially when you request who-is privacy.

There is a lot of registrars and not all are equally easy to work with - sometimes you only realize it when you want to transfer your domain to another registrar and the pain begins.

* The owner contact
* The pay-the-bills contact
* The technical contact

I know - there is a lot of registrars - so you may know some I don't or you may prefer one over mine - so this is to discuss why one registrar should be chosen over another, good vs. bad etc. - the registrar I have chosen is a well-known registrar with decades of spotless operation.

## Joker registrar
Joker is a swiss based registrar with the privacy and security Switzerland is known for.

As I am former self-employed IT consultant working with nearly every aspect of IT operation, hosting, DNS, maintenance and domain reseller - joker.com has by been among the best registrars to work with (I still have reseller account - albeit not using it anymore).

Joker's [service portfolio][3] has grown over the years - including both a free who-is privacy and an extended privacy at a nominal cost - and besides the yearly domain fee - everything else is free as in free coffee. The also offer free mail-forward using spam- and malware filters and a dynamic DNS service if you ISP is rotating your public IP.

## Domain name
You may have an idea - but is it available? You can [search][4] Joker's database to get an idea of what is available. The registration is straight forward and easy to complete.

## Joker DNS management
While there is several possibilities for setting up the DNS - with joker you leave it at the A record.

But as many users - if you tell them the web address - will be prepending the domain with www - it is best practice to have a CNAME record pointing to your root domain

When logged into your joker account

1. If not already there click **My Joker** -> **Domains** -> **DNS**
2. Click Add New Record
    a. Set Type field to A
    b. leave Name(Subdomain) empty
    c. enter your public IP in IP address
    c. click Add
3. Repeat above - this time with CNAME
    a. In Type - select CNAME
    b. In Name - enter www
    c. In Alias - enter your domain name without www
    d. Click Add
4. Repeat for any email you want to add
    a. Set Type to Email
    b. Set the leftmost part of the email address
    c. Set an existing mailbox to receive the mail
    d. Click Add

If you do not have a dedicated public IP - even the DHCP assigned addresses rarely change - you may need to use their Dynamic DNS service (which I have no experience with).

When you are done - Click Save changes at the page bottom.

DNS records created have a TTL (time to live) at 86400s - so waiting for a change to propagate can be a pain. Clicking options for the A-record - then the Edit icon - you can set TTL to e.g. 600s which is 10m. Remember to Save changes - otherwise the changes will not be saved.

## Setting up your router
The last step is configuring your router. As there is a lot of different interfaces I cannot possibly know them all.

But the requirements is identical

1. Assign a static IP address on your LAN to your NextCloud box
    (you may already have done so - if - skip to next bullet)
2. Open your routers configuration interface and login
3. Locate the configuration section/page called Portforwarding
4. Create a rule from WAN to LAN and allow incoming traffic
   - port 80 -> nextcloud IP
   - port 443 -> nextcloud IP
5. Save the changes and restart your router.
6. Test access
7. Important Security considerations
   - Within minutes of opening your router - botnets will begin scanning your system for vulnerabilities
   - The bots will very quickly know it is a nextcloud instance and begin hammering your login
   - The bots will also try to access known nextcloud extension packages which are known to be exploitable
   - Keep your nextcloud up-to-date
   - Regularly check your logs
   - Employ a fail-2-ban mechanism 
     - A term used for blocking an IP for a period of time when it repeatedly fail to login
     - I don’t know if NextCloud has something built-in.

### Testing your setup
Depending on your router - you may not be able to connect directly to your routers WAN address - in which case you can test the access using e.g. your phone to connect to it (disable wireless on your phone - forcing it to use broadband).

Another option is connecting to the internet using a VPN - then access your domain - but bear in mind that you will need to disconnect from VPN to be able to access your router’s configuration interface - if you need to troubleshoot the rules.

If you can connect :partying_face:

## Conclusion
That is about it - welcome to the world of server admins.


[1]: https://pacbang-linux.org
[2]: https://root.nix.dk
[3]: https://joker.com/domain/features
[4]: https://joker.com