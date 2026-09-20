<h1 align="center">getsmt</h1>
<p align="center">The one-line installer for Schooi's Multitool</p>
<p align="center">
	<img src="https://img.shields.io/github/languages/top/SchooiCodes/getsmt" alt="GitHub top language">
	<img src="https://img.shields.io/github/commit-activity/w/SchooiCodes/getsmt" alt="GitHub commit activity">
	<img src="https://img.shields.io/github/languages/code-size/SchooiCodes/getsmt" alt="GitHub code size in bytes">
	<img src="https://img.shields.io/github/license/SchooiCodes/getsmt" alt="GitHub License">
	<a href=https://smt.gleeze.com><img src="https://img.shields.io/badge/installer-smt.gleeze.com-purple" alt="smt.gleeze.com"></a>
</p>

About
-
This repo hosts the bootstrapper for [Schooi's Multitool (SMT)](https://github.com/SchooiCodes/smt) - the script served at `smt.gleeze.com` that downloads and runs the SMT setup in one line.

The entire installer is a single PowerShell script stored as `index.html`, so `irm` can fetch and `iex` can execute it directly with no manual downloads.

How It Works
-
1. Checks for 64-bit Windows 10/11 and exits early on unsupported systems
2. Self-elevates to Administrator (re-launching itself, preserving flags like `-Silent`)
3. Prepares a temp workspace at `$env:TEMP\SMT` and a log at `$env:TEMP\SMT_Log.txt`
4. Checks Defender Tamper Protection status:
   - Enabled -> opens Windows Security so you can add an exclusion manually, then continues anyway
   - Disabled -> adds a temporary exclusion for the setup exe and verifies it
   - Unknown (third-party AV) -> continues without an exclusion
5. Downloads `Schooi's Multitool Setup.exe` from the [smt repo](https://github.com/SchooiCodes/smt/raw/main/Schooi%27s%20Multitool%20Setup.exe) with 3 retries and a file-size sanity check
6. Runs the installer, verifies its exit code
7. Always cleans up - removes the AV exclusion and deletes the temp folder - even on failure

The AV exclusion step never blocks the install. If it fails or can't be verified, the script logs why and proceeds anyway.

Compatibility
-
| Requirement | Minimum | Recommended |
| ----------- | ------- | ----------- |
| Windows Version | 10+ 64-bit | 11 64-bit |
| Privileges | Administrator | Administrator |
| Powershell Version | 5.1 | 5.1+ |
| Network Availability | Connected | Connected |
| Disk Space | ~300kB for the bootstrapper + SMT size | size may vary based on usage |

Installation
-
### Recommended Method
Open Powershell as an Administrator and run:
```ps1
irm "https://smt.gleeze.com/" | iex
```

### Mirrors
If the main URL is down, use any of these (same script):
```ps1
irm "http://smt.farted.net/" | iex
```
```ps1
irm "http://getsmt.ftp.sh/" | iex
```
```ps1
irm "http://getsmt.us.to" | iex
```
([VirusTotal scan](https://www.virustotal.com/gui/file/77553494de93dfe8dec7986109f1cd93675d77d81969e1a8dabd3289b5f500a8)[^1] - [script contents](https://smt.gleeze.com/))

Usage
-
### Silent Install
Skip the manual Tamper Protection prompt and run unattended:
```ps1
irm "https://smt.gleeze.com/" | iex; Install-SMT -Silent
```
Or if saved locally as `.ps1`:
```ps1
.\index.html -Silent
```
> Note: when piped via `irm | iex`, switches can't be passed directly through the pipe. The script preserves bound parameters on self-elevation, so `-Silent` survives the admin relaunch.

| Parameter | Description |
| --------- | ----------- |
| `-Silent` | Skips the manual-exclusion pause when Tamper Protection is on and continues without an exclusion |

### Logs & Temp Files
| Path | Purpose |
| ---- | ------- |
| `$env:TEMP\SMT_Log.txt` | Timestamped log of every step |
| `$env:TEMP\SMT\SMTSetup.exe` | Downloaded installer (deleted after run) |
| `$env:TEMP\SMT\SkipMSGBox` | Marker file used by setup |

### Repo Layout
| File | Purpose |
| ---- | ------- |
| `index.html` | The bootstrapper PowerShell script (served as the site) |
| `CNAME` | Custom domain (`smt.gleeze.com`) for GitHub Pages |

Related
-
- Main tool: [SchooiCodes/smt](https://github.com/SchooiCodes/smt) - over **130** command line tools, installers, and utilities
- SMT website: [smt.xubi.org](https://smt.xubi.org)
- Full feature list: [schooicodes.github.io/smtweb/features](https://schooicodes.github.io/smtweb/features/index.html)

Contributing
-
Contributions are welcome! If you find a bug in the bootstrapper (elevation, download retries, AV handling, cleanup) please open an issue or submit a pull request.

License
-
getsmt uses the MIT license, find more [here](https://github.com/SchooiCodes/getsmt/blob/main/LICENSE). Same license as [SMT](https://github.com/SchooiCodes/smt/blob/main/LICENSE).

[^1]: The .exe installer made in NSIS gets flagged by many AVs, including Windows Defender, due to it downloading SMT's files. Many pieces of malware download .bat files, and are called "droppers", so AVs immediately flag anything that does the same. For that reason, this bootstrapper uses a temporary, auto-removed exclusion - or no exclusion at all if Tamper Protection is on.
