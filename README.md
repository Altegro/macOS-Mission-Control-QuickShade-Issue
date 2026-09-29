# macOS Mission Control missing thumbnails — troubleshooting guide

A practical troubleshooting guide for a specific Mission Control symptom:

> **Windows are open, but their previews disappear from the Desktop/Space thumbnails at the top of Mission Control.**

The goal of this repository is to help people check **third-party software and other common causes before assuming macOS itself is broken, resetting system preferences, or reinstalling macOS**.

QuickShade is the first **confirmed and fully reproducible cause** documented here, but this guide is intentionally broader: if you do not use QuickShade, continue with the troubleshooting steps below.

## Confirmed cause #1 — QuickShade


https://github.com/user-attachments/assets/8f2ae79a-3c6f-48b5-9d96-6d6ca5c8905d


### TL;DR — check QuickShade first

On the tested system, the issue is **fully reproducible** with QuickShade:

- **macOS:** 27.0.1 (26A434), installed through the normal public Software Update channel
- **QuickShade:** 3.0.1 (15), installed from the Mac App Store
- **Trigger:** QuickShade → **Enable Shade = ON**
- **Result:** windows disappear from the Mission Control Desktop/Space thumbnails
- **Disable Enable Shade:** thumbnails return immediately
- **Quit QuickShade:** thumbnails return immediately
- No reboot, logout, Dock reset, or macOS reinstall is required

### Fastest fix

1. Open **QuickShade**.
2. Turn **Enable Shade** **OFF**.
3. Open Mission Control again.

If the thumbnails immediately come back, you have reproduced the same conflict.

You can also quit QuickShade from Terminal:

```bash
killall QuickShade
```

To verify that it is no longer running:

```bash
pgrep -fl QuickShade
```

If there is no output, QuickShade is no longer running.

If you do not need QuickShade, disable its auto-start or uninstall it.

> **Before resetting Mission Control, Dock, Spaces, WindowManager, WindowServer, display settings, or reinstalling macOS: check QuickShade first.**

---

## Minimal reproduction

This was reproduced repeatedly on the configuration above.

1. Install/open QuickShade 3.0.1 (15) from the Mac App Store.
2. Open Mission Control and confirm the Desktop/Space thumbnails contain window previews.
3. In QuickShade, enable **Enable Shade**.
4. Open Mission Control again.
5. The windows disappear from the Desktop/Space thumbnails.
6. Turn **Enable Shade** off.
7. Open Mission Control again.
8. The thumbnails immediately work again.

You can also reproduce the application-level test:

```bash
open -a QuickShade
```

Enable the shade, reproduce the bug, then:

```bash
killall QuickShade
```

The thumbnails should return immediately.

This repository started from a reproducible QuickShade compatibility issue observed on one tested Mac, then expanded into a broader Mission Control troubleshooting guide. It is not an official Apple or QuickShade bug tracker, and behavior may differ on other hardware or macOS builds.

---

# General Mission Control troubleshooting

If you do **not** have QuickShade, or disabling it does not fix the problem, continue here.

These are the diagnostic steps used to isolate the confirmed QuickShade case, but most of them are useful for **other user-specific Mission Control thumbnail problems as well**.

The main principle is simple:

> **Rule out third-party software and user-session differences before resetting macOS preferences.**

**Do not start by deleting preferences.** Work from the least destructive checks to the most invasive ones.

## 1. Restart Dock and WindowManager

Mission Control state can sometimes get stuck.

```bash
killall Dock
```

If that does not help:

```bash
killall WindowManager
```

A brief screen flash or UI restart is normal.

In the confirmed QuickShade case, neither of these fixed the problem while QuickShade shading was active.

---

## 2. Test without an external display

Disconnect external displays and test Mission Control on the built-in display.

If you use multiple displays, you can also temporarily test:

**System Settings → Desktop & Dock → Mission Control → Displays have separate Spaces**

Turn it off, log out/in if macOS requests it, and test again.

This was not the root cause in the confirmed QuickShade case, but it can help separate a multi-display Spaces problem from a general Mission Control problem.

---

## 3. Test Apple's default display scaling

If you use a custom HiDPI resolution, BetterDisplay, SwitchResX, Lunar, DisplayBuddy, or similar software:

1. Quit the display utility completely.
2. Use **System Settings → Displays → Default**.
3. Test Mission Control again.

On the tested Mac, the default logical resolution was 1512 × 982 and the physical Retina display was 3024 × 1964.

The original investigation also tested a non-default scaled setup. It was **not** the cause of this QuickShade issue.

---

## 4. Temporarily test 60 Hz instead of ProMotion

