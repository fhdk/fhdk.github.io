---
title: 'Import pfx certificats on Manjaro'
date: '14:11 11-01-2024'
taxonomy:
    category:
        - docs
---

[Transport Layer Security - ArchWiki ](https://wiki.archlinux.org/title/Transport_Layer_Security)

See also file:///usr/share/ca-certificates/trust-source/README

To extract the certificate stored in a .pfx file to a PEM file (if the it fails - append the **-legacy** option)

    openssl pkcs12 -in cert.pfx -clcerts -out cert.pem [-legacy]

To extract also the private key (again - if this fails - append **-legacy** option)

    openssl pkcs12 -in cert.pfx -nodes -out cert.pem [-legacy]
    
A pfx file is usually password protected, so input the password when challenged.

Copy the extracted file to

    sudo cp cert.pem /usr/share/ca-certificates/trust-source/anchors/

And update the trust

    sudo update-ca-trust
    