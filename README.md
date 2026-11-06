# mouse-desktop-plus

Control Windows virtual desktops with a mouse gesture. Hold the wheel button and drag left
or right to move to the neighbouring desktop, drag up to open Task View, or drag down to
show the desktop.

> **About this fork.** mouse-desktop-plus is a fork of
> [fpinero/mouse-desktop](https://github.com/fpinero/mouse-desktop).
> It retains the original horizontal desktop-switching gestures and adds two
> vertical gestures: drag up to open Task View and drag down to show the desktop.
>
> This fork has its own releases and installer. Follow the installation
> instructions below to get the version with all features. The original
> project's installation methods are documented separately for users who
> prefer the original, horizontal-only functionality.

No Administrator privileges, no runtime to install, no driver, no service. One 270 KB
executable that works with any mouse of any brand.

The original project has a product page at <https://mouse-desktop.nivelepsilon.com>, with a recording of
the gesture, the checksums to verify a download against, and, if a managed machine refuses
to run it, the request your IT department will want.

## Why this exists

Windows already changes virtual desktop with `Ctrl+Win+Left` and `Ctrl+Win+Right`, but
that means taking a hand off the mouse. Vendor software such as Logitech Options or Razer
Synapse can remap a mouse button to that shortcut, and it is genuinely pleasant to use.

The problem is that every one of those tools needs an Administrator account to install. On
a managed corporate laptop there is no such account, so the comfortable way of working
disappears along with it. This utility gives it back, using only what a standard user is
allowed to do.

Several free tools already do something similar. They were evaluated first, and why none
of them fitted is written up in [docs/prior-art.md](docs/prior-art.md).

## What it does

- Hold the wheel button and drag horizontally to move one desktop per 60 pixels.
- Keep dragging in the same direction to walk across several desktops in one motion.
- Hold the wheel button and drag **up** to open Task View (`Win+Tab`).
- Hold the wheel button and drag **down** to show the desktop (`Win+D`).
- Press and release the wheel button without dragging and the click is passed on intact,
  so middle click keeps closing browser tabs.

The first gesture recognised during a press decides what that press does. A vertical drag
fires once and then the press is finished, and a press that started switching desktops
ignores later vertical movement, so a single hold never mixes the two.

The trigger button, the distance, the direction of the horizontal gesture and the rest are
configurable.

## Requirements

- Windows 10 version 21H2 or later, or Windows 11.
- A mouse with a wheel button, or any of the other configurable triggers.
- Nothing else. The executable links only against DLLs that ship with Windows.

## Installing

Everything below works from an ordinary user account. No Administrator
privileges, additional runtimes, drivers or services are required.

### Step 1: check the machine will run it at all

On a managed machine, do this before anything else. It takes a minute and it is the
cheapest way to find out whether the rest is worth attempting.

The probe has to be an unsigned executable that has never been seen before, because that
is what this utility is and it is the property application control actually judges. Do not
copy a system tool such as `notepad.exe` and run the copy: that is a flagged technique in
its own right, blocked by a separate Attack Surface Reduction rule
(`c0033c00-d16d-4114-a5a0-dc9b3a7d2ceb`, "Block use of copied or impersonated system
tools"), so it fails on machines that would have run this utility perfectly well. The C#
compiler that ships in the box with the .NET Framework builds throwaway probes instead,
with nothing to install.

```powershell
$dir = "$env:LOCALAPPDATA\Programs\mouse-desktop"
New-Item -ItemType Directory -Force -Path $dir | Out-Null
$csc = "$env:WINDIR\Microsoft.NET\Framework64\v4.0.30319\csc.exe"
if (-not (Test-Path $csc)) { $csc = "$env:WINDIR\Microsoft.NET\Framework\v4.0.30319\csc.exe" }
Set-Content "$dir\probe.cs" 'class P{static void Main(){System.Console.WriteLine("probe ok");}}'
foreach ($i in 1..3) {
    & $csc /nologo /target:exe /out:"$dir\probe$i.exe" "$dir\probe.cs"
    & "$dir\probe$i.exe"
}
```

Read the outcome as a reliable no and an unreliable yes.

Three `probe ok` lines mean the machine probably runs unsigned programs from your profile,
so carry on. They are not a proof. The rule that does most of this blocking asks the
Defender cloud about a file it has never seen before, and when that lookup times out the
file goes through anyway. On the machine these instructions were tested against, one probe
in fifteen slipped past that way while the release executable was blocked every single
time. It is also why the snippet compiles three separate binaries instead of running one
of them three times: each compilation produces a different file, whereas a second run of
the same one only re-reads a verdict that has already been cached. What actually settles
the question is the next step, when you start the utility itself.

An "Access denied" on any of the three rounds is unambiguous in the other direction:
something is blocking unsigned executables, and this utility will be blocked the same way.
A message about your organisation's policy points at an application control policy. A bare
denial with no dialog, where reading the file fails as well as running it, points at
Attack Surface Reduction. Either way, read the troubleshooting section before going any
further.

Delete `probe.cs` and the `probe1.exe` to `probe3.exe` files when you are done.

### Step 2: download the release

Download the latest release of **mouse-desktop-plus** from the project's
[GitHub Releases](https://github.com/a-endari/mouse-desktop-plus/releases/latest) page.

You need two files from the same release:

1. **`mouse-desktop-x64.exe`** – the application executable.
2. **`install.ps1`** – the installation script.

Download both files into the same folder. For example, your Downloads folder is fine.

Use the `arm64` executable instead if you have a Windows ARM device. The x64
version also works on ARM devices through emulation.

> **Important:** Make sure you download both files from the releases of
> `mouse-desktop-plus`, not from the original `fpinero/mouse-desktop` repository.
> The original application does not include the Task View and Show Desktop gestures.

### Step 3: install

Open PowerShell in the folder containing the downloaded files and run:

```powershell
.\install.ps1 -Source .\mouse-desktop-x64.exe
```

The installer automatically:

- Creates `%LOCALAPPDATA%\Programs\mouse-desktop` if it does not exist.
- Copies the executable into that directory.
- Creates a Windows startup entry under `HKEY_CURRENT_USER`.
- Starts the application.

No Administrator privileges are required.

If PowerShell refuses to run the script because of the execution policy, allow
scripts for the current session only:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Then run the installer command again.

You can also use `-NoAutostart` to install without creating a startup entry,
or `-NoStart` to install without immediately launching the application.

The first time the executable runs, Windows SmartScreen may display a blue
"Windows protected your PC" panel because the executable is not signed.
Choose **More info**, then **Run anyway**.

### Step 4: check it works

1. An icon of two blue arrows appears in the notification area. On Windows 11 it may be
   hidden behind the chevron next to the clock; drag it onto the taskbar to keep it in
   sight.
2. Create a second virtual desktop with `Ctrl+Win+D`.
3. Hold the wheel button and drag about 60 pixels to the left. The desktop should change.
4. Hold the wheel button and drag about 60 pixels up. Task View should open.
5. Press `Esc` to close it, then hold the wheel button and drag about 60 pixels down. The
   desktop should be shown.
6. Middle click a browser tab. It should close, which confirms that a click without a drag
   is still passed through.
7. Right click the tray icon and confirm "Start with Windows" is ticked.
8. Sign out and back in. The icon should come back on its own.

### Portable, with no installation at all

Put `mouse-desktop.exe` anywhere, including a memory stick, and run it. To keep the
configuration beside the executable rather than in your profile, create an empty
`config.toml` in the same folder. The utility fills it in on the next start and reads it
from there, so nothing is left behind on the machine.

### Updating

Download the new executable and installer script from the latest
[GitHub Release](https://github.com/a-endari/mouse-desktop-plus/releases/latest).

Run the installer again with the new executable:

```powershell
.\install.ps1 -Source .\mouse-desktop-x64.exe
```

The installer stops the running copy before replacing it, so there is nothing else
to do. Your configuration file is not touched.

A new release is a new file, and a managed machine judges it from scratch. A laptop
that runs the current version without complaint can still block the next one for a
while, as described under "Blocked by Attack Surface Reduction" below.

### Removing

The uninstall script is included in the repository:

```powershell
.\uninstall.ps1
```

This stops the utility, removes the startup entry and deletes the installation folder.
Add `-RemoveConfig` to delete the configuration too.

If you downloaded only the release assets, you can obtain the uninstall script
from the repository's `scripts` directory.

### Using the original mouse-desktop instead

<details>
<summary><strong>Install the original horizontal-only version (without Task View and Show Desktop)</strong></summary>

If you prefer the original application, you can install it using the methods
provided by its author. This version supports horizontal mouse gestures for
switching between virtual desktops, but does not include the two vertical
gestures added by this fork.

**Option 1: winget**

If your organisation has not disabled winget:

```powershell
winget install --id fpinero.mouse-desktop
```

By default it installs into your user profile and puts a `mouse-desktop`
command on the path.

**Option 2: Scoop**

If you already use Scoop:

```powershell
scoop bucket add fpinero https://github.com/fpinero/scoop-bucket
scoop install mouse-desktop
```

**Option 3: download from the original project's releases**

Visit the original project's
[GitHub Releases](https://github.com/fpinero/mouse-desktop/releases)
and download the appropriate executable.

**Starting the application**

Package managers handle downloading and installing the executable, but
they do not automatically start it or configure Windows startup.

Start it once with:

```powershell
mouse-desktop
```

Afterwards, right-click its notification-area icon and enable
"Start with Windows".

**Updating**

For winget:

```powershell
winget upgrade --id fpinero.mouse-desktop
```

For Scoop:

```powershell
scoop update mouse-desktop
```

**Removing**

For winget:

```powershell
winget uninstall --id fpinero.mouse-desktop
```

For Scoop:

```powershell
scoop uninstall mouse-desktop
```

Neither package manager knows about the startup entry, which lives under
`HKEY_CURRENT_USER`, so it may leave an entry pointing at a file that no
longer exists. Disable "Start with Windows" from the tray menu before
uninstalling.

The configuration is stored in `%APPDATA%\mouse-desktop`. Delete that
folder manually if you want to remove it as well.

For further information, documentation and troubleshooting, visit the
[original project website](https://mouse-desktop.nivelepsilon.com).

</details>

### Troubleshooting

| What you see | What it means | What to do |
| --- | --- | --- |
| "This app has been blocked by your system administrator" | AppLocker or App Control is blocking user-writable folders | Ask IT to allow the file by hash, or to name a folder you may run programs from, and install there |
| SmartScreen panel on first run | The executable is unsigned | "More info", then "Run anyway". Building it yourself avoids the question |
| The script will not run | PowerShell execution policy | `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` |
| No icon in the notification area | It may simply be hidden | Click the chevron next to the clock and drag the icon onto the taskbar |
| The gesture works everywhere except one window | That window belongs to an elevated process | Expected, see the known limitations |
| Nothing happens at all | Another copy may already be running | Check with `Get-Process mouse-desktop`, and start it with `--verbose` to get a log next to the configuration file |
| A bare "Access denied", with no dialog, and even reading the file fails | A Defender Attack Surface Reduction rule is blocking it on reputation | See "Blocked by Attack Surface Reduction" below |

#### Blocked by Attack Surface Reduction

Attack Surface Reduction is not AppLocker and it does not announce itself. Rule
`01443614-cd74-433a-b99e-2ecdc07bfc25`, "Block executable files from running unless they
meet a prevalence, age, or trusted list criterion", blocks executables that are unsigned
and too new to have a reputation, which is exactly what a fresh build of this utility is.

It is easy to misread, because it gives a bare "Access denied" with no dialog and it denies
plain reads as well as execution, so even `Get-FileHash` fails and the symptom looks like
a file permission problem. It is not. The rule follows the file and not the folder, so
another directory, a portable copy and a renamed file all behave the same, while a signed
and prevalent executable in that very folder runs normally.

`Get-MpPreference` will not show it either: a standard user cannot read the policy merged
from Intune, and the rule list comes back empty. The event log is the reliable check.

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-Windows Defender/Operational' -MaxEvents 50 |
    Where-Object Id -eq 1121
```

Event 1121 is a block, 1122 is audit only. The message carries the rule id and the path.

There is no way around this from a user account, and looking for one is the wrong move on a
managed machine. What does work, cheapest first: wait, because age is one of the three
criteria and a fresh build fails it; submit the file to Microsoft through the Defender
Security Intelligence submission portal, which is the intended route for a reputation block
and needs no local privileges; ask IT for an Attack Surface Reduction exclusion by hash; or
sign the release with an Authenticode certificate, which is the only fix that also travels
to other managed machines.

Waiting works, but not on the timetable the documentation suggests. Microsoft describes
the age criterion in terms of hours. On the machine these instructions were tested
against, the release binary was refused every single time for the next forty four hours,
ninety blocks in the event log, and then the very same file, byte for byte, ran without
a complaint two weeks later. Nobody tried in between, so the day it turned is unknown.
Budget ten days rather than two, and do not read a block on the second day as a permanent
verdict.

The clock is not yours to start. Age is counted by the Defender cloud from the first time
it saw that hash anywhere in its telemetry, not from the moment the file reached your
Downloads folder, so deleting it and downloading it again advances nothing, and neither
does renaming it or moving it. The flip side is convenient: once a file has cleared it
stays cleared wherever it goes, so copying it into the install folder afterwards is safe.
Every new release is a different hash and starts again from zero.

The two remedies are hard to tell apart afterwards. In the case above the file was
submitted through the portal on the first day, so there is no way to know from the outside
whether it finally passed on age or on the review. Submitting early costs nothing and at
least guarantees the cloud has seen the file at all.

## Configuration

The file lives at `%APPDATA%\mouse-desktop\config.toml` and is created with comments on
the first run. Open it from the tray menu, edit it, then choose "Reload configuration".

| Setting | Default | Meaning |
| --- | --- | --- |
| `trigger` | `"middle"` | Button to hold: `middle`, `right`, `x1` or `x2` |
| `threshold_px` | `60` | Distance that triggers a gesture: one desktop horizontally, Task View or Show Desktop vertically |
| `repeat_cooldown_ms` | `250` | Shortest delay between two desktop switches. Does not apply to the vertical gestures, which fire once per press |
| `max_vertical_ratio` | `1.0` | How far from horizontal a drag may stray and still count as a desktop switch. Does not affect the vertical gestures |
| `invert` | `false` | Swap the direction of desktop switching. Up and down are not affected |
| `replay_click_when_no_gesture` | `true` | Pass the click on when there was no drag |
| `enabled` | `true` | Start with gesture recognition switched on |

`x1` and `x2` are the two side buttons found on most mice, usually mapped to back and
forward. Choosing one of them as the trigger makes the gesture feel closest to a vendor
button remap.

## Known limitations

**Elevated windows.** While the window in front belongs to a process running as
Administrator, such as an elevated terminal, Task Manager started as administrator, or the
UAC prompt itself, the gesture does nothing. Windows deliberately hides input from
lower-privileged processes through User Interface Privilege Isolation, and there is no way
around it that does not require the Administrator account this utility exists to avoid.

**Vertical gestures.** A vertical drag is recognised when it travels at least
`threshold_px` and is longer vertically than horizontally. It fires once per press, so
holding the button and dragging further does nothing more. `Win+D` toggles, so dragging
down a second time on an already shown desktop restores the windows. There is currently no
setting to switch the vertical gestures off or to change which direction does what.

**No wrap around.** On the first or the last desktop, dragging further does nothing. That
is how `Ctrl+Win+Arrow` behaves too.

**Unsigned executable.** The binary carries no code signing certificate, so Windows
SmartScreen warns the first time it runs, and a managed laptop with AppLocker or App
Control policies may refuse to run it from a user-writable folder at all. If that happens,
the options are to have IT allow it by hash or to place it in a folder the policy permits.
Building it yourself from this repository is the surest way to know what you are running.

**Corporate policy.** Nothing here bypasses a policy. It only avoids needing rights a
standard user does not have.

## How it works

A global low level mouse hook, `SetWindowsHookEx(WH_MOUSE_LL, ...)`, watches for the
trigger button being held. When the pointer has travelled far enough horizontally, the
utility synthesises `Ctrl+Win+Left` or `Ctrl+Win+Right` with `SendInput`. When it has
travelled far enough vertically, it synthesises `Win+Tab` for an upward drag or `Win+D` for
a downward one. None of these calls needs any privileges.

Switching could also be done through `IVirtualDesktopManagerInternal`, a COM interface
that changes desktops directly. This project does not use it: Microsoft changes its GUIDs
with nearly every major Windows update, which silently breaks callers several times a
year. Synthesising the documented shortcut is slower by a few milliseconds and survives
Windows updates.

The trigger button is hidden from other applications while it is held, so a drag cannot
also open a context menu or start an autoscroll. If the button is released without a drag,
a real click is injected at the pointer, which is what keeps ordinary clicking with that
button working.

## Building and testing

```powershell
cargo build --release
cargo test                          # gesture, configuration and icon logic
cargo clippy --all-targets -- -D warnings
cargo test -- --ignored --nocapture # end to end, needs an interactive session
```

The end to end test starts the real executable, synthesises a wheel drag, and checks
through the documented `IVirtualDesktopManager` interface that Windows actually changed
desktop. It creates two scratch desktops for the measurement and closes both afterwards.
It takes over the mouse pointer for about five seconds, so do not run it while you are
using the machine.

Stop any installed copy before running it. The utility refuses to start twice, so the copy
the test launches would put up a dialog instead of installing its hook. The test checks
for this and says so rather than reporting a broken gesture.

Manual check, worth doing after any change to the hook:

1. Create three virtual desktops with `Ctrl+Win+D`.
2. Start the utility and confirm the icon in the notification area.
3. Hold the wheel button and drag right over the desktop, over Explorer and over a
   browser. Each should move one desktop forward.
4. Keep dragging without releasing. It should keep moving forward.
5. Middle click a browser tab. It should close, which proves the click was passed on.
6. Hold the wheel button and drag up. Task View should open once, and dragging further
   without releasing should do nothing more.
7. Close Task View, then hold the wheel button and drag down. The desktop should be shown
   once.
8. Drag right until the desktop changes, then without releasing drag up. Nothing should
   happen, because the press already chose the horizontal gesture.
9. Release the wheel button after a vertical drag. No middle click should reach the window
   underneath.

## License

MIT. See [LICENSE](LICENSE).