If your Mac normally uses ProMotion:

**System Settings → Displays → Refresh Rate → 60 Hz**

Then test Mission Control again.

This did not resolve the confirmed QuickShade case, but it is a low-risk way to rule out a display refresh/layout interaction.

---

## 5. Test a new macOS user account

Create a temporary new user:

**System Settings → Users & Groups → Add User**

Log in to the new account, create two or three Desktops/Spaces, open a few windows, then open Mission Control.

Interpretation:

- **Works in the new user:** the problem is probably user-specific — preferences, login items, background software, accessibility tools, or per-user state.
- **Fails in the new user too:** the problem is more likely system-wide.

In the confirmed case, Mission Control worked correctly in a new user account.

---

## 6. Test your normal account in Safe Mode

This was the key diagnostic step in this case.

On Apple silicon:

1. Shut the Mac down completely.
2. Press and hold the power button until **Loading startup options** appears.
3. Select the startup disk.
4. Hold **Shift**.
5. Choose **Continue in Safe Mode**.
6. Log in to the same user account that normally has the problem.
7. Test Mission Control before manually launching other apps.

Interpretation:

- **Works in Safe Mode:** strongly suspect third-party software, background agents, login items, accessibility tools, remote-desktop tools, display tools, or window-management utilities.
- **Still broken in Safe Mode:** investigate per-user preferences/state or a deeper macOS problem.

In the confirmed QuickShade case, Mission Control worked normally in Safe Mode.

---

## 7. Do not trust only the visible Login Items list

Holding Shift during login or disabling visible Login Items is useful, but it does **not** rule out every background component.

macOS can still have:

- LaunchAgents
- LaunchDaemons
- Service Management background items
- system extensions
- helper tools
- accessibility-enabled utilities

Useful inventory commands:

### User LaunchAgents

```bash
find ~/Library/LaunchAgents -maxdepth 1 -type f -print 2>/dev/null | sort
```

### System LaunchAgents

```bash
find /Library/LaunchAgents -maxdepth 1 -type f -print 2>/dev/null | sort
```

### System LaunchDaemons

```bash
find /Library/LaunchDaemons -maxdepth 1 -type f -print 2>/dev/null | sort
```

### Background Task Management database

```bash
sfltool dumpbtm
```

### System extensions

```bash
systemextensionsctl list
```

### Running processes

```bash
ps -axo pid=,comm= | sort -k2
```

### Loaded user launch services

```bash
launchctl list
```

Pay special attention to software that interacts with:

- display brightness or overlays
- display scaling
- Mission Control
- Accessibility APIs
- window snapping/management
- mouse/trackpad gestures
- screen recording
- remote desktop
- menu bar/window composition

Examples include display dimming tools, BetterDisplay/Lunar/DisplayLink-type software, Rectangle/Magnet/Moom/AltTab/Hammerspoon-style window tools, remote-desktop software, input remappers, and similar utilities.

Do not assume a listed application is guilty merely because it is installed. Prefer an **ON/OFF reproduction test** like the QuickShade test above.

---

## 8. Stop suspicious applications one at a time

Do not disable twenty things at once if you want to identify the real cause.

For an application named `ExampleApp`:

```bash
killall ExampleApp
```

Immediately test Mission Control.

If the problem disappears, relaunch the app:

```bash
open -a ExampleApp
```

If the problem comes back, you have a much stronger reproduction.

That is exactly how this QuickShade conflict was isolated.

---

# Preference resets — last resort only

The following steps were tested during the investigation, but **they did not fix the confirmed QuickShade case** while QuickShade shading was enabled.

Only use them if QuickShade is not your cause and you have already completed the safer tests above.

## Important: back up first

Never delete preference domains without exporting/copying the current values first.

### Back up Spaces

```bash
defaults export com.apple.spaces ~/Desktop/com.apple.spaces-backup.plist
```

### Back up Dock and WindowManager

```bash
mkdir -p ~/Desktop/mission-control-backup

defaults export com.apple.dock   ~/Desktop/mission-control-backup/com.apple.dock.plist

defaults export com.apple.WindowManager   ~/Desktop/mission-control-backup/com.apple.WindowManager.plist

mkdir -p ~/Desktop/mission-control-backup/ByHost

cp ~/Library/Preferences/ByHost/com.apple.dock*.plist   ~/Desktop/mission-control-backup/ByHost/ 2>/dev/null || true
```

### Back up per-user WindowServer display preferences

```bash
mkdir -p ~/Desktop/windowserver-backup

cp ~/Library/Preferences/ByHost/com.apple.windowserver*.plist   ~/Desktop/windowserver-backup/ 2>/dev/null || true
```

