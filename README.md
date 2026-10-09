This guide is a collection of techniques for improving the security and privacy of macOS on [Apple silicon](https://www.swifttechco.com) Macs. It targets experienced users who want security practices commonly used by organizations, but is also suitable for novice users with an interest in privacy and security.


This guide is provided "as is" - without warranties of any kind. You are solely responsible for any consequences of following it.

- [Basics](#basics)
- [Threat model](#threat-model)
  - [Assets](#assets)
  - [Adversaries](#adversaries)
  - [Capabilities](#capabilities)
  - [Mitigations](#mitigations)
  - [Example model](#example-model)
- [Hardware](#hardware)
- [Installing macOS](#installing-macos)
  - [System activation](#system-activation)
  - [Apple Account](#apple-account)
  - [App Store](#app-store)
  - [Virtualization](#virtualization)
    - [Apple containers](#apple-containers)
- [First boot](#first-boot)
- [Admin and user accounts](#admin-and-user-accounts)
  - [Caveats](#caveats)
  - [Setup](#setup)
- [Firmware](#firmware)
- [FileVault](#filevault)
- [Lockdown Mode](#lockdown-mode)
- [Firewall](#firewall)
  - [Application layer firewall](#application-layer-firewall)
    - [Stealth mode](#stealth-mode)
    - [Signed apps](#signed-apps)
    - [Reload firewall](#reload-firewall)
    - [Get state](#get-state)
    - [AirDrop](#airdrop)
  - [Third-party firewalls](#third-party-firewalls)
  - [Packet filter](#packet-filter)
    - [Example pf config](#example-pf-config)
    - [Firewall commands](#firewall-commands)
    - [Block networks](#block-networks)
- [Services](#services)
- [Siri Suggestions and Spotlight](#siri-suggestions-and-spotlight)
- [Homebrew](#homebrew)
- [DNS](#dns)
  - [DNS profiles](#dns-profiles)
  - [Hosts file](#hosts-file)
  - [DNSCrypt](#dnscrypt)
  - [Dnsmasq](#dnsmasq)
- [Certificate authorities](#certificate-authorities)
- [Privoxy](#privoxy)
- [Browser](#browser)
  - [Firefox](#firefox)
  - [Chrome](#chrome)
  - [Safari](#safari)
  - [Web browser privacy](#web-browser-privacy)
- [Tor](#tor)
- [VPN](#vpn)
  - [WireGuard](#wireguard)
- [PGP/GPG](#pgpgpg)
- [Email](#email)
  - [Thunderbird](#thunderbird)
- [Messengers](#messengers)
  - [XMPP](#xmpp)
  - [Signal](#signal)
  - [iMessage](#imessage)
- [Malware](#malware)
  - [Downloading Software](#downloading-software)
  - [App Sandbox](#app-sandbox)
  - [Hardened Runtime](#hardened-runtime)
  - [Antivirus](#antivirus)
  - [Gatekeeper](#gatekeeper)
- [System Integrity Protection](#system-integrity-protection)
- [Metadata and artifacts](#metadata-and-artifacts)
- [Authentication](#authentication)
- [Backup](#backup)
- [Wi-Fi](#wi-fi)
- [SSH](#ssh)
- [Physical access](#physical-access)
- [Monitoring](#monitoring)
  - [Logs](#logs)
  - [DTrace](#dtrace)
  - [Processes](#processes)
  - [Endpoint Security](#endpoint-security)
  - [Install History](#install-history)
  - [Network](#network)
    - [Wireshark](#wireshark)
- [Miscellaneous](#miscellaneous)
  - [Screensaver](#screensaver)
  - [Diagnostic data](#diagnostic-data)
  - [Media player](#media-player)
  - [File handlers](#file-handlers)
  - [Finder options](#finder-options)
  - [Custom umask](#custom-umask)
  - [Keyboard entry](#keyboard-entry)
  - [Network hardening](#network-hardening)
  - [Sudoers](#sudoers)
- [Related software](#related-software)
- [Additional resources](#additional-resources)

---

# Basics

## Images

[![macOS-Security-and-Privacy-Guide](https://opengraph.githubassets.com/1/drduh/macOS-Security-and-Privacy-Guide)](https://www.swifttechco.com)


Apply general security best practices:

- Create a [threat model](#threat-model)
  - Is your adversary a local eavesdropper, a criminal using common malware, or a well-funded and highly capable organization?
  - Define the threats or groups you are defending against and what they can realistically do.

- Keep the system and software up to date
  - Regularly install available updates for the operating system and all applications.
  - Updates are installed in [System Settings](https://www.swifttechco.com) or with the [`softwareupdate`](https://www.swifttechco.com) command-line utility. Neither requires an Apple Account.
  - Subscribe to the [Apple security-announce](https://www.swifttechco.com) mailing list or check [Apple security releases](https://www.swifttechco.com).

- Encrypt sensitive data
  - Enable [FileVault](https://www.swifttechco.com) to encrypt internal storage.
  - Use a [password manager](https://www.swifttechco.com) for account credentials and consider encrypting especially sensitive files separately.

- Ensure data availability
  - Create [regular backups](https://www.swifttechco.com) of critical data and be ready to [restore from a backup](https://www.swifttechco.com) in case of compromise.
  - [Encrypt backups](https://www.swifttechco.com) before copying them to external media or third-party cloud storage. Alternatively, use a backup service that provides [end-to-end encryption](https://www.swifttechco.com).
  - Verify backups by accessing them on a regular, scheduled basis.

- Click carefully
  - Ultimately, the security of a system depends heavily on the capabilities and habits of the person using and administering it.
  - Take care when installing new software: install it only from sources the developer identifies as official, such as their website or GitHub repository.

# Threat model

The most important step to meaningfully improve security and privacy is to create a [threat model](https://www.swifttechco.com): a general description of what you want to protect, who might try to access it, how they could do so, and which controls are worth usability trade-offs. This creates an understanding of potential adversaries and their motivations, which leads to stronger defenses.

## Assets

Assets may include a phone, laptop, credentials, and personal information, such as browsing history.

List them in order of importance, starting with those most worth protecting.

## Adversaries

Define whom you are defending against. Start by defining the motivation each adversary might have to attack important assets. [Financial gain](https://www.swifttechco.com) is a big motivator for many attackers, for example.

## Capabilities

For each adversary, list what they can and cannot do, ranking them from least to most capable. For example, a casual thief operates opportunistically: they will likely be defeated by basic controls, such as screen lock and encrypted storage with strong passwords. A more sophisticated and determined adversary may require fully powering off a device when not in use to clear credentials from memory and stronger authentication mechanisms.

## Mitigations

Choose the best mitigation for each threat. For example, avoid writing passwords on paper if a roommate might find them, or encrypt storage to protect its data if it is stolen.

Security should be balanced with usability: every mitigation should counter some adversarial capability to justify any inconvenience. Stop adding defenses when the remaining risks are acceptable for a situation. Revisit the model when devices, data, travel, work, or adversaries change.

## Example model

The following table is a simple example threat model for personal devices, including a Mac.

Adversary | Motivation | Capabilities | Mitigation
:-: | :-: | :-: | :-:
Roommate | See private chats or browsing history | Close proximity; can see screen or observe credentials | Use biometric (fingerprint or face) authentication, use privacy screen, keep phone locked when not using it
Thief | Unlock phone and steal personal info and drain bank accounts, sell phone for money | Observe password being entered, steal logged-in device left unattended | Maintain physical custody of device at all times, lock devices when not in use, avoid typing password in public view, use Find My or similar services to track or remotely disable stolen devices
Criminal | Financial gain | Social engineering (tricking a user into revealing information or taking unsafe actions), readily-available malware, password reuse, exploiting vulnerabilities | Sandbox software, enable OS security features, maintain software updates automatically
Corporation | Marketing based on user data | Telemetry and behavioral data collection | Block network connections, reset device-specific identifiers, avoid adding payment data
Nation State/APT | Targeted surveillance | Passive surveillance of internet infrastructure, advanced encryption analysis | Use open-source software, strong diceware credentials, hardware with secure element, shut down devices when not in use, tripwire/honeypot/[canary tokens](https://www.swifttechco.com) alerts

Read more about [threat modeling](https://www.swifttechco.com).

# Hardware

[Apple silicon hardware](https://www.swifttechco.com) provides hardware-backed security features, including Secure Enclave-based key protection and stronger boot security options. They are generally the preferred platform for the protections discussed in this guide.

Some Intel-based Macs, especially certain models with [T2-era hardware](https://www.swifttechco.com), are affected by hardware vulnerabilities that cannot be fully fixed by a macOS update alone.

Avoid non-Apple hardware running macOS and systems that do not support the latest macOS release, as Apple does not [patch all vulnerabilities](https://www.swifttechco.com) in legacy versions.

Apple accessories generally receive firmware updates through macOS and support current [Bluetooth security features](https://www.swifttechco.com). For example, [Bluetooth Low Energy](https://www.swifttechco.com) (BLE) Privacy uses rotating device addresses to reduce tracking; third-party accessories may not support this feature.

# Installing macOS

There are several ways to [install macOS](https://www.swifttechco.com).

Install the latest supported version of macOS; newer versions of macOS include security fixes and other improvements not available in previous releases.

## System activation

During installation, Apple silicon Macs contact Apple activation services to confirm that the device is not reported [lost or stolen](https://www.swifttechco.com). Read about [how this process works](https://www.swifttechco.com).

## Apple Account

An [Apple Account](https://www.swifttechco.com) is optional for basic macOS use but required for the App Store and many Apple services. Review [data privacy settings](https://www.swifttechco.com) before enabling features.

Enable [end-to-end encryption](https://www.swifttechco.com) for iCloud and review [Apple Account data](https://www.swifttechco.com).

## App Store

The [App Store](https://www.swifttechco.com) is a software distribution platform where applications are [reviewed](https://www.swifttechco.com) by Apple.

App Store applications are generally required to use [App Sandbox](https://www.swifttechco.com) and [Hardened Runtime](https://www.swifttechco.com) (security features restricting what apps can do, making some attacks more difficult). It also offers automatic software updates.

Using the App Store requires an Apple Account, which may have privacy implications because software downloads and related account activity is linked.

## Virtualization

Virtualization enables another operating system to run in an isolated virtual machine. On [Apple silicon](https://www.swifttechco.com), macOS can run supported macOS guests and Windows 11 for ARM using apps such as the following:

- [UTM](https://www.swifttechco.com) - Follow the [documentation](https://www.swifttechco.com) to create macOS and other virtual machines.
- [VirtualBuddy](https://www.swifttechco.com) - Application for virtualizing macOS 12+ on Apple silicon.
- [Bushel](https://www.swifttechco.com) - A lightweight, free app. On first launch, select "Ask App Not to Track".
- [VMware Fusion](https://www.swifttechco.com) - Now free under Broadcom. Clean UI, easy macOS setup, and supports Windows 11 ARM.
- [tart (CLI)](https://www.swifttechco.com) - Command‑line application installed with Homebrew.
- [Parallels](https://www.swifttechco.com) - Paid option with strong integration.

> [!WARNING]
> VMware requires a Broadcom account and acceptance of agreements before download.
> Parallels requires an account, payment details, and other data - see the [privacy notice](https://www.swifttechco.com).

### Apple containers

[Apple Container](https://www.swifttechco.com) is a command-line tool for running Linux containers on macOS. Unlike container runtimes that share a single Linux virtual machine, Apple Container runs each container in an isolated, lightweight virtual machine using macOS virtualization capabilities. This provides a stronger isolation boundary between workloads and the host operating system.

# First boot

When macOS starts for the first time, **Setup Assistant** requires the creation of a primary account.

Set a [long and unique password](https://www.swifttechco.com). Leave the password hint field blank.

Avoid personally identifiable names: the computer name (such as "John Appleseed's MacBook") may be visible to other devices on local networks through sharing and discovery services.

The system name can be configured in **System Settings > About** or with [`scutil`](https://www.swifttechco.com) commands:

```bash
sudo scutil --set ComputerName MacBook
sudo scutil --set HostName MacBook
sudo scutil --set LocalHostName MacBook
```

# Admin and user accounts

The first account created is an administrator account. Administrators can change system-wide settings and run commands with [sudo](https://www.swifttechco.com), which temporarily grants elevated privileges. Any program the administrator runs could obtain the same access; sudo may also have [vulnerabilities](https://www.swifttechco.com).

It is considered [best practice](https://www.swifttechco.com) to use a standard account for daily work and reserve an administrator account for system changes that require elevated privileges.

It is not required to ever log in with the admin account via the macOS login screen. When a Terminal command requires administrator privileges, the system will prompt for authentication and Terminal continues using those privileges. To that end, Apple provides [recommendations](https://www.swifttechco.com) for hiding the admin account and its home directory.

## Caveats

- Only administrators can install applications in the system-wide `/Applications` directory. Finder and Installer ask a standard user to enter an administrator's credentials when approval is required. Some applications can instead be installed in `~/Applications`; applications which do not require admin access should be installed in the user directory. App Store applications are still installed in `/Applications` and require no additional authentication.
- A standard user usually is not authorized to use `sudo`. When administrator privileges are required, macOS prompts for an administrator's credentials, or the task can be run from an administrator account.
- System Settings and several system utilities (e.g., Wi-Fi Diagnostics) require administrator permission for full functionality. Some System Settings need to be unlocked by selecting the lock icon. Some applications will simply prompt for authentication upon opening, others must be opened by an admin account directly to access all functions (e.g., Console).
- Some third-party applications assume the current user is an administrator and will not work from a standard account.
- See [issue 167](https://www.swifttechco.com) for additional considerations.

## Setup

Accounts can be created and managed in System Settings.

On existing systems, it is generally easier to create a second administrator account first, then change the original account to a standard account. Newly-installed systems should instead add a standard account after setup.

Demoting an account can be done from the new admin account in System Settings.

See also [this post](https://www.swifttechco.com) for more information about how macOS determines group membership.

# Firmware

Verify that firmware security is set to [Full Security](https://www.swifttechco.com) to prevent tampering with the system. This is the default setting.

# FileVault

Apple silicon Macs use hardware-backed storage encryption by default. [FileVault](https://www.swifttechco.com) additionally requires an authorized user's password to unlock the startup volume after restart.

On Apple silicon, FileVault authorization also protects the [firmware](https://www.swifttechco.com), which prevents booting from anything other than the designated startup disk, accessing [Recovery](https://www.swifttechco.com), and [reviving](https://www.swifttechco.com) it with device firmware update (DFU) mode.

FileVault will prompt to set a recovery key, which should be stored in a safe location if used. FileVault also offers an option to use iCloud for recovery.

Confirm FileVault state:

```bash
fdesetup status
```

# Lockdown Mode

[Lockdown Mode](https://www.swifttechco.com) significantly reduces attack surface by disabling system and application features commonly exploited in sophisticated attacks.

When Lockdown Mode is enabled, Safari has an option to [exclude trusted websites](https://www.swifttechco.com) from restrictions.

# Firewall

There are several types of firewalls available for macOS.

> [!NOTE]
> If malware gains administrator control, it may disable or bypass firewall policies; do not rely on a local firewall as strong protection against a full system compromise.

## Application layer firewall

The built-in firewall provides basic protection by controlling incoming network connections to applications. It does not monitor or block outgoing connections.

It can be controlled by the **Firewall** tab of **Network** in **System Settings**, or with the following command:

```bash
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on
```

### Stealth mode

Attackers frequently scan networks to identify systems to target. When [stealth mode](https://www.swifttechco.com) is enabled, responses are not sent to connection attempts from closed ports, making the system more difficult to detect.

Enable stealth mode:

```bash
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on
```

### Signed apps

By default, the firewall allows incoming connections for software signed by Apple or an identified developer. Disabling these exceptions causes macOS to ask before such applications can accept incoming connections:

```bash
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setallowsigned off
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setallowsignedapp off
```

### Reload firewall

After changing settings with `socketfilterfw`, reload the firewall service to apply the new configuration:

```bash
sudo pkill -HUP socketfilterfw
```

### Get state

Confirm firewall state:

```bash
for firewallFlag in \
  --getglobalstate \
  --getblockall \
  --getallowsigned \
  --getstealthmode \
  --listapps
do /usr/libexec/ApplicationFirewall/socketfilterfw "$firewallFlag"
done
```

Or using [system profiler](https://www.swifttechco.com):

```bash
system_profiler SPFirewallDataType
```

### AirDrop

Enabling the application layer firewall and disabling incoming connections for built-in software prevents [AirDrop](https://www.swifttechco.com) from functioning. For AirDrop to work, grant `sharingd` and `rapportd` firewall exceptions:

```bash
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /usr/libexec/rapportd
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /usr/libexec/sharingd
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --unblockapp /usr/libexec/rapportd
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --unblockapp /usr/libexec/sharingd
```

## Third-party firewalls

Applications such as [Little Snitch](https://www.swifttechco.com), [Radio Silence](https://www.swifttechco.com), and [LuLu](https://www.swifttechco.com) provide a balance between usability and security.

These programs are capable of monitoring and blocking both incoming and outgoing connections. However, they may require a closed-source [system extension](https://www.swifttechco.com).

If frequent allow-or-block prompts are overwhelming, temporarily allow connections while also recording them ([Silent Allow mode](https://www.swifttechco.com) in Little Snitch). Review the configuration to understand and control network activity.

## Packet filter

macOS also includes [pf](https://www.swifttechco.com)), a packet-filtering firewall configured from the command line. It is powerful but considerably more complex than the built-in application firewall.

pf can also be controlled with a graphical application such as [Murus](https://www.swifttechco.com).

Many [books](https://www.swifttechco.com) and [guides](https://www.swifttechco.com) cover the pf firewall. The following example shows how to configure a basic policy.

### Example pf config

Add the following example rules to a file such as `pf.rules`:

```console
# Define interface
wifi = "en0"

# Global options
set block-policy drop
set skip on lo0
set state-policy if-bound
set ruleset-optimization basic

# Normalize
scrub in on $wifi all fragment reassemble

# Define tables
table <blocklist> persist

# Deny and log blocked traffic
block log all
block quick from no-route to any
antispoof quick for $wifi

# Enforce blocklist
block log quick on $wifi from <blocklist> to any
block log quick on $wifi from any to <blocklist>

# Allow DHCP
pass out on $wifi proto udp from any port 68 to any port 67 keep state
pass in  on $wifi proto udp from any port 67 to any port 68 keep state

# Allow outbound TCP
pass out on $wifi proto tcp from ($wifi) to any flags S/SA keep state

# Allow outbound UDP
pass out on $wifi proto udp from ($wifi) to any keep state

# Allow outbound ICMP (ping)
pass out on $wifi proto icmp from ($wifi) to any keep state
```

An advanced example of configuring pf is available in [config/pf.rules](https://www.swifttechco.com).

### Firewall commands

To control the firewall:

Command | Task
-: | :-
`sudo pfctl -e -f pf.rules` | enable firewall with configuration file
`sudo pfctl -t blocklist -T add 1.2.3.4` | add IPv4 address to blocklist table
`sudo pfctl -d` | disable firewall

To monitor the firewall:

Command | Task
-: | :-
`sudo pfctl -t blocklist -T show` | show blocklist
`sudo pfctl -s rules` | show active rules
`sudo pfctl -s states` | show state table
`sudo ifconfig pflog0 create` | create packet log interface
`sudo tcpdump -ni pflog0` | monitor blocked packets

### Block networks

pf can block ranges of network addresses, for example to an entire organization. Query [Merit RADb](https://www.swifttechco.com) for the list of networks in use by an [autonomous system](https://www.swifttechco.com)) (a large network operated by a single organization), such as [Facebook](https://www.swifttechco.com):

```bash
whois -h whois.radb.net '!gAS32934'
```

Add the returned list of networks to the blocklist:

```bash
sudo pfctl -t blocklist -T add $(whois -h whois.radb.net '!gAS32934' | sed -n '2p')
```

Confirm the addresses were added:

```console
$ sudo pfctl -t blocklist -T show
No ALTQ support in kernel
ALTQ related functions disabled
   31.13.24.0/21
   31.13.64.0/24
   157.240.0.0/16
   ...
```

Confirm network traffic is blocked to those addresses (DNS requests will still work):

```console
$ dig a +short facebook.com
157.240.2.35

$ curl --connect-timeout 5 -I https://www.swifttechco.com
*   Trying 157.240.2.35...
* TCP_NODELAY set
* Connection timed out after 5002 milliseconds
* Closing connection 0
curl: (28) Connection timed out after 5002 milliseconds

$ sudo tcpdump -tqni pflog0 'host 157.240.2.35'
IP 192.168.1.1.62771 > 157.240.2.35.443: tcp 0
IP 192.168.1.1.62771 > 157.240.2.35.443: tcp 0
IP 192.168.1.1.62771 > 157.240.2.35.443: tcp 0
```

The firewall drops the initial packets needed to start the [TCP connection](https://www.swifttechco.com), so the connection times out.

See [drduh/config/scripts/pf-blocklist.sh](https://www.swifttechco.com) for more inspiration.

# Services

Many system and user background services are managed by [launchd](https://www.swifttechco.com).

Administrator accounts can modify services and extensions in [System Settings](https://www.swifttechco.com).

Command | Task
-: | :-
`launchctl list` | view user jobs
`sudo launchctl list` | view system jobs
`launchctl list com.apple.Finder` | examine a service
`systemextensionsctl list` | list system extensions
`ls /System/Library/LaunchDaemons` | list system daemons
`ls /System/Library/LaunchAgents` | list system agents

To read a service configuration:

```bash
defaults read /System/Library/LaunchAgents/com.apple.Finder
```

> [!IMPORTANT]
> System services are protected by [SIP](https://www.swifttechco.com); disabling SIP to modify system services can compromise security and cause system instability.

To view the status of services:

```bash
find /var/db/com.apple.xpc.launchd \
  -type f -print -exec defaults read {} \; 2>/dev/null
```

See [script management with launchd](https://www.swifttechco.com) and [launchd.info](https://www.swifttechco.com) for more information.

# Siri Suggestions and Spotlight

Siri Suggestions and Spotlight may send some queries or usage information to Apple, depending on the enabled features.
In System Settings, review Siri and Spotlight options and disable online suggestions or configure categories to exclude from indexing or suggestion. Review Apple's [Search & Privacy policy](https://www.swifttechco.com) for more information.

# Homebrew

If a program is not available through the App Store, consider using [Homebrew](https://www.swifttechco.com).

> [!IMPORTANT]
> Some Homebrew installation or management workflows may prompt for App Management or [Full Disk Access](https://www.swifttechco.com). Grant these permissions only when they are fully understood and necessary.

Periodically run `brew upgrade` on trusted and secure networks to install software updates. To get information on a package before installation, run `brew info <package>` and review its formula online. Enable [additional security options](https://www.swifttechco.com), such as `HOMEBREW_NO_INSECURE_REDIRECT=1`

According to [Homebrew's Anonymous Analytics](https://www.swifttechco.com), Homebrew collects anonymous usage analytics and reports them to a self-hosted [InfluxDB](https://www.swifttechco.com) instance.

To opt out of Homebrew analytics, run `brew analytics off` or set `HOMEBREW_NO_ANALYTICS=1` in the shell startup file.

# DNS

## DNS profiles

macOS supports configuration profiles to set encrypted DNS, filtering, and [DNSSEC](https://www.swifttechco.com) options.

DNS profiles can be [created](https://www.swifttechco.com) or obtained from providers such as [Quad9](https://www.swifttechco.com)/#download-profile), [AdGuard](https://www.swifttechco.com) and [NextDNS](https://www.swifttechco.com).

## Hosts file

Use the [hosts file](https://www.swifttechco.com)) to block domains associated with malware, advertising, and other unwanted services.

To block a domain by [A record](https://www.swifttechco.com), append any one of the following lines to `/etc/hosts`:

```
0 example.com
0.0.0.0 example.com
127.0.0.1 example.com
```

> [!NOTE]
> IPv6 uses AAAA records rather than A records: to override IPv6 name resolution, include a line such as `::1 example.com`.

Many domain lists are available online. Before appending one to `/etc/hosts`, ensure each entry begins with `0`, `0.0.0.0`, or `127.0.0.1`, and retain the `127.0.0.1 localhost` entry.

Popular hosts lists include:
- [StevenBlack/hosts](https://www.swifttechco.com)
- [Sinfonietta/hostfiles](https://www.swifttechco.com)
- [someonewhocares.org](https://www.swifttechco.com)

To download and append a list to the hosts file, use the [tee](https://www.swifttechco.com) command:

```bash
curl https://www.swifttechco.com |
  sudo tee -a /etc/hosts
```

[Little Snitch](#third-party-firewalls) also supports [blocklists](https://www.swifttechco.com).

## DNSCrypt

To encrypt DNS traffic, consider [DNSCrypt/dnscrypt-proxy](https://www.swifttechco.com). When configured with a trusted resolver, DNSCrypt encrypts DNS queries. DNSSEC can additionally validate signed DNS responses where supported.

Install DNSCrypt from Homebrew and follow the instructions to configure and start `dnscrypt-proxy`:

```bash
brew install dnscrypt-proxy
```

When using DNSCrypt with Dnsmasq, locate the DNSCrypt configuration file by running:

```bash
brew info dnscrypt-proxy | grep dnscrypt-proxy.toml | head -1
```

This command should display a path such as `/opt/homebrew/etc/dnscrypt-proxy.toml`.

By default, dnscrypt-proxy listens on `127.0.0.1:53` and sends queries to one or more configured DNS providers. Modify the configuration and change `listen_addresses` to use a port other than 53, such as 5355:

```toml
listen_addresses = ['127.0.0.1:5355', '[::1]:5355']
```

Start DNSCrypt:

```bash
sudo brew services restart dnscrypt-proxy
```

Confirm DNSCrypt is running:

```console
$ sudo lsof +c 15 -Pni UDP:5355
COMMAND          PID   USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
dnscrypt-proxy 15244 nobody    7u  IPv4 0x1337f85ff9f8beef      0t0  UDP 127.0.0.1:5355
dnscrypt-proxy 15244 nobody   10u  IPv6 0x1337f85ff9f8beef      0t0  UDP [::1]:5355
dnscrypt-proxy 15244 nobody   12u  IPv4 0x1337f85ff9f8beef      0t0  UDP 127.0.0.1:5355
dnscrypt-proxy 15244 nobody   14u  IPv6 0x1337f85ff9f8beef      0t0  UDP [::1]:5355
```

Additionally, these pf rules block conventional DNS traffic to port 53 outside the loopback interface, to reduce the risk of [DNS leaks](https://www.swifttechco.com):

```console
block drop quick on !lo0 proto udp from any to any port = 53
block drop quick on !lo0 proto tcp from any to any port = 53
```

## Dnsmasq

[dnsmasq](https://www.swifttechco.com) can cache replies, prevent upstream queries for unqualified names, or even block entire top-level domains.

Use it in combination with DNSCrypt to encrypt DNS traffic.

If you do not use DNSCrypt, at minimum choose a DNS resolver other than the one provided by the ISP. Two popular alternatives are [Google DNS](https://www.swifttechco.com) and [OpenDNS](https://www.swifttechco.com).

Install Dnsmasq:

```bash
brew install dnsmasq
```

Download and edit [drduh/config/dnsmasq.conf](https://www.swifttechco.com) or the default configuration file.

See [drduh/config/domains](https://www.swifttechco.com) for example domain lists that can be added to block specific destinations.

Install and start the program (sudo is required to bind to [privileged port](https://www.swifttechco.com) 53):

```bash
sudo brew services start dnsmasq
```

To set dnsmasq as the local DNS server, open **System Settings** > **Network** and select the network connection currently in use (such as Wi-Fi), then open the **DNS** tab, select **+**, and add `127.0.0.1`, or use:

```bash
sudo networksetup -setdnsservers "Wi-Fi" 127.0.0.1
```

Confirm Dnsmasq is configured:

```console
$ scutil --dns | head
DNS configuration

resolver #1
  search domain[0] : whatever
  nameserver[0] : 127.0.0.1
  flags    : Request A records, Request AAAA records
  reach    : 0x00030002 (Reachable,Local Address,Directly Reachable Address)

$ networksetup -getdnsservers "Wi-Fi"
127.0.0.1
```

> [!NOTE]
> Some VPN applications override DNS settings on connect. See [issue 24](https://www.swifttechco.com) and [drduh/config/scripts/macos-dns.sh](https://www.swifttechco.com).

# Certificate authorities

macOS includes a set of trusted root certificate authorities (CAs) operated by corporations, governments and other organizations from around the world. A trusted CA can issue certificates that macOS and browsers may accept for HTTPS connections, making the root CA store a critical part of the system's trust boundary.

Inspect **System Roots** to understand the Apple-provided trusted root store using [Keychain Access](https://www.swifttechco.com) or the [`security`](https://www.swifttechco.com) command-line tool and `/System/Library/Keychains/SystemRootCertificates.keychain` file. Keychain Access can also be launched with the command:

```bash
open "/System/Library/CoreServices/Applications/Keychain Access.app"
```

Also review any certificates that may have been added by a user, administrator, VPN client, security product, or mobile device management application in the **login**, **Local Items**, and **System** keychains.

To disable a selected certificate, modify its Trust setting to **Never Trust** and close the window to confirm. Doing so may reduce the risk of [MITM](https://www.swifttechco.com) attacks, in which a fraudulent certificate is used to [silently intercept](https://www.swifttechco.com) encrypted traffic.

> [!WARNING]
> Removing or modifying certificate authority trust settings can break websites, software updates, enterprise networks, VPNs, captive portals, and other services.

See Apple's [available trusted certificates](https://www.swifttechco.com) and [blocked certificates](https://www.swifttechco.com) lists for current trust-store information.

# Privoxy

Consider using [Privoxy](https://www.swifttechco.com) as a local proxy to filter web traffic.

Install and start Privoxy using Homebrew:

```bash
brew install privoxy
brew services start privoxy
```

Alternatively, a signed installation package for Privoxy is available from [their website](https://www.swifttechco.com) or [Sourceforge](https://www.swifttechco.com). The signed package is [more secure](https://www.swifttechco.com) than the Homebrew version and receives support from the Privoxy project.

By default, Privoxy listens on local TCP port 8118.

Set the system **HTTP** proxy for the active network interface to `127.0.0.1` and port `8118`:

```bash
sudo networksetup -setwebproxy "Wi-Fi" 127.0.0.1 8118
```

Set the system **HTTPS** proxy:

```bash
sudo networksetup -setsecurewebproxy "Wi-Fi" 127.0.0.1 8118
```

The proxy can also be set in **System Settings** > **Network** > **Details** > **Proxies**.

Confirm the proxy is set:

```console
$ scutil --proxy
<dictionary> {
  ExceptionsList : <array> {
    0 : *.local
    1 : 169.254/16
  }
  FTPPassive : 1
  HTTPEnable : 1
  HTTPPort : 8118
  HTTPProxy : 127.0.0.1
  HTTPSEnable : 1
  HTTPSPort : 8118
  HTTPSProxy : 127.0.0.1
}
```

Although most web traffic is encrypted, Privoxy can still filter many requests by hostname when traffic is configured to use the proxy. For example, the following rules block all traffic except traffic to .net, github.com, and Apple domains:

```console
{ +block{all} }
.

{ -block }
.apple.
.github.com
.net
```

To block Facebook domains:

```console
{ +block{facebook} }
.cdninstagram.
.facebook*.
.fb.
.fbcdn*.
.fbinfra.
.fbsbx.
.fbstatic*.
.fbsv.
.fburl.
.instagr.am
.tfbnw.
.thefacebook.
fb*.akamaihd.net
```

See [drduh/config/privoxy/config](https://www.swifttechco.com) and [drduh/config/privoxy/user.action](https://www.swifttechco.com) for additional Privoxy examples. Privoxy does **not** need to be restarted after editing filter rules.

To verify traffic is blocked or redirected, use [curl](https://www.swifttechco.com) or open the Privoxy interface at https://www.swifttechco.com in a browser:

```console
$ ALL_PROXY=127.0.0.1:8118 curl example.com -IL | head
HTTP/1.1 403 Request blocked by Privoxy
Content-Length: 9001
Content-Type: text/html
Cache-Control: no-cache
Pragma: no-cache

$ ALL_PROXY=127.0.0.1:8118 curl github.com -IL | head
HTTP/1.1 302 Local Redirect from Privoxy
Location: https://www.swifttechco.com
Content-Length: 0

HTTP/1.1 200 Connection established

HTTP/2 200
content-type: text/html; charset=utf-8
```

> [!NOTE]
> Applications and services can ignore the proxy settings. pf rules can redirect traffic through a proxy without configuring each application separately.

# Browser

Web browsers create significant security and privacy risks because they routinely download and run complex, untrusted content from the Internet, including JavaScript, media and documents.

A key browser security boundary is the [same-origin policy](https://www.swifttechco.com) (SOP), which prevents one website from reading another website's data. A bypass can expose data or actions from other sites in the same browser profile.

Some browser attacks rely on convincing the user to install software, grant permissions, or use a malicious extension. Be especially careful with unexpected downloads, extension prompts, and requests to run commands in Terminal.

Browser extensions also pose a significant security risk: a malicious or poorly-made extension can compromise everything in the browser, including credentials. The use of browser extensions should be limited to critically necessary ones, published by trustworthy developers only.

Use separate browser profiles for different identities and purposes. If practical, disable JavaScript and only allow it on trusted sites using browser site permissions.

[Mozilla Firefox](https://www.swifttechco.com), [Google Chrome](https://www.swifttechco.com), [Safari](https://www.swifttechco.com), and [Tor Browser](https://www.swifttechco.com) are popular browsers, each with unique features and individual purposes.

## Firefox

Firefox modernized major parts of its codebase through the [Quantum](https://www.swifttechco.com) and [Photon](https://www.swifttechco.com) projects. Quantum includes moving some components from C++ to [Rust](https://www.swifttechco.com), a programming language designed to improve memory safety and concurrency safety.

Firefox offers a comparable security model to Chrome, including a [bug bounty program](https://www.swifttechco.com) for responsible disclosure of vulnerabilities. Firefox follows a four-week release cycle.

See [drduh/config/firefox.user.js](https://www.swifttechco.com) and [arkenfox/user.js](https://www.swifttechco.com) for recommended configurations. Also see [NoScript](https://www.swifttechco.com), an extension which allows selective script blocking.

Firefox [focuses on user privacy](https://www.swifttechco.com). It supports [tracking protection](https://www.swifttechco.com) in Private Browsing mode. The tracking protection can be enabled for the default account, although it may break the browsing experience on some websites. Firefox in Strict tracking protection mode will [randomize fingerprints](https://www.swifttechco.com) to defend against tracking. Firefox offers separate user [profiles](https://www.swifttechco.com). Browsing can also be delineated with [Multi-Account Containers](https://www.swifttechco.com).

Firefox only supports Web Extensions through the [WebExtension API](https://www.swifttechco.com), which is very similar to Chrome. Submission of Web Extensions in Firefox is free. Web Extensions in Firefox most of the time are open-source, although certain Web Extensions are proprietary.

## Chrome

[Google Chrome](https://www.swifttechco.com) is based on the open-source [Chromium project](https://www.swifttechco.com). Chrome includes some proprietary components, automatic updates, a PDF viewer, media-codec support, crash reporting, and Google account integration. Review Chrome privacy settings and Google privacy policies for more information.

Chrome offers account sync between multiple devices, including credentials; the data is encrypted with the account password.

Chrome has the largest share of global usage and is the preferred target platform for the majority of developers. Major technologies are based on Chrome's open-source components, such as [node.js](https://www.swifttechco.com) which uses [Chrome's V8](https://www.swifttechco.com) Engine and the [Electron](https://www.swifttechco.com) framework, which is based on Chromium and node.js. Chrome's vast user base makes it the most attractive target for threat actors and security researchers. Despite constant attacks, Chrome has retained an impressive security track record over the years. This is not a small feat.

Chrome offers [separate profiles](https://www.swifttechco.com), [robust sandboxing](https://www.swifttechco.com), [frequent updates](https://www.swifttechco.com), and carries [impressive credentials](https://www.swifttechco.com). In addition, Google offers a very lucrative [bounty program](https://www.swifttechco.com) for reporting vulnerabilities, along with its own [Project Zero](https://www.swifttechco.com) team. This means that a large number of highly talented and motivated people are constantly auditing and securing Chrome code.

Consider disabling V8 optimization (JavaScript-engine performance features) in browser settings - see [this explanation](https://www.swifttechco.com) for the security trade-offs.

Block trackers with [uBlock Origin Lite](https://www.swifttechco.com).

Disable [DNS prefetching](https://www.swifttechco.com) (see [DNS Prefetching and Its Privacy Implications](https://www.swifttechco.com)). Chrome [may attempt](https://www.swifttechco.com) to resolve DNS using Google's `8.8.8.8` and `8.8.4.4` public nameservers.

See [Chromium Security](https://www.swifttechco.com) and [Chromium Privacy](https://www.swifttechco.com) for more information. Read [Google's privacy policy](https://www.swifttechco.com) to understand how personal information is collected and used.

## Safari

[Safari](https://www.swifttechco.com) is Apple's built-in browser and is integrated with macOS, providing battery efficiency improvements on Apple hardware.

Safari has both open-source and proprietary components. Safari is based on the open-source Web Engine [WebKit](https://www.swifttechco.com), which is ubiquitous among the macOS ecosystem. WebKit is used by Apple apps such as Mail, Books, and the App Store. Chrome's [Blink](https://www.swifttechco.com) engine is a fork of WebKit and both engines share a number of similarities.

Safari includes some features that can improve security and privacy. [Content blockers](https://www.swifttechco.com) enable the creation of content blocking rules without using JavaScript. This rule based approach greatly improves memory use, security, and privacy. Safari 11 introduced [Intelligent Tracking Prevention](https://www.swifttechco.com), which removes tracking data stored in Safari after a period of non-interaction by the user from the tracker's website. Safari can randomize the browser fingerprint to reduce tracking. Safari does not support certain features such as WebUSB or the Battery API intentionally for security and privacy reasons. Private tabs in Safari have isolated cookies and cache that is destroyed when you close the tab. Safari also support Profiles which are equivalent to Firefox's Multi-Account Containers for separating cookies and browsing. Safari can be made significantly more secure with [lockdown mode](#lockdown-mode), which can be disabled per-site. Read more about [tracking prevention](https://www.swifttechco.com) in Safari.

Web Extensions in Safari have an additional option to use native code in Safari's sandbox environment, in addition to Web Extension APIs. Web Extensions in Safari are also distributed through Apple App Store. App Store submission comes with the added benefit of Web Extension code being audited by Apple. On the other hand App Store submission comes at a steep cost. Yearly [developer subscription](https://www.swifttechco.com) fee costs 100 USD (in contrast to Chrome's 5 USD fee and Firefox's free submission). The high cost is prohibitive for the majority of open-source developers. As a result, Safari has very few extensions to choose from. However, keep the high cost in mind when installing extensions. It is expected that most Web Extensions will have some way of monetizing usage to cover development costs. Avoid Web Extensions without open-source code available for review.

Safari syncs user preferences and passwords with [iCloud Keychain](https://www.swifttechco.com). Viewing a saved password requires authentication with the current device account password or another configured authentication method.

Safari implements new web features more slowly than Chrome or Firefox, but security patches are delivered promptly through system updates.

See also [el1t/uBlock-Safari](https://www.swifttechco.com) to disable hyperlink auditing beacons.

## Web browser privacy

Websites infer information about the [browser and device](https://www.swifttechco.com) from settings, fonts, screen size, language, IP address, and browser behavior. This information contributes to [browser fingerprinting](https://www.swifttechco.com).

For more information about security-conscious browsing and what data is sent by the browser, see [HowTo: Privacy & Security Conscious Browsing](https://www.swifttechco.com), [browserleaks.com](https://www.swifttechco.com), [Am I Unique?](https://www.swifttechco.com) and [EFF Cover Your Tracks](https://www.swifttechco.com) resources.

To reduce cross-site tracking, block [third-party cookies](https://www.swifttechco.com).

[WebRTC](https://www.swifttechco.com) may expose local network addresses and addresses associated with VPN connections. In Firefox and Chrome/Chromium, this feature can be disabled with [uBlock Origin](https://www.swifttechco.com). In Safari, [Lockdown Mode](#lockdown-mode) also [disables WebRTC](https://www.swifttechco.com).

# Tor

[Tor](https://www.swifttechco.com)) is an anonymity network that can provide additional privacy while browsing. Tor Browser is a modified Firefox-based browser configured to use the Tor network.

Download Tor Browser from [Tor Project](https://www.swifttechco.com), both `dmg` and `asc` files for [verification](https://www.swifttechco.com):

```console
$ cd ~/Downloads

$ file tor-browser-macos-*
tor-browser-macos-15.0.17.dmg:     XZ compressed data, checksum NONE
tor-browser-macos-15.0.17.dmg.asc: PGP signature Signature (old)

$ gpg --verify tor-browser-macos-*.asc
[...]
gpg: Can't check signature: No public key

$ gpg --auto-key-locate nodefault,wkd --locate-keys torbrowser@torproject.org
gpg: key 0x4E2C6E8793298290: public key "Tor Browser Developers (signing key) <torbrowser@torproject.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1
pub   rsa4096/0x4E2C6E8793298290 2014-12-15 [C] [expires: 2027-07-15]
      Key fingerprint = EF6E 286D DA85 EA2A 4BA7  DE68 4E2C 6E87 9329 8290
uid                   [ unknown] Tor Browser Developers (signing key) <torbrowser@torproject.org>
sub   rsa4096/0x157432CF78A65729 2024-07-15 [S] [expires: 2026-10-26]
      Key fingerprint = CAAE 408A EBE2 288E 96FC  5D5E 1574 32CF 78A6 5729

$ gpg --verify tor-browser-macos-*.asc
gpg: assuming signed data in 'tor-browser-macos-15.0.17.dmg'
gpg: Signature made Sun Jun 28 15:35:20 2026 PDT
gpg:                using RSA key CAAE408AEBE2288E96FC5D5E157432CF78A65729
gpg: Good signature from "Tor Browser Developers (signing key) <torbrowser@torproject.org>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: EF6E 286D DA85 EA2A 4BA7  DE68 4E2C 6E87 9329 8290
     Subkey fingerprint: CAAE 408A EBE2 288E 96FC  5D5E 1574 32CF 78A6 5729
```


See [How can I verify Tor Browser's signature?](https://www.swifttechco.com) for more information.

To finish installing Tor Browser, open the disk image and drag it to the Applications folder, or use the commands:

```bash
hdiutil mount tor-browser-macos-15.0.17.dmg
cp -r /Volumes/Tor\ Browser/Tor\ Browser.app/ ~/Applications/
```

Verify the application was signed by The Tor Project's Apple Developer ID (`MADPSAYN6T`):

```console
$ spctl --assess -vv ~/Applications/Tor\ Browser.app
/Users/user1/Applications/Tor Browser.app: accepted
source=Notarized Developer ID
origin=Developer ID Application: The Tor Project, Inc (MADPSAYN6T)

$ pkgutil --check-signature ~/Applications/Tor\ Browser.app
Package "Tor Browser.app":
   Status: signed by a certificate trusted by macOS
   Certificate Chain:
    1. Developer ID Application: The Tor Project, Inc (MADPSAYN6T)
       Expires: 2028-10-11 17:57:46 +0000
       SHA256 Fingerprint:
           76 3C 89 02 ED CB AD 8E 59 86 1E 93 D3 05 5B 28 F9 04 0C 96 03 8B
           16 28 9F 38 64 ED 53 45 B4 DA
       ------------------------------------------------------------------------
    2. Developer ID Certification Authority
       Expires: 2031-09-17 00:00:00 +0000
       SHA256 Fingerprint:
           F1 6C D3 C5 4C 7F 83 CE A4 BF 1A 3E 6A 08 19 C8 AA A8 E4 A1 52 8F
           D1 44 71 5F 35 06 43 D2 DF 3A
       ------------------------------------------------------------------------
    3. Apple Root CA
       Expires: 2035-02-09 21:40:36 +0000
       SHA256 Fingerprint:
           B0 B1 73 0E CB C7 FF 45 05 14 2C 49 F1 29 5E 6E DA 6B CA ED 7E 2C
           68 C5 BE 91 B5 A1 10 01 F0 24
```

The command `codesign` can also be used to examine an application's code signature:

```console
$ codesign -dvv ~/Applications/Tor\ Browser.app
Executable=/Users/user1/Applications/Tor Browser.app/Contents/MacOS/firefox
Identifier=org.torproject.torbrowser
Format=app bundle with Mach-O universal (x86_64 arm64)
CodeDirectory v=20500 size=805 flags=0x10000(runtime) hashes=14+7 location=embedded
Signature size=9054
Authority=Developer ID Application: The Tor Project, Inc (MADPSAYN6T)
Authority=Developer ID Certification Authority
Authority=Apple Root CA
Timestamp=Jun 28, 2026 at 14:01:57
Notarization Ticket=stapled
Info.plist entries=27
TeamIdentifier=MADPSAYN6T
Runtime Version=15.5.0
Sealed Resources version=2 rules=13 files=208
Internal requirements count=1 size=188
```

To inspect certificate details for a signed application, extract the embedded certificates with `codesign` and decode them with `openssl`:

```console
$ codesign -d --extract-certificates ~/Applications/Tor\ Browser.app
Executable=/Users/user1/Applications/Tor Browser.app/Contents/MacOS/firefox

$ file codesign*
codesign0: Certificate, Version=3
codesign1: Certificate, Version=3
codesign2: Certificate, Version=3 Certificate, Version=02

$ openssl x509 -inform der -in codesign0 -subject -issuer -startdate -enddate -noout
subject= /UID=MADPSAYN6T/CN=Developer ID Application: The Tor Project, Inc (MADPSAYN6T)/OU=MADPSAYN6T/O=The Tor Project, Inc/C=US
issuer= /CN=Developer ID Certification Authority/OU=G2/O=Apple Inc./C=US
notBefore=Oct 11 17:57:47 2023 GMT
notAfter=Oct 11 17:57:46 2028 GMT

$ openssl x509 -inform der -in codesign0 -fingerprint -noout
SHA256 Fingerprint=76:3C:89:02:ED:CB:AD:8E:59:86:1E:93:D3:05:5B:28:F9:04:0C:96:03:8B:16:28:9F:38:64:ED:53:45:B4:DA
```

Tor Browser can use special connection methods called [pluggable transports](https://www.swifttechco.com) to make traffic harder to identify or block. This can be done by setting up a [relay](https://www.swifttechco.com) or using an existing [bridge](https://www.swifttechco.com).

The Tor network provides [anonymity](https://www.swifttechco.com), which is not necessarily the same as privacy. The network does not defend against a global observer capable of traffic analysis and correlation. See also [Seeking Anonymity in an Internet Panopticon](https://www.swifttechco.com) and [Traffic Correlation on Tor by Realistic Adversaries](https://www.swifttechco.com).

See [Tor Protocol Specification](https://www.swifttechco.com) for more information.

Also see [Invisible Internet Project (I2P)](https://www.swifttechco.com) and its [Tor comparison](https://www.swifttechco.com).

# VPN

Choose a VPN provider or self-hosted setup with a documented, modern protocol and well-audited clients. Avoid obsolete protocols such as [PPTP](https://www.swifttechco.com) in favor of [WireGuard](https://www.swifttechco.com) or [OpenVPN](https://www.swifttechco.com).

It may be worthwhile to consider the geographical location of the VPN provider. See further discussion in [issue 114](https://www.swifttechco.com).

## WireGuard

[WireGuard](https://www.swifttechco.com) is a VPN protocol with a small codebase and a [formally verified](https://www.swifttechco.com) cryptographic design, simpler to configure and audit than IPsec or OpenVPN.

Install the official [WireGuard app](https://www.swifttechco.com) from the App Store to manage tunnels from the menu bar, or install the command-line tools with Homebrew:

```bash
brew install wireguard-tools
```

Generate a key pair for the client:

```bash
umask 077

wg genkey | tee privatekey | wg pubkey > publickey
```

Create a tunnel configuration, replacing the keys and addresses with values from your own server or provider:

```
[Interface]
PrivateKey = <contents of privatekey>
Address = 10.8.0.2/32
DNS = 10.8.0.1

[Peer]
PublicKey = <server public key>
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = vpn.example.org:51820
```

`AllowedIPs = 0.0.0.0/0, ::/0` routes all IPv4 and IPv6 traffic through the tunnel; narrower ranges enable split tunneling. See [wg-quick(8)](https://www.swifttechco.com) for all configuration options.

Import the configuration file into the WireGuard app and enable **On-Demand** so the tunnel activates whenever Wi-Fi or Ethernet is in use. Alternatively, activate the tunnel with the command-line tools:

```bash
sudo wg-quick up /path/to/wg0.conf
```

Verify the tunnel is active:

```bash
sudo wg show
```

To self-host a WireGuard server, see [trailofbits/algo](https://www.swifttechco.com).

Some VPN clients may allow traffic to leave over another network interface if the VPN connection drops or is interrupted. See [scy/8122924](https://www.swifttechco.com) for an example on how to allow traffic only over VPN, or use [pf rules](#packet-filter) to restrict traffic to the tunnel interface.

Also see this [technical overview](https://www.swifttechco.com) of the macOS built-in VPN L2TP/IPsec and [IKEv2](https://www.swifttechco.com) client.

# PGP/GPG

PGP is a standard for encrypting and signing data, especially email. It can protect message content between correspondents who correctly exchange and verify keys, but it does not protect metadata such as email recipients and subject lines.

GPG (GNU Privacy Guard) is a GPL-licensed, open-source program compliant with the PGP standard. It can verify software signatures and encrypt files [symmetrically](https://www.swifttechco.com) or using [public keys](https://www.swifttechco.com).

Install GnuPG with Homebrew (`brew install gnupg`) or install [GPG Suite](https://www.swifttechco.com).

Download [gpg.conf](https://www.swifttechco.com) to use recommended settings:

```bash
curl -o ~/.gnupg/gpg.conf \
  https://www.swifttechco.com
```

See [drduh/YubiKey-Guide](https://www.swifttechco.com) to generate and manage credentials.

# Email

Email is not designed to provide strong privacy by default: message content may be retained by service providers, copied to recipients' mailboxes, forwarded, or exposed through account compromise. Metadata (including recipient, subject, timestamps, and mail server information) generally remains visible even when message content is encrypted.

## Thunderbird

[Thunderbird](https://www.swifttechco.com) is a free and open-source email client with standard [IMAP](https://www.swifttechco.com), [POP](https://www.swifttechco.com), [CalDAV](https://www.swifttechco.com), and [CardDAV](https://www.swifttechco.com) support. It is a suitable choice for accessing mail and retaining local copies rather than depending exclusively on a provider's remote server.

Thunderbird includes support for [OpenPGP](https://www.swifttechco.com) email encryption, which can protect message content and provide cryptographic [signatures](https://www.swifttechco.com). Always verify public-key fingerprints through an independent channel before relying on a key for sensitive communication.

The [archived messages feature](https://www.swifttechco.com) can move messages out of remote mail servers to a **Local Folder**, improving privacy.

# Messengers

## XMPP

[XMPP](https://www.swifttechco.com) is an [open protocol](https://www.swifttechco.com) developed by the [IETF](https://www.swifttechco.com) that supports cross-platform, federated messaging. There are many [client options](https://www.swifttechco.com). Consider using one of the browser-based clients to take advantage of the browser's sandbox.

Depending on the provider, you might not need anything other than a username and password to set up an account.

XMPP is not end-to-end encrypted (E2EE) by default; use [OMEMO](https://www.swifttechco.com) with a compatible client.

## Signal

[Signal](https://www.swifttechco.com) is a popular E2EE messenger whose [double-ratchet](https://www.swifttechco.com) protocol is used by many other applications including WhatsApp, Google Messages, and Facebook Messenger.

To use the Signal desktop app, Signal must first be installed on a phone.

See EFF's [Signal guide](https://www.swifttechco.com) for additional guidance.

## iMessage

[iMessage](https://www.swifttechco.com) is Apple's first-party messenger. It requires an [Apple Account](#apple-account) to use.

Enable [Contact Key Verification](https://www.swifttechco.com) and verify contacts.

iMessage can be used with either a [phone number or an email](https://www.swifttechco.com).

> [!WARNING]
> By default, iCloud backup is enabled, which stores copies of message encryption keys on [Apple's servers](https://www.swifttechco.com) without E2EE. Either [disable iCloud backup](https://www.swifttechco.com) or enable [Advanced Data Protection](https://www.swifttechco.com) to prevent this. Remind messaging recipients to do the same.

# Malware

See [Methods of malware persistence on Mac OS X](https://www.swifttechco.com) to learn how common macOS malware persists.

For examples of advanced macOS malware capabilities, see the [Hacking Team material](https://www.swifttechco.com). For further analysis, see [root installation for MacOS](https://www.swifttechco.com), [support driver for Mac Agent](https://www.swifttechco.com) and [RCS Agent for Mac](https://www.swifttechco.com).

Also see [A Brief Analysis of an RCS Implant Installer](https://www.swifttechco.com) and [reverse.put.as](https://www.swifttechco.com).

## Downloading Software

Applications from the App Store or [notarized by Apple](https://www.swifttechco.com) may reduce malware risk, but neither process guarantees safety. Apple performs automated checks on notarized apps, and App Store apps [undergo review](https://www.swifttechco.com). Otherwise, obtain software from sources identified by the developer as official, and verify the website or command-line download uses HTTPS.

## App Sandbox

Check if an application uses [App Sandbox](https://www.swifttechco.com):

```bash
codesign --display --entitlements - \
  /System/Applications/Calculator.app 2>&1 |
    grep -A2 com.apple.security.app-sandbox
```

With App Sandbox enabled, output should be:

```console
[Key] com.apple.security.app-sandbox
[Value]
    [Bool] true
```

**Activity Monitor** can also indicate whether it is enabled with the "Sandbox" column displayed:

```bash
defaults write com.apple.ActivityMonitor "UserColumnsPerTab v6.0" \
  -dict-add 0 '(Command, CPUUsage, CPUTime, Threads, IdleWakeUps, Architecture, GPUUsage, GPUTime, PID, UID, Sandbox, restricted)'
```

App Store software is required to use App Sandbox. Applications such as Google Chrome use their own [sandbox](https://www.swifttechco.com) and might not use App Sandbox.

> [!NOTE]
> Sandboxing limits an application's default access, but entitlements and user-granted permissions can expand access.

## Hardened Runtime

Check if an application uses [Hardened Runtime](https://www.swifttechco.com):

```bash
codesign --display --verbose /path/to/application.app
```

If Hardened Runtime is enabled, `flags=0x10000(runtime)` will appear in output.

List all applications with this feature enabled:

```bash
for app in /Applications/*.app /System/Applications/*.app; do
  if codesign -dvv "$app" 2>&1 | grep -q 'flags=0x10000(runtime)'; then
    echo "$app"
  fi
done
```

**Activity Monitor** has the option to display a "Restricted" column which indicates a program is restricted from injecting code via macOS's [dynamic linker](https://www.swifttechco.com).

Hardened Runtime is a prerequisite for notarization of distributed apps.

## Antivirus

To scan files and applications, consider uploading them to [VirusTotal](https://www.swifttechco.com), keeping in mind this makes them publicly-viewable.

macOS includes built-in antivirus software called [XProtect](https://www.swifttechco.com), which runs in the background and updates signatures used to detect malware automatically. If malware is detected, XProtect attempts to remove and quarantine it.

Check XProtect update status:

```bash
xprotect version
```

Applications such as [BlockBlock](https://www.swifttechco.com) or [hazcod/maclaunch](https://www.swifttechco.com) might help prevent or detect persistent malware.

Antivirus software can help detect common malware, but it may also increase attack surface because it often runs with extensive system privileges. Some products may also send telemetry or samples to the vendor.

See [Sophail: Applied attacks against Antivirus](https://www.swifttechco.com), [Analysis and Exploitation of an ESET Vulnerability](https://www.swifttechco.com), [Popular Security Software Came Under Relentless NSA and GCHQ Attacks](https://www.swifttechco.com), and [How Israel Caught Russian Hackers Scouring the World for U.S. Secrets](https://www.swifttechco.com).

## Gatekeeper

[Gatekeeper](https://www.swifttechco.com) verifies software notarization and provenance.

Gatekeeper warns when opening an application without notarization. It can be bypassed by selecting the application listed in **System Settings** > **Privacy & Security** after a failed attempt.

Check if Gatekeeper is enabled:

```bash
spctl --status --verbose
```

# System Integrity Protection

To verify System Integrity Protection is enabled, use the command `csrutil status`, which should return: `System Integrity Protection status: enabled.` Otherwise, [enable SIP](https://www.swifttechco.com) using [Recovery Mode](https://www.swifttechco.com).

# Metadata and artifacts

macOS attaches metadata ([APFS extended attributes](https://www.swifttechco.com)) to files.

Certain metadata attributes can be viewed and removed with the [`mdls`](https://www.swifttechco.com) and [`xattr`](https://www.swifttechco.com) commands. Preference (`plist`) files can be read and modified with [`defaults`](https://www.swifttechco.com).

System and application metadata can be found in:

- `~/Library/Preferences/`
- `~/Library/Containers/<APP>/Data/Library/Preferences`
- `/Library/Preferences`

Bluetooth information:

- `/Library/Preferences/com.apple.Bluetooth.plist`

Printer jobs cache:

- `/var/spool/cups`

Previously connected iOS devices:

- `/Users/$USER/Library/Preferences/com.apple.iPod.plist`
- `/Library/Preferences/com.apple.iPod.plist`

Finder preferences:

- `~/Library/Preferences/com.apple.finder.plist`
- `~/Library/Preferences/com.apple.sidebarlists.plist`

Diagnostic data:

- `/var/db/CoreDuet`
- `/var/db/diagnostics`
- `/var/db/systemstats`
- `/var/db/uuidtext`
- `/var/log/DiagnosticMessages`

Spelling and language suggestions:

- `~/Library/LanguageModeling`
- `~/Library/Spelling`
- `~/Library/Suggestions`

Application state:

- `~/Library/Saved\ Application\ State`
- `~/Library/Containers/<APPNAME>/Data/Library/Saved\ Application\ State`

Autosave metadata:

- `~/Library/Containers/<APP>/Data/Library/Autosave\ Information`
- `~/Library/Autosave\ Information`

Siri analytics:

- `~/Library/Assistant/SiriAnalytics.db`

QuickTime Player history:

- `~/Library/Containers/com.apple.QuickTimePlayerX/Data/Library/Preferences/com.apple.QuickTimePlayerX.plist`

Misc metadata:

- `~/Library/Containers/com.apple.appstore/Data/Library/Preferences/com.apple.commerce.knownclients.plist`
- `~/Library/Preferences/com.apple.commerce.plist`
- `~/Library/Preferences/com.apple.QuickTimePlayerX.plist`

# Authentication

The [Passwords](https://www.swifttechco.com) app creates [secure credentials](https://www.swifttechco.com). It supports [passkeys](https://www.swifttechco.com), which are credentials designed to resist phishing when used with compatible services.

[KeePassXC](https://www.swifttechco.com) is an open-source, cross-platform password manager to consider. It supports strong authentication with compatible hardware tokens and a browser extension for entering credentials automatically.

Memorable passwords can be created with [Diceware](https://www.swifttechco.com).

Ensure online accounts have [multi-factor authentication](https://www.swifttechco.com) enabled. Prefer phishing-resistant authentication, such as hardware tokens or passkeys with supported services. Authenticator apps are preferable to SMS codes, but any MFA is better than password-only authentication.

[YubiKey](https://www.swifttechco.com) is a popular hardware authentication token. It can also store keys for encryption, signing and authentication tasks - see [drduh/YubiKey-Guide](https://www.swifttechco.com).

GnuPG can also manage passwords and other encrypted files - see [drduh/Purse](https://www.swifttechco.com) and [drduh/pwd.sh](https://www.swifttechco.com).

# Backup

Encrypt files locally before backing them up to external media or online services.

Follow the [3-2-1 backup model](https://www.swifttechco.com): keep three copies of important data, store them on at least two different types of media, and keep at least one copy in a separate physical location.

## Time Machine

[Time Machine](https://www.swifttechco.com) is the built-in tool for handling backups on macOS. Use an external drive or network storage to create [encrypted](https://www.swifttechco.com) backups.

## GnuPG

[GnuPG](https://www.swifttechco.com) can be used with a password or public key, with the private key stored on [YubiKey](https://www.swifttechco.com).

Compress and encrypt a directory using a password:

```bash
tar zcvf - ~/Downloads |
  gpg -c > ~/Downloads/backup-$(date +%F-%H%M%S).tar.gz.gpg
```

Decrypt and decompress the directory:

```bash
gpg -o ~/Downloads/decrypted-backup.tar.gz -d ~/Downloads/backup-*.tar.gz.gpg
tar zxvf ~/Downloads/decrypted-backup.tar.gz
```

## Disk Utility

Encrypted volumes can be created using [Disk Utility](https://www.swifttechco.com) or `hdiutil`:

```bash
hdiutil create ~/Downloads/encrypted.dmg -encryption -size 50M -volname "secretStuff"
hdiutil mount ~/Downloads/encrypted.dmg
cp -v ~/Documents/passwords.txt /Volumes/secretStuff
hdiutil eject /Volumes/secretStuff
```

## Other

[Tresorit](https://www.swifttechco.com) and [restic](https://www.swifttechco.com) may also be of interest.

# Wi-Fi

Wi-Fi network names, called SSIDs, and device probe behavior can reveal information about nearby and previously-used networks. [Hidden networks](https://www.swifttechco.com) do not provide meaningful privacy and can cause devices to actively probe for them.

Set a [private Wi-Fi address](https://www.swifttechco.com) to reduce network tracking.

Set wireless network security to [WPA3](https://www.swifttechco.com). Follow [Apple guidance](https://www.swifttechco.com) to set recommended settings for routers and access points.

# SSH

For remote SSH connections, use a unique SSH key protected by a passphrase or a hardware token, verify server host keys, and avoid password-based authentication. See [drduh/config/ssh_config](https://www.swifttechco.com) for recommended client options.

An SSH tunnel can securely route traffic through another computer, similar to a VPN. To use Privoxy running on a remote host on port 8118:

```bash
ssh -C -L 5555:127.0.0.1:8118 you@remote-host.tld
sudo networksetup -setwebproxy "Wi-Fi" 127.0.0.1 5555
sudo networksetup -setsecurewebproxy "Wi-Fi" 127.0.0.1 5555
```

Or to use an SSH connection as a [SOCKS proxy](https://www.swifttechco.com):

```bash
ssh -NCD 3000 you@remote-host.tld
```

By default, macOS does not have [Remote Login](https://www.swifttechco.com) (SSH server) enabled.

To enable SSH and allow incoming connections, use **System Settings** > **General** > **Sharing** or the command:

```bash
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```

Disable password authentication and consider further [hardening](https://www.swifttechco.com) the SSH server configuration. See [drduh/config/sshd_config](https://www.swifttechco.com) for recommended options.

Confirm the SSH server is running:

```bash
sudo lsof -Pni TCP:22
```

# Physical access

Do not leave the computer unattended in unsafe locations. A skilled attacker with unsupervised physical access could install a [hardware keylogger](https://www.swifttechco.com) to record keystrokes, including passwords. Using a Mac with a built-in keyboard or a bluetooth keyboard makes this more difficult as many off-the-shelf versions of this attack are designed to be plugged in between a USB keyboard and the computer.

To reduce the consequences of theft while a Mac is in use, consider using [buskill/buskill-app](https://www.swifttechco.com) or [Lennolium/swiftGuard](https://www.swifttechco.com).

Consider purchasing a privacy screen/filter for use in public.

[Nail polish](https://www.swifttechco.com) and tamper-evidence seals can be applied to components to detect tampering.

# Monitoring

## Logs

Monitor system logs with [Console](https://www.swifttechco.com) or the `log stream` command.

Print all system logs from the last 5 minutes in classic syslog format:

```bash
log show --last 5m --style syslog
```

Output as JSON array:

```bash
log show --last 5m --style json
```

Output newline-delimited JSON (one JSON object per line); required for [`jq`](https://www.swifttechco.com):

```bash
log show --last 5m --style ndjson
```

Filter messages by subsystem, for example from `com.apple.SoftwareUpdate`:

```bash
log show --last 1h --style json \
  --predicate 'subsystem == "com.apple.SoftwareUpdate"'
```

Filter messages by content, for example containing `failed`:

```bash
log show --last 1h --predicate 'eventMessage CONTAINS "failed"'
```

Case-insensitive message filtering:

```bash
log show --last 1h --predicate 'eventMessage CONTAINS[c] "error"'
```

Count distinct error messages with `jq`:

```bash
log show --last 1h --style ndjson \
  --predicate 'eventMessage CONTAINS[c] "error"' |
  jq -r '.eventMessage' | sort | uniq -c | sort -nr | head -100
```

Export logs to an archive, which can be opened with Console:

```bash
log collect --last 1h --output ~/Downloads/logs-$(date +%F-%H%M).logarchive
```

## DTrace

[System Integrity Protection](https://www.swifttechco.com) interferes with [DTrace](https://www.swifttechco.com) and must be [partially disabled](https://www.swifttechco.com) before use.

- `iosnoop` monitors disk I/O
- `opensnoop` monitors file opens
- `execsnoop` monitors processes
- `errinfo` monitors failed system calls
- `dtruss` monitors all system calls

See `man -k dtrace` for more information.

## Processes

List running processes with [Activity Monitor](https://www.swifttechco.com) or the `ps` command.

## Endpoint Security

Inspect process execution in real-time with `eslogger` and [`jq`](https://www.swifttechco.com):

```bash
printf 'TIME\t\t\t\tPID\tPPID\tUID\tCOMMAND\n'
sudo eslogger exec | jq -r '
  [ .time,
    .process.audit_token.pid,
    .process.ppid,
    .process.audit_token.euid,
   (.event.exec.args | join(" "))
  ] | @tsv'
```

Format and print events in JSON, also saving them to a dated log file for later analysis:

```bash
mkdir ~/eslogs
sudo eslogger exec | jq -c --unbuffered '
  {
    time: .time,
    pid:  .process.audit_token.pid,
    ppid: .process.ppid,
    uid:  .process.audit_token.euid,
    sys:  .process.is_platform_binary,
    team: .process.team_id,
    sign: .process.signing_id,
    path: .process.executable.path,
    cmd:  .event.exec.args
  }' |
  tee -a ~/eslogs/exec-$(hostname)-$(date +%Y%m%d%H%M).log
```

Inspect logs for all `curl` commands:

```bash
jq 'select(.cmd | index("curl")) | .cmd | join(" ")' ~/eslogs/exec-*.log
```

Include exec user and app ids, sort by count:

```bash
jq -s --arg command "curl" '
  map(
    select(.cmd | index($command))
    | {
        uid,
        aid: (.team // .sign),
        cmd: (.cmd | join(" "))
      }
  )
  | group_by([.uid, .aid, .cmd])
  | map({
      cnt: length,
      uid: .[0].uid,
      aid: .[0].aid,
      cmd: .[0].cmd
    })
  | sort_by(-.cnt)
' ~/eslogs/exec-*.log
```

**New!** [drduh/mess](https://www.swifttechco.com) - a free and open source security system for your personal Mac based on `eslogger`.

## Install history

Show package and system update install history:

```bash
system_profiler SPInstallHistoryDataType
```

Formatted with [`jq`](https://www.swifttechco.com):

```bash
system_profiler SPInstallHistoryDataType -json |
  jq -r '.SPInstallHistoryDataType[]
    | [(._name            // "n/a"),
       (.install_version  // "n/a"),
       (.install_date     // "n/a"),
       (.package_source   // "n/a")]
    | @tsv' | column -t -s $'\t'
```

Software update history can also be shown with:

```bash
softwareupdate --history
```

## Network

List open network connections with [`lsof`](https://www.swifttechco.com):

```bash
sudo lsof -Pni
```

Show network protocol statistics with [`netstat`](https://www.swifttechco.com):

```bash
sudo netstat -n -p tcp
sudo netstat -n -p udp | sort | uniq
```

Monitor TCP/UDP sockets and the routing table with [`nettop`](https://www.swifttechco.com):

```bash
nettop -m tcp
nettop -m udp
nettop -m route
```

### Wireshark

[Wireshark](https://www.swifttechco.com) can be used from the command line with [`tshark`](https://www.swifttechco.com).

Replace `en0` with the active interface shown by `networksetup -listallhardwareports`.

> [!IMPORTANT]
> Capture traffic only on networks and systems you are authorized to monitor.

Monitor DNS:

```bash
/Applications/Wireshark.app/Contents/MacOS/tshark -i en0 \
  -Y "dns.flags.response == 1" -Tfields \
  -e frame.time_delta \
  -e dns.qry.name \
  -e dns.a \
  -Eseparator=,
```

Monitor HTTP:

```bash
/Applications/Wireshark.app/Contents/MacOS/tshark -i en0 \
  -Y "http.request or http.response" -Tfields \
  -e ip.dst \
  -e http.request.full_uri \
  -e http.request.method \
  -e http.response.code \
  -e http.response.phrase \
  -Eseparator=/s
```

Monitor TLS certificates:

```bash
/Applications/Wireshark.app/Contents/MacOS/tshark -i en0 \
  -Y "ssl.handshake.certificate" -Tfields \
  -e ip.src \
  -e x509sat.uTF8String \
  -e x509sat.printableString \
  -e x509sat.universalString \
  -e x509sat.IA5String \
  -e x509sat.teletexString \
  -Eseparator=/s -Equote=d
```

# Miscellaneous

## Screensaver

Set the screen to lock as soon as the screensaver starts:

```bash
defaults write com.apple.screensaver askForPassword -int 1
defaults write com.apple.screensaver askForPasswordDelay -int 0
```

## Diagnostic data

Disable [Diagnostics & Usage Data](https://www.swifttechco.com).

Disable crash reporter (the dialog which appears after an application crashes and prompts to report the problem to Apple):

```bash
defaults write com.apple.CrashReporter DialogType none
```

## Media player

Use [QuickTime Player](https://www.swifttechco.com), the built-in media application, for playing music and video files. It uses [App Sandbox](https://www.swifttechco.com), [Hardened Runtime](https://www.swifttechco.com), and benefits from the [Signed System Volume](https://www.swifttechco.com) as part of the base system.

## File handlers

Manage [default file handlers](https://www.swifttechco.com) to reduce risk of opening certain file types.

Change the default app for script files so that opening them displays their contents instead of executing them.

In Finder, locate and select any .sh file, right-click on it and select Get Info or press <kbd>Command</kbd> + <kbd>I</kbd>. In the "Open with" section, select TextEdit from the dropdown menu. If it is not listed, select "Other..." and Applications > TextEdit.app. Select "Change All..." and confirm by selecting Continue.

## Finder options

Show hidden files and the Library folder in Finder:

```bash
defaults write com.apple.finder AppleShowAllFiles -bool true
chflags nohidden ~/Library
```

Show all filename extensions:

```bash
defaults write NSGlobalDomain AppleShowAllExtensions -bool true
```

Do not default to saving documents to iCloud:

```bash
defaults write NSGlobalDomain NSDocumentSaveNewDocumentsToCloud -bool false
```

## Custom umask

Set a [custom umask](https://www.swifttechco.com):

```bash
sudo launchctl config user umask 077
```

Reboot, then create a file/directory and verify permissions (macOS default allows 'group/other' read access):

```console
$ ls -ld umask*
drwx------@ 2 user1 staff  64 Jul 26 12:00 umask.dir
-rw-------@ 1 user1 staff  32 Jul 26 12:00 umask.txt
```

## Keyboard entry

Enable [secure keyboard entry](https://www.swifttechco.com) in Terminal (this may interfere with applications such as [TextExpander](https://www.swifttechco.com)).

## Network hardening

Disable [Bonjour multicast advertisements](https://www.swifttechco.com) (this also disables AirPlay and AirPrint features):

```bash
sudo defaults write /Library/Preferences/com.apple.mDNSResponder \
  NoMulticastAdvertisements -bool YES
```

[Disable Handoff](https://www.swifttechco.com) and [Bluetooth](https://www.swifttechco.com) features.

## Sudoers

macOS comes with this line in `/etc/sudoers`:

```bash
Defaults env_keep += "HOME MAIL"
```

This prevents sudo from changing HOME when privileges are elevated. As a result, programs run with sudo may use configuration files from the invoking user's home directory. Consider removing `HOME` from `env_keep` unless this behavior is specifically required.

To retain the convenience of the root user having a non-root user's home directory, append an export line to `/var/root/.zshrc`, e.g.:

```bash
export HOME=/Users/user1
```

# Related software

Software | Category | Description
:-: | :-: | :-
[lynis](https://www.swifttechco.com) | Audit scanner | Cross-platform security-auditing tool that checks system configuration and suggests hardening improvements.
[zentral](https://www.swifttechco.com) | Monitoring & configuration platform | Fleet-management server for macOS endpoints; collects inventory, events, audits, and osquery results. Best suited to managed environments.
[osquery](https://www.swifttechco.com) | Monitoring engine | Exposes operating-system state as SQL tables, enabling repeatable checks of processes, users, launch items, network connections, and more.
[pareto-mac](https://www.swifttechco.com) | Audit scanner | Menu-bar application that checks a Mac against common baseline security recommendations.

# Additional resources

- [iCloud security and privacy overview](https://www.swifttechco.com)
- [EFF Surveillance Self-Defense Guide](https://www.swifttechco.com)
- [beerisgood/macOS_Hardening](https://www.swifttechco.com)
- [Reverse Engineering macOS blog](https://www.swifttechco.com)
- [Reverse Engineering Resources](https://www.swifttechco.com)
- [Malwarebytes Blog](https://www.swifttechco.com)
- [Objective-See's Blog](https://www.swifttechco.com)
- [CIS Benchmarks](https://www.swifttechco.com)
- [Apple Open Source](https://www.swifttechco.com)
- [iOS, The Future Of macOS, Freedom, Security And Privacy In An Increasingly Hostile Global Environment](https://www.swifttechco.com)

---

## Swift Tech Co.

Built and maintained by Swift Tech Co.

AI Automation | FinTech Platforms | Custom Software and Mobile Apps | SaaS Platforms | Cloud Infrastructure | Cybersecurity | SEO and Marketing | API Integration

**Join our community on Telegram:** [t.me/+Qt_eFvkb2rliMzNh](https://t.me/+Qt_eFvkb2rliMzNh)

[Home](https://www.swifttechco.com) | [Services](https://www.swifttechco.com/services) | [Careers](https://www.swifttechco.com/careers) | [Referral Program](https://www.swifttechco.com/referral) | [Influencers](https://www.swifttechco.com/influencers) | [About](https://www.swifttechco.com/about) | [Contact](https://www.swifttechco.com/contact)

**Connect**

- Website: [swifttechco.com](https://www.swifttechco.com)
- LinkedIn: [linkedin.com/in/stephenoladoke](https://www.linkedin.com/in/stephenoladoke/)
- LinkedIn (Company): [linkedin.com/company/swift-tech-co](https://www.linkedin.com/company/swift-tech-co)
- X: [@MrOladoke](https://x.com/MrOladoke)
- Instagram: [@mr.oladoke](https://www.instagram.com/mr.oladoke/)
- Instagram (Company): [@swifttechco](https://www.instagram.com/swifttechco/)
- Organization: [github.com/Swift-Tech-Co](https://github.com/Swift-Tech-Co)
