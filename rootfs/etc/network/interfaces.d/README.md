# Where did `wlan0` go?

Since I want to avoid accidentally publishing my WiFi SSID and password, I `git rm --cached wlan0`ed it, and added a `.gitignore` to prevent accidentally re-adding it.

Here is a sanitized version of its contents:

```text
# To enable wireless networking, uncomment the following lines and -naturally-
# replace with your network's details.
#
allow-hotplug wlan0
iface wlan0 inet static
    address 192.168.1.5/24
    gateway 192.168.1.1
    wpa-proto RSN # Force WPA2 negotiation.
    wpa-ssid "NSA Surveillance Van"
    wpa-psk UncleS@mIsWatchingY0u
# iface wlan0 inet6 dhcp
```