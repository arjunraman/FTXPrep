# FTX WiFi Red Team

Reference guides for a Field Training Exercise (FTX) covering wireless network exploitation and GPS-based geolocation of a target access point.

> **License:** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/) — Free to share for non-commercial purposes with attribution. No derivatives may be distributed. See [LICENSE](LICENSE) for full terms including warranty disclaimer, limitation of liability, and indemnification notice.
>
> **Authorized Use Only:** These techniques are for use exclusively in authorized training environments and FTX scenarios where explicit written permission has been granted by the network owner. Use against any system without authorization is illegal under 18 U.S.C. § 1030 (CFAA), the UK Computer Misuse Act 1990, EU Directive 2013/40/EU, and equivalent laws worldwide. By accessing this repository you agree to the terms in [LICENSE](LICENSE). The author assumes no liability for unauthorized or unlawful use.

---

## Exercises

| # | Exercise | Description |
|---|---|---|
| 1 | [WiFi Exploitation](Exercise1_WiFi_Exploitation.md) | Capture WPA2 handshake, crack the PSK, enumerate hosts, and execute effects |
| 2 | [GPS Geolocation](Exercise2_GPS_Geolocation.md) | Geolocate the target AP with Kismet and visualize in Google Earth |
| — | [Operator Reference Card](Wifi_GPS_Operator%20Notes) | Condensed quick-reference covering both exercises |

---

## Hardware

Both exercises require external hardware. The built-in laptop NIC does not support monitor mode or packet injection and must not be used to connect to the target network.

| Item | Recommended Model | Est. Cost | Notes |
|---|---|---|---|
| USB WiFi Adapter | [Alfa AWUS036ACH](https://www.amazon.com/s?k=Alfa+AWUS036ACH) | ~$40 | Dual-band, supports monitor mode and packet injection on Kali Linux out of the box |
| USB GPS Receiver | [GlobalSat BU-353-S4](https://www.amazon.com/s?k=GlobalSat+BU-353-S4) | ~$25 | NMEA-compatible, recognized automatically by Kismet via `/dev/ttyUSB0` |

**Total estimated hardware cost: ~$65**

---

## Software

Most tools are pre-installed on Kali Linux. Items marked with * require a one-time setup step — see **Prerequisites** below.

| Tool | Purpose |
|---|---|
| aircrack-ng suite | WiFi capture, deauth, and cracking |
| `macchanger` | MAC address randomization |
| `nmap` | Host discovery and service scanning |
| `gpsd` + `cgps` | GPS daemon and live satellite status viewer |
| `kismet` + `kismetdb_to_kml` | Wireless detection with GPS tagging and KML export |
| `vlc` | RTSP live video stream access from camera targets |
| `rockyou.txt` * | Password dictionary — may need to be unzipped on first use |
| Google Earth * | Visualize GPS-tagged AP location — not pre-installed, requires download |
| [tplink_smartplug.py](https://github.com/softScheck/tplink-smartplug) * | TP-Link smart plug control — download from GitHub |

---

## Prerequisites (First-Time Setup)

Run these once before the first exercise session:

```bash
# 1. Unzip rockyou.txt if not already extracted
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# 2. Install Google Earth
wget -q -O /tmp/google-earth.deb https://dl.google.com/dl/earth/client/current/google-earth-pro-stable_current_amd64.deb
sudo dpkg -i /tmp/google-earth.deb
sudo apt-get install -f -y   # resolve any dependency issues

# 3. Download the TP-Link smart plug script
wget -O tplink_smartplug.py https://raw.githubusercontent.com/softScheck/tplink-smartplug/master/tplink_smartplug.py

# 4. Verify gpsd is installed
sudo apt-get install -y gpsd gpsd-clients
```

---

## References

- [aircrack-ng suite](https://www.aircrack-ng.org/)
- [Kismet](https://www.kismetwireless.net/)
- [tplink-smartplug script](https://github.com/softScheck/tplink-smartplug)
