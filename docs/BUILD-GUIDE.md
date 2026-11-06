# AetherConsole build guide

> [!CAUTION]
> **Read this first. This is the fastest way to void your Maestro's warranty
> and lose support from FlexRadio.** You'll be opening the case, adding a
> drive, changing the BIOS and running software FlexRadio doesn't provide or
> support. FlexRadio is not involved in AetherConsole and isn't obliged to
> help with a Maestro that has been modified.
>
> **The inside of the Maestro is unforgiving.** The wires and connectors
> inside the panel are very short and under tension, the internal speaker's
> wires most of all. The display isn't glued in and shifts easily, and the
> power button falls out of place once the panel is opened. The slightest
> mishandling while the case is open can leave the unit beyond repair.
>
> **Don't start unless you're willing to accept every risk.** That includes a
> Maestro that no longer works and repairs no one will cover. Go in as if you
> just lit $1,500 on fire and are hoping for the best. AetherConsole comes with
> no warranty of any kind (see [`LICENSE`](../LICENSE)), and you follow this
> guide entirely at your own risk.

## Before you start

**What you're building.** AetherConsole turns a FlexRadio Maestro "C" into a
touch-screen station that starts straight into AetherSDR. Its knobs, buttons
and lights work the radio. Nothing is taken away: the Maestro's original
software stays on its own built-in drive, and every time it starts, a screen
asks which one you want, **AetherSDR** or **SmartSDR**.

**How it works.** You add a second drive, an NVMe stick, to the Maestro. On
your own computer you write the ready-made AetherConsole image onto that drive
through a USB adapter, then fit the drive inside the Maestro. The only time you
need a keyboard on the Maestro is one BIOS setting. Everything after that
happens on the Maestro's touch screen.

**How to use this guide.** Do the steps in order. Each step tells you what's
happening in plain words, says what success looks like, and, where it helps,
ends with a **prompt** you can paste into an AI assistant if you'd like one to
walk you through it. An assistant is optional.

**Time:** allow an hour or two. Most of it is the download and writing the
drive.

## What you need

