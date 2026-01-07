---
title: 'Disable NetworkManager connectivity check'
taxonomy:
    category:
        - docs
---

## Connectivity check
NetworkManager handles network connection and periodically checks if an internet connection exist - default 300s.

Disable this by creating a file
```
sudo cp /usr/lib/NetworkManager/conf.d/20-connectivity.conf /etc/NetworkManager/conf.d
```
Append the following to the file
```
echo "interval=0" | sudo tee -a /etc/NetworkManager/conf.d/20-connectivity.conf
```
Reload the configuration files and restart the service
```bash
sudo systemctl daemon-reload
sudo systemctl restart NetworkManager
```
The interval = 0 disables the check.
Change the uri to what ever service trusted or controllable.

Consider the implications as NetworkManager is now agnostic on the internet connection.

