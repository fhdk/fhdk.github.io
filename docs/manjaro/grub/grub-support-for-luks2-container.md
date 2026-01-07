---
published: true
date: '2025-07-06 00:00'
publish_date: '2025-07-06 00:00'
title: 'Grub load system from luks2 container'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

grub only supports container using PBKDF2 container type.

See this [commit comment](https://cgit.git.savannah.gnu.org/cgit/grub.git/commit/?id=365e0cc3e7e44151c14dd29514c2f870b49f9755) dated 1010-01-10 where Argon2i and Argon2id was postponed.

For Arch and distributions based on Arch there the option of building a custom grub loader using the recipe located in AUR as [grub-improved-luks2-git](https://aur.archlinux.org/packages/grub-improved-luks2-git).

An article on the details was found at [https://mdleom.com/blog/2022/11/27/grub-luks2-argon2/](https://mdleom.com/blog/2022/11/27/grub-luks2-argon2/)
