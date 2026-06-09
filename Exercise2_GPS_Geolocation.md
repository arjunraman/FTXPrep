# Exercise 2 — GPS Geolocation with Kismet

> **Authorized Use Only:** This material is for use exclusively in authorized training environments and FTX scenarios where explicit written permission has been granted by the network owner. Use against any system without authorization is illegal under 18 U.S.C. § 1030 (CFAA), the UK Computer Misuse Act 1990, EU Directive 2013/40/EU, and equivalent laws worldwide. By accessing this material you agree to the full terms in [LICENSE](LICENSE). Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/).

> **Prerequisite:** Complete Exercise 1 first. If you followed the Exercise 1 cleanup steps, the Alfa was reset to managed mode — re-enable monitor mode before starting this exercise.

---

### 1. Re-enable monitor mode and verify hardware
```bash
# Re-enable monitor mode (required if coming from Exercise 1 cleanup)
airmon-ng start wlan1
iwconfig    # confirm wlan1mon is listed

# Start gpsd if not already running
sudo gpsd /dev/ttyUSB0 -F /var/run/gpsd.sock

# Verify GPS receiver and signal
ls /dev/ttyUSB0       # confirm GPS receiver is detected by the OS
cgps -s               # confirm GPS lock — wait for "3D FIX" before proceeding
airodump-ng wlan1mon  # confirm the target BSSID is still visible from your position
```
The GPS receiver communicates via the `gpsd` daemon using the NMEA 0183 protocol. When plugged in, the OS registers it as a serial device at `/dev/ttyUSB0`. If the path is missing, try `dmesg | tail` to see if the device was recognized but assigned a different path (e.g. `/dev/ttyUSB1`). `gpsd` must be running before `cgps -s` will show any data — if it was already running from a prior session, the `gpsd` command above will return an error, which is fine.

`cgps -s` connects to `gpsd` and displays live satellite data. A **2D fix** (≥3 satellites) provides lat/long but altitude is estimated. A **3D fix** (≥4 satellites) provides accurate lat, long, and altitude — always wait for 3D before starting collection, as poor fix quality will produce imprecise placemarks. Satellite acquisition typically takes 30–90 seconds outdoors with clear sky view.

- [gpsd documentation](https://gpsd.gitlab.io/gpsd/)
- [NMEA 0183 standard overview](https://www.nmea.org/content/STANDARDS/NMEA_0183_Standard)

---

### 2. Run Kismet
```bash
sudo kismet
```
Kismet is a **passive** wireless network detector — it does not transmit any frames, making it undetectable by the target AP. Unlike `airodump-ng`, every device record Kismet stores is tagged with the GPS coordinates at the time of detection, building a spatial map of the RF environment.

Kismet writes its data to a `.kismet` file (a SQLite database) in the current directory, logging each detected BSSID, signal strength, and GPS position. The web UI runs locally at `http://localhost:2501`. Right-click that link in the terminal to open it in a browser.

- [Kismet quickstart guide](https://www.kismetwireless.net/docs/readme/quickstart/)
- [Kismet GPS integration](https://www.kismetwireless.net/docs/readme/gps/)

---

### 3. Configure Kismet in the GUI
1. Hamburger menu (top left) → **Enable Datasource** → select `wlan1mon` → **All Channels**.
2. Confirm the GPS icon in the top-right shows active satellite data (coordinate values updating).
3. Let Kismet collect until the target BSSID appears in the **Devices** tab.
4. `Ctrl+C` Kismet in the terminal and close the browser.

Selecting **All Channels** tells Kismet to channel-hop across the full 2.4 GHz and 5 GHz spectrum. The GPS coordinate recorded for each device is the position with the strongest observed signal — if you move around while collecting, Kismet refines the stored location. Stopping with `Ctrl+C` gracefully flushes all pending writes and closes the SQLite database. Killing the process (`kill -9`) can corrupt the `.kismet` file.

- [Kismet datasource configuration](https://www.kismetwireless.net/docs/readme/datasources/)
- [Kismet web UI overview](https://www.kismetwireless.net/docs/readme/webui/)

---

### 4. Convert the capture to KML
```bash
sudo kismetdb_to_kml --in Kismet-<TIMESTAMP>.kismet --out OUTPUT.kml
```
KML (Keyhole Markup Language) is an XML-based geographic format standardized by the OGC and used by Google Earth, Google Maps, and most GIS platforms. `kismetdb_to_kml` queries the `.kismet` SQLite database and exports each detected device as a `<Placemark>` element containing its name (SSID/BSSID), GPS coordinate, and signal data.

Elevated privileges are required because Kismet (run as root) owns the `.kismet` file. The output filename (`OUTPUT.kml`) will be written to the current working directory.

> Replace `Kismet-<TIMESTAMP>.kismet` with the actual filename generated during your collection — it follows the format `Kismet-YYYYMMDD-HH-MM-SS-N.kismet`.

- [kismetdb_to_kml documentation](https://www.kismetwireless.net/docs/readme/kismetdb/kismetdb_to_kml/)
- [KML reference (Google Developers)](https://developers.google.com/kml/documentation/kmlreference)

---

### 5. Visualize in Google Earth
1. Open **Google Earth** from the Kali application menu.
2. **File → Open** → change the file filter to *All Files* → select `OUTPUT.kml`.
3. Search for the target BSSID (`<TARGET_BSSID>`) in the left panel to locate it on the map.
4. Right-click the target placemark → **Properties** → change the symbol icon and save to make it easier to identify.
5. Record the target's location using the required coordinate system:
   - **Lat/Long (decimal degrees)** — visible under Properties.
   - **MGRS (Military Grid Reference System)** — Tools (top banner) → change to Military Grid System → check Properties for the grid coordinate.

**MGRS background:** MGRS is a geocoordinate standard used by NATO militaries. It divides the Earth into 6°×8° Grid Zone Designators, each subdivided into 100km² squares identified by a two-letter code. Within each square, position is expressed as an easting/northing pair at the desired precision: 6-digit (100m), 8-digit (10m), or 10-digit (1m). Example: `38SMB4611052890` = 38S zone, MB square, 10m precision.

- [MGRS explanation — NGA](https://www.nga.mil/ProductsServices/GeodesyandGeophysics/GeodeticTranslator.html)
- [Army MGRS reference (ATP 3-34.80)](https://armypubs.army.mil/)
- [Google Earth KML import guide](https://support.google.com/earth/answer/7365595)

---

### 6. Clean up
```bash
# Remove all artifacts from the exercise
rm *.kismet OUTPUT.kml

# Clear shell history
echo -n > ~/.zsh_history && exec zsh
```
The `echo -n >` truncates the history file in place without deleting it. `exec zsh` starts a fresh shell session with an empty history. Also remove the target WiFi network from **Settings Manager → Advanced Network Configurations** in Kali so the Alfa does not auto-reconnect when the next trainee plugs it in.