| Item | Notes |
|---|---|
| FlexRadio Maestro, model "C" | The x86 version, built on a LattePanda 3 Delta board. Other Maestro models won't work. |
| An M.2 2280 NVMe SSD, M-key | 128 GB or more is plenty. Any PCIe NVMe drive works; a Gen4 drive simply runs at the board's slower speed. Get one **without a heatsink**, which won't fit inside the case. [Amazon search](https://www.amazon.com/s?k=M.2+2280+NVMe+SSD+256GB+no+heatsink) |
| A USB enclosure for M.2 NVMe drives | Holds the drive while you write the image to it. Make sure the listing says **NVMe** ("NVMe" or "NVMe/SATA"); many M.2 enclosures are SATA-only and won't take an NVMe drive. [Amazon search](https://www.amazon.com/s?k=M.2+NVMe+USB+enclosure) |
| A computer to write the image from | Windows, macOS or Linux, with [balenaEtcher](https://etcher.balena.io/) (any of the three) or [Rufus](https://rufus.ie/) (Windows). |
| A wireless keyboard with a **small (nano) USB dongle** | The Maestro's USB ports can only be reached with the case open, and a normal plug or cable won't fit. The dongle goes in while the case is open (step 3) and stays in, since removing it means opening the case again. |
| A network: Ethernet with a router that hands out addresses (DHCP), or Wi-Fi | The first-start wizard sets it up on the touch screen. |
| Your FlexRadio radio, on the same network | AetherSDR connects to it. |

## How it fits together

- **The new NVMe drive** gets the AetherConsole image: Arch Linux, AetherSDR,
  the panel software and the start-up chooser.
- **The Maestro's built-in drive (eMMC)** keeps its original Windows and
  SmartSDR software, untouched. At the first start the console copies one
  small folder from it, the Windows boot manager, read-only, so the start-up
  screen can launch Windows.
- **The boot loader** (systemd-boot) on the NVMe starts Linux by default.
  Linux logs in by itself as a kiosk user and shows the start-up chooser.
- **The panel daemon** links the Maestro's knobs, buttons and lights to the
  radio. It is installed and turned on in the image.

## 0. Ground rules for your AI

If you use an AI assistant, paste this into it first:

```text
You're helping me install AetherConsole on a FlexRadio Maestro "C" (LattePanda
3 Delta), following the AetherConsole build guide, which I've given you. I'm not
an expert, so explain each step in one or two plain sentences, and tell me what
success looks like. Rules for the whole job:
- Before anything writes to a disk, have me show you the list of disks and
  wait for me to confirm which one is the NVMe in the USB enclosure. Never
  write to any other disk.
- Never ask me to paste a password into this chat.
- Never read, copy, search, decompile or modify FlexRadio software
  (SmartSDR, SmartSDR for Maestro, the Maestro's firmware).
- If something doesn't match the guide, stop and ask me.
```

## 1. Download the image and check it

**What's happening:** you download the image file and confirm it arrived
intact.

1. Open the [latest release](https://github.com/aethersdr/maestro-packages/releases/latest)
   and download both files: `maestro-console-<version>.img.xz` and
   `maestro-console-<version>.img.xz.sha256`. Don't unpack the `.xz`; the
   writing tools read it as it is.
2. Compare the file's SHA-256 with the one in the `.sha256` file (it's also
   on the release page). In a terminal, in the download folder:

| Your computer | Command |
|---|---|
| Windows | `certutil -hashfile maestro-console-<version>.img.xz SHA256` |
| macOS | `shasum -a 256 maestro-console-<version>.img.xz` |
| Linux | `sha256sum -c maestro-console-<version>.img.xz.sha256` |

**Check:** the long hexadecimal number printed matches the one in the
`.sha256` file exactly (Linux prints `OK`). If it doesn't, download again.

> **Prompt:** "Do step 1 of the guide with me: help me check the SHA-256 of
> the image I downloaded against its .sha256 file."

## 2. Write the image to the NVMe drive

**What's happening:** the image is written onto the new drive in the USB
enclosure. **Everything on that drive is erased.**

1. Put the NVMe in the USB enclosure and plug it into your computer.
2. Open balenaEtcher (or Rufus), choose the `.img.xz` file, and choose the
   enclosure as the target. **Get this right:** pick it by its size and model,
   and if more than one drive is listed, unplug every other USB drive first.
3. Write it, and let the tool verify the result.
4. Eject the enclosure and unplug it. If your computer offers to "format" the
   drive afterwards, say no: it simply can't read the drive's Linux partition.

The image is only a few gigabytes; at the first start the console grows it to
fill the whole drive.

**Check:** the tool reports the write and its verification as successful.

> **Prompt:** "Do step 2 of the guide with me: I'm about to write the image
> with balenaEtcher. Help me make sure the target is the NVMe in the USB
> enclosure, by its size and model, before I press Flash."

## 3. Fit the drive and set the BIOS

**What's happening:** the drive goes inside the Maestro, and you tell the
Maestro's BIOS to start from it. This is the only step that needs the
keyboard.

> [!WARNING]
> **Handle the inside with great care.** The slightest mishandling here can
> leave the Maestro beyond repair.
> - **The wires and connectors are very short and under tension**, the
>   **internal speaker's wires most of all**. Open the case slowly, keep the
>   halves close together, and never pull, twist or lift one half far enough
>   to put weight on a cable.
> - **The display isn't glued in.** It's held only by the assembled case and
>   moves around easily once the panel is open. Support it and keep it
>   level; don't let it slide or tip.
> - **The power button is held in only by the closed case.** It will fall out
>   of its spot when the panel opens. Note how it sits before you open the
>   case, catch it, and set it aside somewhere safe.

1. Shut the Maestro down and unplug its power. Remove the battery pack if one
   is fitted.
2. Open the case gently, keeping the halves close together so no cable is
   stretched, especially the speaker wires. Keep the display supported, and
   set the power button aside.
3. Fit the NVMe into the LattePanda board's M.2 M-key slot, and secure it with
   its screw. Work on a static-safe surface and keep track of every screw.
4. **Plug the wireless keyboard's dongle into a free USB port now.** The ports
   can only be reached with the case open. Leave the dongle in; you'll use the
   keyboard for the BIOS in a moment, and later to pick a boot entry if you
   ever need to.
5. **Before closing, check three things:** the power button is back in its
   spot, the display sits square in its opening, and no wire, the speaker's
   especially, is pinched or pulled tight. Then close the case slowly and
   evenly, and check that the power button presses and springs back.
6. Connect Ethernet (if you're using it) and power.
7. Power on and tap **Delete** on the wireless keyboard to open the BIOS
   setup. On LattePanda boards,
   **F7** opens a one-time boot menu instead.
8. In the boot settings, make the NVMe the first boot option. It may be
   listed as "UEFI OS" or by the drive's model. Make sure Secure Boot is
   off. Save and exit, usually with F4 or F10.

**Check:** the Maestro shows the AetherSDR splash.

> **Prompt (only if it doesn't start):** "After the BIOS change my Maestro
> shows <what you see>. Help me check, using the F7 boot menu, whether it sees
> the NVMe."

## 4. The first start

**What's happening:** the first time it starts, the console finishes setting
itself up, then a short wizard asks you a few questions on the touch screen.

On its own, before the screen comes up, the console:

- grows its system partition to fill the drive;
- registers its boot loader with the Maestro's BIOS, and keeps it first after
  every start from then on;
- copies the Windows boot manager from the built-in drive, read-only, so the
  chooser's **SmartSDR** button can start the original software.

If a start is cut short, it finishes what's left the next time.

Then the **setup wizard** comes up: the network (Ethernet is found by itself;
for Wi-Fi, pick your network and type its password on the on-screen keyboard),
your time zone, and, optionally, Tailscale (step 7). After the wizard you see
the start-up chooser. It waits for a tap; it never picks on its own.

The console names itself `maestro-xxxx` on your network, where `xxxx` is the
last four hex digits of its Ethernet MAC address.

**Check:** the chooser shows **AetherSDR** and **SmartSDR**.

## 5. Test it

1. Tap **AetherSDR**. The first time, AetherSDR asks which radio to use. Pick
   yours on the touch screen; it remembers it from then on.
2. Quit AetherSDR. You return to the chooser, not a desktop.
3. Tap **SmartSDR**. The Maestro restarts into its original software. Restart
   it from there, and it comes back to the chooser.

**The clock.** Windows keeps the Maestro's hardware clock in local time, while
Linux expects universal time. After using Windows, the console's clock can be
hours off until network time corrects it, which happens automatically within a
minute or so.

## 6. The panel: knobs, buttons and lights

**What's happening:** the panel daemon is already running. It reads the
Maestro's knobs and buttons and drives its lights, talking to your radio
alongside AetherSDR.

**There is no radio to set.** You pick the radio in AetherSDR, as usual, and
the panel follows the radio AetherSDR is connected to: on your LAN, or routed
over Tailscale or another network. It pairs with this console's AetherSDR and
no other.

**Check:** with AetherSDR running, turn the tuning knob and the frequency
moves; the button lights follow the radio, and the power button's light
changes from white to the radio's status color.

**The panel transmits like the radio's own controls.** MOX, TUNE, ATU, the
paddle and the PTT jack key the radio as soon as AetherSDR is connected; there
is no separate software switch. You decide what goes out the same way you do
from AetherSDR: break-in on or off for CW, and the TX slice (the TX badge on
the VFO flag) for voice. Connect an antenna or dummy load before you key.

## 7. Optional: Tailscale

**What's happening:** Tailscale puts the console on your private Tailscale
network, so it can reach a radio that isn't on the same network, without
opening ports on your router.

Tap the network badge on the chooser, then **Tailscale**, and scan the QR code
with your phone. Approve the device in your Tailscale account. Tailscale uses
the console's own name, `maestro-xxxx`.

## Looking after it

| Task | How |
|---|---|
| Update the system | On the console: the gear on the chooser, then **Update now** (and **Restart now** when it asks). This updates Linux and the console's own software. |
| Update AetherSDR | The update above does not change AetherSDR: the image carries the AetherSDR release that was current when the image was built. A newer image brings a newer AetherSDR. |
| Change Wi-Fi | The chooser's Wi-Fi button opens the network setup any time. With no network at start-up, the console opens it before the chooser. |
| Start over | The gear on the chooser, then **Reset to defaults**. It forgets the saved networks, the time zone, Tailscale and the console's own settings, then restarts into the setup wizard. AetherSDR keeps its settings. |

## If something goes wrong

| Problem | What to do |
|---|---|
| Linux won't start | Hold the space bar on the wireless keyboard at power-on and pick `arch` or `arch-test` from the menu. |
| Nothing starts from the NVMe | Press F7 at power-on and pick the Windows Boot Manager on the built-in drive; the Maestro starts as it originally did. Then check the BIOS boot order (step 3). |
| The SmartSDR button doesn't start Windows, but F7 → Windows Boot Manager does | Windows may have updated its boot files since the console copied them. Use F7 → Windows Boot Manager to start SmartSDR. |
| Clock hours off after using Windows | Wait a minute for network time to correct it. |
| Panel knobs do nothing | AetherSDR must be running and connected to a radio. |

The boot menu (space bar at power-on):

| Entry | What it starts |
|---|---|
| `arch-rt` (default) | Linux with the real-time kernel and the splash |
| `arch` | Linux with the standard kernel (fallback) |
| `arch-test` | Linux with the standard kernel and no splash, for troubleshooting |
| `windows` | the Maestro's original Windows |
