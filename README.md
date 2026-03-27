# Vestaboard Simulator

A full-screen, browser-based Vestaboard split-flap display simulator. Designed to run on a TV via an Amazon Fire Stick, Raspberry Pi, or any device with a web browser.

## Live URL

**https://vestaboardhtml.vercel.app**

## How It Works

- Single `index.html` file with all CSS and JS inline — no frameworks, no build step
- `config.json` controls messages, symbols, rotation speed, background color, and sound
- Hosted on Vercel via GitHub — push changes to `main` and Vercel auto-deploys
- The app re-fetches `config.json` periodically (default: every 60 minutes) so the TV picks up new messages without a manual refresh

## Changing Messages

Edit `config.json`, commit, and push to GitHub. The live board updates automatically.

```json
{
  "messages": [
    "GOOD MORNING {heart}",
    "THE BEST IS YET TO COME",
    "RYAN {heart} BUNKER FAMILY"
  ]
}
```

No physical access to the TV is required to change messages. Just push to GitHub.

## Config Options

| Setting | Values | Description |
|---|---|---|
| `mode` | `"rotate"` / `"static"` | Cycle through messages or show only the first |
| `rotation_interval_seconds` | Number | Seconds between message changes |
| `config_refresh_minutes` | Number | How often the app re-fetches config (0 to disable) |
| `background` | `"black"` / `"white"` | Background color mode (default: black) |
| `sound_enabled` | `true` / `false` | Toggle flip sound |
| `sound_volume` | `0.0` - `1.0` | Sound volume |
| `symbols` | Array | Custom symbols with shortcode, display character, optional color |

## Symbols

Define symbols in the `symbols` array. Use them in messages with `{shortcode}`.

```json
"symbols": [
  { "shortcode": "heart", "display": "❤", "color": "#FF0000" },
  { "shortcode": "star", "display": "★", "color": "#FFD700" },
  { "shortcode": "moon", "display": "☽" }
]
```

If `color` is omitted, the symbol renders in the default letter color.

---

## Display Options: Getting the App on Your TV

The Samsung Frame TV's built-in browser is not suitable for this — it can't run full-screen without browser chrome and has an outdated rendering engine. You need an external device plugged into an HDMI port. Here are three options, ranked from easiest to most robust.

---

### Option 1: Amazon Fire TV Stick + ClickSimply Kiosk (Easiest)

**Setup time:** ~5 minutes. No code, no sideloading, no terminal commands.

**What you need:**
- Amazon Fire TV Stick (any model, ~$30-40)
- HDMI port on the Samsung Frame

**Setup:**

1. Plug the Fire TV Stick into the Frame's HDMI port and complete initial Fire TV setup (Wi-Fi, Amazon account).
2. On the Fire TV, go to the **Amazon Appstore** and search for **"ClickSimply Kiosk"** (free, no account required).
3. Open ClickSimply Kiosk and enter your Vercel URL: `https://vestaboardhtml.vercel.app`
4. The app displays your URL full-screen with no browser chrome. The Fire TV will not go to sleep while the app is open.

**Tip:** Entering the URL is much easier using the **Amazon Fire TV phone app** (free, iOS/Android) — it gives you a real keyboard instead of the on-screen remote keyboard.

**Pros:**
- Simplest possible setup — install an app, type a URL, done
- No sideloading, no terminal, no SSH
- Fire TV stays awake automatically while the app is running

**Cons:**
- **No auto-launch on boot.** If the Fire Stick loses power (power outage, TV unplugged), you need to manually reopen the ClickSimply app with the remote.
- Changing the URL requires clearing the app cache and re-entering it.
- The Fire TV Stick's browser engine is slightly older — CSS animations may be less smooth than on a Pi.

**Best for:** Testing the app quickly, or setups where someone is occasionally near the TV to recover from power outages.

---

### Option 2: Amazon Fire TV Stick + Fully Kiosk Browser (Medium)

**Setup time:** ~20 minutes. Requires sideloading an APK (downloading an app file outside the Appstore).

**What you need:**
- Amazon Fire TV Stick (any model, ~$30-40)
- HDMI port on the Samsung Frame
- A phone or computer on the same Wi-Fi to help with sideloading

**Setup:**

1. Plug the Fire TV Stick into the Frame's HDMI port and complete initial Fire TV setup.
2. On the Fire TV, go to **Settings > My Fire TV > Developer Options** and enable **"Apps from Unknown Sources"** (also called "Install unknown apps"). If you don't see Developer Options, go to Settings > My Fire TV > About > click the device name 7 times rapidly to unlock it.
3. Install the **Downloader** app from the Amazon Appstore (free).
4. Open Downloader, enter this URL: `https://www.fully-kiosk.com/en/#download` and download the **Fully Kiosk Browser APK for Fire OS**.
5. Install the APK when prompted.
6. Open Fully Kiosk Browser and enter your Vercel URL as the Start URL.
7. In Fully Kiosk settings, enable:
   - **Kiosk Mode** (locks the device to the browser)
   - **Launch on Boot** (auto-starts after power outages)
   - **Keep Screen On**

**Pros:**
- **Auto-launches on boot** — survives power outages without intervention
- Remote admin panel lets you change the URL from your phone/laptop
- Full kiosk lockdown (no accidental navigation away)
- Prevents screen sleep

**Cons:**
- Requires sideloading (not hard, but more steps than the Appstore)
- Fire OS compatibility is noted as "may have issues" by the Fully Kiosk developer — test before relying on it
- Paid license (~$7.90 one-time per device) to remove a small watermark
- The Fire TV Stick's browser engine is still slightly older than a Pi's Chromium

