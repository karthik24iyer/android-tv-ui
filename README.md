# Android TV UI

A fast, calm home screen and cleanup kit for my Android TV, by **Karthikeyan Iyer**.

Your apps, your HDMI inputs, a clock and phone batteries. No ads, no sponsored rows.
Apps are turned off, never deleted, and everything can be undone.

## My TV

VW (Skyworth) SW-22AE NF: Google TV, Android 14, 1.9 GB RAM, at `192.168.0.6` on the KnS Wi-Fi.
That IP is reserved for the TV's MAC `FC:23:CD:91:E3:B1` in the TP-Link router
(Network → LAN Settings → Address Reservation).

- 17 apps turned off, screensaver off, sleep after 5 minutes. Every change, its old value and how
  to undo it: [tv-cleanup-changes.txt](tv-cleanup-changes.txt).
- This home screen installed, with Google TV's home screen and setup app turned off
  (the setup app kept switching Google TV's home back on).
- Free memory: 713 MB before. The apps turned off were using about 330 MB.

## Phone battery on the home screen

Phones on the Wi-Fi can show their battery left of the Wi-Fi icon, like `Karthik's S23  84% ⚡`.
The home screen listens on port 8765; a phone hides itself after 30 quiet minutes.

```
http://192.168.0.6:8765/battery?name=Karthik's%20S23&level=84&charging=on
```

`charging` takes on/off, true/false, yes/no, or a number (charging wattage: above 0 means charging). Spaces in the name must be `%20`.

**MacroDroid on the phone**

1. Add Macro. Triggers: **Battery Level → Any change**, plus **Power Connected** and
   **Power Disconnected** (so the bolt shows straight away).
2. Action: **Connectivity → HTTP Request**, method GET, URL as above, with `84` and `on`
   replaced by the magic text (the `...` button) for *Battery level* and *Charging wattage (W)*.
3. Samsung: Settings → Apps → MacroDroid → Battery → **Unrestricted**, or readings get delayed.

Test from a laptop: `curl "http://192.168.0.6:8765/battery?name=Test&level=50"` prints `ok`.

## Setup

1. On the TV: Settings → Device Preferences → About → press **Build** 7 times. Then in
   **Developer options** turn on **USB debugging** and **Wireless debugging**.
2. Connect: `adb connect 192.168.0.6:5555`, or pair with the code under Wireless debugging
   (`adb pair <ip:port> <code>`, then `adb connect <ip:port>`).
3. See what's safe to turn off: `sh candidates.sh 192.168.0.6:5555` (changes nothing).
4. Build and install the home screen:
   ```sh
   JAVA_HOME=/opt/homebrew/opt/openjdk@17 sh build.sh     # Java 21 breaks d8
   GOOGLE_TV=1 sh install.sh 192.168.0.6:5555 "<apps>"    # app order is in tv-cleanup-changes.txt;
                                                          # GOOGLE_TV=1 lets it turn Google TV's home off
   sh install.sh 192.168.0.6:5555 "" "bluetooth,sound,picture"   # optional shortcuts by the clock
   ```
5. Check nothing came back after a TV update: `sh check.sh 192.168.0.6:5555 --fix tv-cleanup-changes.txt`.
6. Test it end to end: `sh ci/smoke-test.sh 192.168.0.6:5555`.

`key.jks` signs the builds and is not in git. **Keep a backup**: without it, the next install has
to uninstall the home screen first, which resets its app order.

On the TV, hold **OK** on an app (or press **Menu**) to move, replace or remove it. The **Add app**
tile adds more; they wrap into rows of six.

## Undo

- Home screen: `sh undo.sh 192.168.0.6:5555` turns the TV's own home screen back on and removes this one.
- One app: `adb shell pm enable <package>`.
- Everything turned off:
  ```sh
  for p in $(adb shell pm list packages -d | tr -d '\r' | sed 's/package://'); do adb shell pm enable "$p"; done
  ```

## Troubleshooting

- **The laptop can't reach the TV at all** (no ping, adb times out): reboot the router. On 2026-10-19
  its 5 GHz band stopped passing traffic between devices until a reboot.
- **TV stuck restarting:** the laptop can usually still connect for a few seconds each boot. Run
  `adb connect`, `adb wait-for-device`, then the "everything turned off" loop above.

## License

MIT, see [LICENSE](LICENSE).
