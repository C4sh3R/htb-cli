<div align="center">
  <img src="https://cdn.simpleicons.org/hackthebox/9FEF00" width="250" alt="Hack The Box"/>
    <img src="https://readme-typing-svg.demolab.com?font=Russo+One&weight=2000&size=180&duration=1&pause=999999&color=000000&center=false&vCenter=true&repeat=false&width=380&height=218&lines=HTB" alt="HTB"/> 
<br><br>
  
&ensp;[<kbd> <br> Features <br> </kbd>](#features)&ensp;
&ensp;[<kbd> <br> Installation <br> </kbd>](#installation)&ensp;
&ensp;[<kbd> <br> Usage <br> </kbd>](#usage)&ensp;
&ensp;[<kbd> <br> Commands <br> </kbd>](#detailed-commands)&ensp;
&ensp;[<kbd> <br> Appearance <br> </kbd>](#appearance)&ensp;
&ensp;[<kbd> <br> Configuration <br> </kbd>](#configuration-variables)&ensp;
&ensp;[<kbd> <br> Contributions <br> </kbd>](#contributions)&ensp;
<br>
 <p>
    <img src="https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white"
         alt="Bash"/>
    <img src="https://img.shields.io/badge/Platform-Linux-1793D1?style=flat-square&logo=linux&logoColor=white"
         alt="Linux"/>
    <img src="https://img.shields.io/badge/API-Hack%20The%20Box-9FEF00?style=flat-square&logo=hackthebox&logoColor=black"
         alt="Hack The Box API"/>
    <img src="https://img.shields.io/badge/License-MIT-blue?style=flat-square"
         alt="MIT License"/>
  </p>
</div>

> A minimalist Bash CLI for HackTheBox using the **official API** (v4, with v5 for flag submissions). Built for speed: spawn machines, grab IPs, and submit flags — all without leaving the terminal.

```
htb spawn Connected      →  spawns the machine, waits for its IP, and copies it to the clipboard
htb own <hash>           →  submits the flag for the active machine automatically
htb season               →  lists season machines and their blood status
```

> [!NOTE]
> `htb-cli` uses the official Hack The Box API. Some operations depend on your account permissions, API availability, and the current machine state.

## Features

- **Only requires `curl` and `jq`** — no Python, no npm, no unusual dependencies
- Automatically resolves machine names to IDs (searches the season list → general list → free-text search)
- After spawning a machine, waits for its IP and copies it to the clipboard (`xclip`)
- Automatically detects the active machine for `own`, `stop`, `reset`, and `ip`
- **Auto-detected icons and colors**: uses Nerd Font glyphs if available, emoji otherwise, and plain ASCII when output is redirected to a file or `TERM` is not UTF-8
- OS icons (Windows/Linux/BSD/macOS), colored difficulty levels, and *own* / *blood* indicators
- `htb font` installs a Nerd Font without touching the package manager (works the same on Arch, Debian, Fedora, and macOS)
- The token is stored once in `~/.config/htb/token` or provided through an environment variable
- Integrated VPN management: multiple profiles (Labs, Fortress, Starting Point, etc.), downloads by product, connection selection menus, and seamless switching

## Requirements

| Tool | Purpose |
|-------------|----------|
| `bash` ≥ 4 | Shell |
| `curl` | API requests |
| `jq` | JSON parsing |
| `xclip` | Copies the IP to the clipboard after spawning a machine (optional) |
| `fontconfig` | Detects and installs Nerd Fonts (optional) |
| `unzip`, `bsdtar`, or `python3` | Extracts the font with `htb font` (any one of the three) |
| `openvpn` | VPN connections with `htb connect` (optional) |

## Installation

```bash
# 1. Clone the repository
git clone https://github.com/C4sh3R/htb-cli.git
cd htb-cli

# 2. Install
chmod +x htb
cp htb ~/.local/bin/htb        # or: sudo cp htb /usr/local/bin/htb

# 3. Save your API token
mkdir -p ~/.config/htb
echo "YOUR_TOKEN_HERE" > ~/.config/htb/token
chmod 600 ~/.config/htb/token
```

Get your token from **HackTheBox → Account Settings → API Token**.

> [!CAUTION]
> Keep your Hack The Box token private. Never commit it to the repository or share it in screenshots, logs, or public posts. If your token is exposed, revoke it and generate a new one.

### Alternative: Environment variable

```bash
export HTB_TOKEN="eyJ0eXAiOiJKV1..."
```

### VPN directory

HTB provides a separate `.ovpn` file for each product (Labs, Fortress, Starting Point, Release Arena, Pro Labs, etc.). `htb` stores them all in a single directory, allowing you to choose which one to connect to.

The directory is automatically detected from `~/Desktop/c4sh3r/HTB/vpn`, `~/HTB/vpn`, and `~/.config/htb/vpn` (the first directory that exists). To specify your own:

```bash
export HTB_VPN_DIR="/path/to/your/ovpn/files"
```

If you prefer the old behavior of using a single fixed file, `HTB_VPN_FILE` is still supported and skips the profile selector:

```bash
export HTB_VPN_FILE="/path/to/your/machines.ovpn"
```

## Usage

```
htb <command> [arguments]
```

### Quick reference

| Command | Description |
|---------|-------------|
| `htb whoami` | Displays your username and VIP status |
| `htb list [N]` | Lists the latest N machines (15 by default), newest first |
| `htb latest` | Shows the latest machine release |
| `htb season` | Lists machines in the active season and their blood status |
| `htb info <name\|id>` | Machine profile: owns, rank, and first bloods |
| `htb active` | Shows the currently spawned machine and its IP |
| `htb spawn <name\|id>` | Spawns a machine and waits for its IP |
| `htb ip [name\|id]` | Waits for and prints the active machine's IP |
| `htb host [dom1 dom2] ...` | Adds domains to `/etc/hosts` using the active machine's IP |
| `htb stop [name\|id]` | Stops the active machine or the specified one |
| `htb reset [name\|id]` | Resets the active machine or the specified one |
| `htb own <flag> [difficulty]` | Submits a flag for the **active** machine |
| `htb own <name\|id> <flag> [difficulty]` | Submits a flag for a specific machine |
| `htb vpn [product]` | Downloads a product's `.ovpn` file (`labs`, `sp`, `fortress`, etc.) |
| `htb vpns` | Lists local VPN profiles and marks the connected one |
| `htb connect [profile]` | Connects to the VPN (displays a menu if no profile is specified) |
| `htb disconnect [profile]` | Disconnects **only** the VPN started by `htb` (by PID) |
| `htb vpnstatus` | Shows the connected VPN and its profile |
| `htb icons` | Displays the detected icon set |
| `htb font [Font]` | Installs a Nerd Font (`Hack` by default) |
| `htb version` | Displays the `htb` version |

## Detailed commands

### `htb whoami`

Checks whether your token works and displays your account information.

```
$ htb whoami
User: C4sh3R  (id 12345)
VIP: true  Server: eu-vip-2
```

### `htb list [N]`

Lists the N most recently released machines. Shows whether you have obtained the user (`U:1`) or root (`R:1`) flag.

```
$ htb list 5
906  Connected  Linux   Easy    2025-11-30  U:1 R:0
905  VariaType  Linux   Medium  2025-11-23  U:0 R:0
904  Pupilpath  Windows Hard    2025-11-16  U:1 R:1
```

### `htb latest`

Shows the latest machine release. Useful for checking what has just been released.

```
$ htb latest
Latest release:
   ID:         906
   Name:       Connected
   OS:         Linux
   Difficulty: Easy
   Release:    2025-11-30T20:00:00.000000Z
   Active:     1
```

### `htb season`

Lists all machines in the active season, including their release dates and blood status.

```
$ htb season
906  Connected  Linux  Easy    rel:2025-11-30  released:true  Ublood:false  Rblood:false
905  VariaType  Linux  Medium  rel:2025-11-23  released:true  Ublood:true   Rblood:false
```

### `htb spawn <name|id>`

Spawns a machine by name or numeric ID. The script resolves the name in this order:

1. Season machine list (new releases appear here)
2. General paginated list (the 100 most recent machines)
3. HTB's free-text search endpoint

After spawning, it polls every 2 seconds until an IP address is assigned, prints it, and copies it to the clipboard.

```
$ htb spawn Connected
[*] Spawning machine ID 906 ...
[+] Machine deployed
[*] Waiting for IP...
[+] IP: 10.129.9.213
    (copied to clipboard)
```

> [!TIP]
> If you need to retrieve the IP later or spawned the machine from another terminal session, run `htb ip` instead of searching for the address manually in the Hack The Box interface.

### `htb ip`

Waits up to 80 seconds for the active machine to receive an IP address. Useful if you spawned the machine in another session.

```
$ htb ip
[+] IP: 10.129.9.213
    (copied to clipboard)
```

### `htb host <domain> <domain2> <domain3> ...`

Adds one or more domains to `/etc/hosts`, pointing them to the IP address of your active HackTheBox machine.

The command automatically retrieves the active machine's IP and updates `/etc/hosts` accordingly. It checks whether the domains or IP already exist to avoid unnecessary duplicate entries.

If an `#HTB` section exists, the entry is added there. Otherwise, the script creates the section automatically.

**Example:**

```bash
# Add a domain to /etc/hosts
htb host target.htb

# Add multiple domains to the same machine
htb host target.htb dev.target.htb admin.target.htb
```

### `htb active`

Shows which machine you currently have spawned.

```
$ htb active
Active: Connected (id 906)
IP: 10.129.9.213
Expires: 2025-12-01 08:00:00
```

### `htb own <flag> [difficulty]`

Submits a flag for the active machine. Difficulty is a number from 1 to 10 (5 by default).

```bash
# Submit a flag for the active machine (auto-detected)
htb own 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d

# Specify the difficulty
htb own 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d 3

# Submit a flag for a specific machine by name
htb own Connected 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d
```

Successful submission:

```
[+] Congratulations! You have got the user flag on Connected.
```

> [!IMPORTANT]
> Verify that the flag belongs to the intended machine before submitting it. If you omit the machine name or ID, the client uses the active machine when that behavior is supported.

### `htb reset`

Resets the active machine. Useful when something gets stuck during exploitation.

```
$ htb reset
[+] The machine has been scheduled for reset.
```

### `htb stop`

Stops the machine and frees up the VPN slot.

```
$ htb stop
[+] Machine stopped.
```

### `htb info <name|id>`

Displays a machine's full profile: operating system, difficulty, points, IP address, your owns and completion times, your rank on the machine, and **who claimed the first bloods**.

If a blood belongs to you, it is marked accordingly.

```
$ htb info Scaffold
╭──────────────────────────────────────────────────────────────────────╮
│   Scaffold                                                          │
╰──────────────────────────────────────────────────────────────────────╯
   ID            978
   OS            Windows
   Difficulty    ● Hard
   Points        40
   IP            (not spawned)
   Release       2026-09-05
   Active        ● yes

   User own        in 1H 2M 14S
   Root own        in 0H 41M 54S
   Your rank       #16

   User blood      admiin  0H 28M 7S
   Root blood      ahos6   0H 10M 46S
```

Each blood owner is compared against the authenticated user's ID, retrieved from `/user/info` at runtime. No user ID is hardcoded, so everyone sees their own bloods marked.

### `htb vpn [product]`

Downloads the `.ovpn` file for the specified product, using the server assigned to you by HTB. With no arguments, it lists the products and the server assigned to each one:

```
$ htb vpn
╭──────────────────────────────────────────╮
│ HackTheBox VPN Products                  │
╰──────────────────────────────────────────╯
   labs                EU Machines VIP+ 5        EU
   starting_point      EU StartingPoint 1        EU
   fortresses          EU Fortress 1             EU
   release_arena       no access
```

Specify a product to download its profile to your VPN directory:

```
$ htb vpn fortress
🚀 Querying servers for fortresses ...
🚀 Downloading EU Fortress 1 (id 429) ...
✅ VPN profile saved to ~/HTB/vpn/fortresses_eu-fort-1.ovpn  (edge-eu-fort-1.hackthebox.eu)
   Connect: htb connect fortresses_eu-fort-1
```

Accepted aliases: `labs`/`machines`/`vip`, `sp`/`starting`, `fortress`, `arena`/`release`, `season`.

**Pro Labs** are handled separately (each has its own VPN). They can be requested by name:

```
$ htb vpn prolab                 # without a name: list available Pro Labs
$ htb vpn prolab trusted         # downloads the VPN for the Trusted Pro Lab
🚀 Querying servers for Pro Lab 70 ...
🚀 Downloading EU Mini Pro Lab 1 (id 710) ...
✅ VPN profile saved to ~/HTB/vpn/eu-mini-prolab-1.ovpn  (edge-eu-mini-prolab-1.hackthebox.eu)
   Connect: htb connect eu-mini-prolab-1
```

You can also use the ID (`htb vpn prolab 70`) and `--pick`/`--tcp`, just like with other products.

| Option | Description |
|---|---|
| `-p`, `--pick` | Displays a menu to select a specific server instead of the assigned one |
| `--tcp` | Downloads the TCP variant |
| `-o <path>` | Saves the profile to a specific path |

The filename is derived from the server (`eu-fort-1`, `eu-dedivip-5`, etc.). If you already have an `.ovpn` file for that server, **the existing file is updated in place** instead of creating a duplicate.

### `htb vpns`

Lists the `.ovpn` files in your VPN directory, identifies each lab type, and marks the currently connected profile:

```
$ htb vpns
╭──────────────────────────────────────────╮
│ VPN Profiles   ~/HTB/vpn                 │
╰──────────────────────────────────────────╯
 ·  eu-starting-point-1-dhcp   Starting Point  edge-eu-starting-point-1-dhcp.hackthebox.eu
 ●  fortresses_eu-fort-1       Fortress        edge-eu-fort-1.hackthebox.eu   ✅ connected
 ·  htb                        Labs VIP+       edge-eu-dedivip-5.hackthebox.eu
```

### `htb connect [profile]`

Connects to the VPN using `openvpn --daemon` and waits up to 15 seconds for `tun0` to come up. Saves the process PID to `/run/htb-openvpn.pid` (`HTB_VPN_PID_FILE`) so that `htb disconnect` knows exactly which process it started.

You can specify the profile in several ways — all of these work:

```bash
htb connect fortress                  # product alias
htb connect fort                      # partial name
htb connect fortresses_eu-fort-1      # filename
htb connect ~/vpn/my-config.ovpn      # direct path
```

Without an argument, if you have more than one `.ovpn` file, a menu is displayed:

```
$ htb connect

   VPN profiles in ~/HTB/vpn

   1) Starting Point  edge-eu-starting-point-1-dhcp.hackthebox.eu
      eu-starting-point-1-dhcp.ovpn
   2) Fortress        edge-eu-fort-1.hackthebox.eu
      fortresses_eu-fort-1.ovpn
   3) Labs VIP+       edge-eu-dedivip-5.hackthebox.eu
      htb.ovpn

   Choose [1-3]: 2
🚀 Connecting fortresses_eu-fort-1.ovpn (sudo)
✅ VPN connected (Fortress): 10.10.15.177/23
```

Only **one HTB VPN connection** is managed at a time. If one is already connected and you request another, the script warns you and asks before switching (`-f` skips the prompt):

```
$ htb connect labs -f
⚠️  An htb VPN connection is already active: fortresses_eu-fort-1.ovpn
📌 Switching to htb.ovpn ...
🚀 Disconnecting fortresses_eu-fort-1.ovpn (pid 20999, sudo)
✅ VPN disconnected (process 20999 terminated).
🚀 Connecting htb.ovpn (sudo)
✅ VPN connected (Labs VIP+): 10.10.14.73/23
```

If the requested profile does not exist or the name is ambiguous, the script reports it without changing anything:

```
$ htb connect eu
❌ 'eu' matches multiple profiles:
   - eu-starting-point-1-dhcp.ovpn
   - fortresses_eu-fort-1.ovpn
```

If you save your sudo password in `~/.config/htb/sudo.pass`, the connection can run without interactive password prompts:

```bash
echo "your_sudo_password" > ~/.config/htb/sudo.pass
chmod 600 ~/.config/htb/sudo.pass
```

> [!WARNING]
> Switching VPN profiles may interrupt your current connection. Save your work and make sure you no longer need the existing VPN session before confirming a switch.

> [!CAUTION]
> The `sudo.pass` file contains a sensitive password. Never commit it to Git or share it. Use restrictive file permissions and avoid storing your password if you do not need this feature.

### `htb disconnect [file.ovpn]`

Disconnects **only** the VPN started by `htb`: reads the PID from `/run/htb-openvpn.pid`, checks that the process is still running and is actually an `openvpn` process, then terminates it. It never runs `pkill openvpn`, so other system VPN sessions (work, another lab, another `.ovpn` file) remain untouched. It uses `~/.config/htb/sudo.pass` if available to avoid prompting for a password.

```
$ htb disconnect
[*] Disconnecting htb VPN (pid 4821, sudo)...
[+] VPN disconnected (process 4821 terminated).
```

If there is no valid PID file (for example, a VPN started with an older version), the script searches for `openvpn` processes launched with an `.ovpn` file from your VPN directory (or the profile you specify). If it finds exactly one, it terminates it. If there are multiple processes, or if the running OpenVPN processes do not belong to `htb`, it leaves them untouched and lists them:

```
$ htb disconnect
[*] OpenVPN is running, but htb did not start any of its processes.
    Leaving other VPN connections untouched. Current processes:
3390 openvpn --config /home/user/work.ovpn --daemon
```

### `htb vpnstatus`

Checks whether `tun0` is active and which profile is being used.

```
$ htb vpnstatus
🌐  VPN connected  10.10.15.177/23
   profile: fortresses_eu-fort-1.ovpn (Fortress)
   htb OpenVPN process: pid 20999
```

```
$ htb vpnstatus
[+] VPN connected: 10.10.14.5/23
    (htb OpenVPN process: pid 4821)
```

---

## Typical workflow — First Blood

```bash
# 1. Connect to the VPN (once per session)
htb connect

# 2. Check the latest releases
htb latest
htb season

# 3. Spawn a machine
htb spawn Connected

# 4. ... hack away ...

# 5. Submit flags
htb own <hash_user>
htb own <hash_root>

# 6. Clean up
htb stop
htb disconnect   # disconnect from the VPN when finished
```

## Common usage examples

```bash
# Check whether the VPN is connected
htb vpnstatus

# Spawn by name or ID
htb spawn 906
htb spawn Connected

# Check the currently active machine
htb active

# Reset a machine if it gets stuck
htb reset

# Submit a flag with a low difficulty rating (it was easy)
htb own abc123... 2

# Submit a flag for a specific machine without making it the active one
htb own VariaType def456... 7

# List the 30 latest machines
htb list 30

# Look up information about any machine
htb info "Lame"
htb info 1
```

## Token security

The token is read in the following order:

1. The `$HTB_TOKEN` environment variable
2. The `~/.config/htb/token` file (or `$HTB_TOKEN_FILE`)

Make sure the file can only be read by your user:

```bash
chmod 600 ~/.config/htb/token
```

> [!IMPORTANT]
> Adding a file to `.gitignore` does not remove secrets that were already committed. If you accidentally publish a credential, revoke it and review the repository history.

## Appearance

`htb` automatically chooses how to render its output, in this order:

| Mode | When it is selected | Example |
|------|---------------------|---------|
| `nerd` | A Nerd Font is installed (`fc-list`) | ` `  ` ` |
| `emoji` | UTF-8 terminal without a Nerd Font | `🪟` `🐧` |
| `ascii` | No UTF-8 support, or output is redirected to a file/pipe | `WIN` `LNX` |

Column widths are calculated using the actual **visible** width (ignoring ANSI codes and counting emoji as two columns), keeping the tables aligned in all three modes.

```bash
htb icons                 # show the detected mode
HTB_ICONS=emoji htb list  # force a specific mode
NO_COLOR=1 htb list       # disable colors
```

### `htb font [Font]`

Installs a Nerd Font in the user's font directory and refreshes the font cache. It does not use `pacman`, `apt`, or `dnf`, so it works the same on any distribution:

```bash
htb font              # installs Hack Nerd Font
htb font JetBrainsMono
```

Downloads from the [ryanoasis/nerd-fonts releases](https://github.com/ryanoasis/nerd-fonts/releases) to `~/.local/share/fonts` (or `~/Library/Fonts` on macOS), extracts using `unzip`, `bsdtar`, or `python3` — whichever is available — and runs `fc-cache`. You then need to select the font in your terminal settings; the command prints the configuration line for kitty, alacritty, and wezterm.

## Configuration variables

| Variable | Default | Description |
|----------|---------|-------------|
| `HTB_TOKEN` | — | API token (environment variable override) |
| `HTB_TOKEN_FILE` | `~/.config/htb/token` | Path to the token file |
| `HTB_VPN_FILE` | *(hardcoded path)* | Default VPN configuration for `htb connect` |
| `HTB_VPN_PID_FILE` | `/run/htb-openvpn.pid` | PID of the `openvpn` process started by `htb` (used by `htb disconnect`) |
| `HTB_ICONS` | *(auto-detected)* | Forces the icon set: `nerd`, `emoji`, or `ascii` |
| `NO_COLOR` | — | If set, disables colors |
| `HTB_NF_VERSION` | `v3.4.0` | Nerd Fonts release downloaded by `htb font` |

## License
This project is distributed under the MIT License - use it however you like.

See [`LICENSE`](LICENSE) for the full license text.

## Contributions
 
Feel free to fork this repository and contribute improvements through pull requests. New content, write-ups, tools, and bug fixes are always welcome. If you find this project useful, consider giving it a ⭐ and sharing your feedback by opening an issue or submitting a pull request.

<div align="center">
  
![Typing SVG](https://readme-typing-svg.demolab.com?font=JetBrains+Mono&size=18&pause=1000&color=00FFB3&center=true&vCenter=true&width=500&lines=All+contributions+are+welcome!;Fork+it.+Hack+it.+PR+it.;Knowledge+shared+is+knowledge+multiplied.)

<sub>Made with ❤️ by <a href="https://c4sh3r.dev/">C4sh$r</a> · htb-cli © 2026</sub>
</div>