**Best for:** Unattended operation where you want auto-recovery from power outages without a Raspberry Pi.

---

### Option 3: Raspberry Pi (Most Robust)

**Setup time:** 30-60 minutes. Requires running terminal commands (copy-paste from this guide).

**What you need:**
- Raspberry Pi 4 or Pi 5 (~$50-75 with case, power supply, and microSD card)
- HDMI cable connected to the Samsung Frame
- Wi-Fi or Ethernet
- A keyboard and monitor for initial setup (can be removed after)

**Why choose this:** The Pi runs a full, modern Chromium browser — the same engine used on desktop computers. CSS animations (the tile flips) will be smoothest here. It auto-boots to your URL, survives power outages, and requires zero ongoing maintenance.

**Setup:**

All commands below are copy-paste. You'll type them into the Pi's terminal (or SSH in from your computer).

#### Step 1: Update the Pi

```bash
sudo apt update && sudo apt upgrade -y
```
*This updates all software on the Pi to the latest versions.*

#### Step 2: Install required packages

```bash
sudo apt install -y chromium-browser xserver-xorg x11-xserver-utils xinit openbox unclutter
```
*This installs: the Chromium web browser, a minimal display system (no full desktop), and a tool to hide the mouse cursor.*

#### Step 3: Configure auto-login

```bash
sudo raspi-config
```
*This opens a settings menu. Navigate to:* **System Options > Boot / Auto Login > Console Autologin**

*This makes the Pi boot directly to a command line, logged in as your user, with no desktop.*

#### Step 4: Create the kiosk startup script

```bash
nano ~/.xinitrc
```
*This opens a text editor. Paste the following (replace `YOUR_VERCEL_URL` with your actual Vercel deployment URL):*

```bash
#!/bin/bash

# Disable screen blanking and power management
xset s off
xset s noblank
xset -dpms

# Hide the mouse cursor
unclutter -idle 0.1 -root &

# Launch Chromium in kiosk mode
chromium-browser \
  --noerrdialogs \
  --disable-infobars \
  --kiosk \
  --disable-session-crashed-bubble \
  --disable-features=TranslateUI \
  --check-for-update-interval=31536000 \
  --disable-component-update \
  --overscroll-history-navigation=0 \
  --disable-pinch \
  --no-first-run \
  --autoplay-policy=no-user-gesture-required \
  'YOUR_VERCEL_URL'
```

*Save and exit: press `Ctrl+X`, then `Y`, then `Enter`.*

Make it executable:

```bash
chmod +x ~/.xinitrc
```

#### Step 5: Auto-start the display on boot

```bash
echo '[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && startx -- -nocursor' >> ~/.bash_profile
```
*This adds one line to your login file that automatically launches the browser when the Pi boots.*

#### Step 6: Reboot and verify

```bash
sudo reboot
```

*The Pi should boot > auto-login > launch Chromium full-screen > display the Vestaboard. No keyboard or mouse needed from this point forward.*

#### Optional: Daily auto-reboot for freshness

```bash
sudo crontab -e
```
*Choose the nano editor if prompted. Add this line at the bottom:*

```
0 4 * * * /sbin/reboot
```

*This reboots the Pi at 4:00 AM daily to clear any stale memory. Save and exit (`Ctrl+X`, `Y`, `Enter`).*

#### Optional: Hourly hard refresh

```bash
sudo apt install -y xdotool
crontab -e
```

*Add this line:*

```
0 * * * * DISPLAY=:0 xdotool key F5
```

*This sends a full page refresh every hour (in addition to the app's soft config re-fetch).*

#### Troubleshooting

| Problem | Fix |
|---|---|
| Black screen on boot | Check HDMI cable. Run `sudo raspi-config` > Display Options > Resolution |
| Chromium crash loop | Add `--disable-gpu` to the chromium-browser command in `.xinitrc` |
| No audio from TV | Run `sudo raspi-config` > System Options > Audio > select HDMI |
| Mouse cursor visible | Verify `unclutter` is installed and the line is in `.xinitrc` |
| Screen goes blank after inactivity | Verify the three `xset` lines are in `.xinitrc` |

**Pros:**
- Best animation performance (full modern Chromium)
- Auto-boots to your URL — fully unattended, survives power outages
- No third-party app dependencies
- Complete control over everything

**Cons:**
- Most expensive hardware (~$50-75)
- Most setup time (~30-60 min of terminal commands)
- Troubleshooting requires terminal access

**Best for:** The final, permanent installation where you want the smoothest experience and zero ongoing maintenance.

---

### Quick Comparison

| | Fire Stick + ClickSimply | Fire Stick + Fully Kiosk | Raspberry Pi |
|---|---|---|---|
| **Setup time** | ~5 min | ~20 min | ~30-60 min |
| **Cost** | ~$30-40 | ~$38-48 | ~$50-75 |
| **Technical skill** | None | Low (sideloading) | Medium (terminal) |
| **Auto-boot after power loss** | No | Yes | Yes |
| **Animation smoothness** | Good | Good | Best |
| **Ongoing maintenance** | Minimal | Minimal | None |
| **Remote URL change** | Clear app cache | Fully Kiosk admin panel | Edit Vercel URL in config |

> **Recommendation:** Start with **Option 1** (Fire Stick + ClickSimply) to test the app on your Frame quickly. If you want unattended operation, upgrade to **Option 2** or **Option 3** later. The app itself is identical regardless of which display device you use — it's just a URL.

---

## Repository

- **GitHub:** https://github.com/bunkerryan-source/vestaboardhtml.git
- **Vercel:** https://vestaboardhtml.vercel.app