Verify that the backup files actually exist before resetting anything.

---

## Reset Spaces

After creating the backup:

```bash
defaults delete com.apple.spaces
killall cfprefsd
```

Then log out and log back in.

Possible side effects include losing/recreating virtual Desktops and application-to-Space assignments.

This did not fix the confirmed QuickShade case.

---

## Reset Dock / Mission Control / WindowManager preferences

After backups:

```bash
defaults delete com.apple.dock
defaults delete com.apple.WindowManager
killall cfprefsd
```

Then log out and log back in.

This can reset Dock layout/preferences, hot corners, and Mission Control-related settings.

This also did not fix the confirmed QuickShade case.

---

## Reset per-user WindowServer display preferences

Again: back these files up first.

Move the files out of the active preference directory instead of permanently deleting them:

```bash
mkdir -p ~/Desktop/windowserver-backup

find ~/Library/Preferences/ByHost   -maxdepth 1   -name 'com.apple.windowserver*.plist'   -exec mv {} ~/Desktop/windowserver-backup/ \;

killall cfprefsd
```

Then restart the Mac.

This did not fix the confirmed QuickShade case either.

---

# Restoring settings after unnecessary troubleshooting

If you already reset these preferences while searching for the cause, restore **only from backups you created before the reset**.

Before restoring, make one more backup of the current state so the operation is reversible.

Example:

```bash
STAMP="$(date +%Y%m%d-%H%M%S)"
SAFETY="$HOME/Desktop/pre-restore-$STAMP"

mkdir -p "$SAFETY/ByHost"

defaults export com.apple.spaces   "$SAFETY/com.apple.spaces.plist" 2>/dev/null || true

defaults export com.apple.dock   "$SAFETY/com.apple.dock.plist" 2>/dev/null || true

defaults export com.apple.WindowManager   "$SAFETY/com.apple.WindowManager.plist" 2>/dev/null || true

cp -p "$HOME"/Library/Preferences/ByHost/com.apple.dock*.plist   "$SAFETY/ByHost/" 2>/dev/null || true

cp -p "$HOME"/Library/Preferences/ByHost/com.apple.windowserver*.plist   "$SAFETY/ByHost/" 2>/dev/null || true
```

Then import the backup domains you actually have:

```bash
defaults import com.apple.spaces   "$HOME/Desktop/com.apple.spaces-backup.plist"

defaults import com.apple.dock   "$HOME/Desktop/mission-control-backup/com.apple.dock.plist"

defaults import com.apple.WindowManager   "$HOME/Desktop/mission-control-backup/com.apple.WindowManager.plist"
```

Restore any matching ByHost files you previously backed up, then:

```bash
killall cfprefsd
```

Restart the Mac normally.

Do **not** blindly copy another person's ByHost files or UUID-specific preference files. They are machine/user specific.

---

# Confirmed result in the original tested case

The long troubleshooting path was useful for isolation, but the final cause was simple:

```text
QuickShade 3.0.1 (15)
        |
        +-- Enable Shade ON
        |       -> Mission Control Space thumbnails lose window previews
        |
        +-- Enable Shade OFF
                -> previews immediately return
```

So if you found this page because Mission Control suddenly looks broken:

**Check QuickShade before changing macOS settings.**

---

## Share confirmations or other reproducible causes

Issues are welcome primarily to **document affected configurations and reproducible causes**, not as a promise of individual technical support.

If you can reproduce the QuickShade behavior, feel free to open an issue with:

- Mac model / Apple silicon or Intel
- macOS version and build
- QuickShade version
- whether QuickShade came from the Mac App Store
- whether **Enable Shade ON** causes the problem
- whether **Enable Shade OFF** immediately fixes it
- whether `killall QuickShade` fixes it
- whether Safe Mode works normally
- whether an external display is connected

If you have the **same Mission Control thumbnail symptom but do not use QuickShade**, an issue is also useful if you have found another reproducible cause. Please describe the smallest ON/OFF or start/stop test that makes the problem disappear and return.

This helps build a list of confirmed configurations and causes for other users.

> This repository is **not a support desk**, and individual troubleshooting or a solution for every configuration is not guaranteed.

---

## Disclaimer

This is an independent Mission Control troubleshooting repository built from a reproducible user investigation. QuickShade is one confirmed cause documented here, not an assumption that every similar Mission Control problem is caused by QuickShade. It is not affiliated with Apple or the QuickShade developer.

Preference-reset commands can change your macOS user configuration. Use backups and understand the commands before running them.
