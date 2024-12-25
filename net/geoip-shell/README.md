# **geoip-shell**
User-friendly and powerful geoblocker for Linux. Supports both **nftables** and **iptables** firewall management utilities.

The idea of this project is making geoblocking (i.e. restricting access from or to Internet addresses based on geolocation) easy on (almost) any Linux system, no matter which hardware, including desktop, server, container, VPS or router, while also being reliable and providing flexible configuration options for the advanced users.

## Table of contents
- [Main Features](#main-features)
- [Usage](#usage)
- [Outbound geoblocking](#outbound-geoblocking)
- [Pre-requisites](#pre-requisites)
- [Notes](#notes)
- [In detail](#in-detail)
- [OpenWrt](#openwrt)
- [Privacy](#privacy)

## **Main Features**
* Core functionality is creating either a whitelist or a blacklist in the firewall using automatically downloaded ip lists for user-specified countries.

* ip lists are fetched either from **RIPE** (regional Internet registry for Europe, the Middle East and parts of Central Asia) or from **ipdeny**, or from **MaxMind**. All 3 sources provide updated ip lists for all regions.

* All firewall rules and ip sets required for geoblocking to work are created automatically during the initial setup.

* Supports automated interactive setup for easy configuration.

* Automates creating auxiliary firewall rules based on user's preferences (for example, when configuring on a host in whitelist mode, geoip-shell will detect LAN subnets and suggest to add them to the whitelist)

* Implements optional (enabled by default) persistence of geoblocking across system reboots and automatic updates of the ip lists.

* After installation, a utility is provided to check geoblocking status and firewall rules or change country codes and geoblocking-related config.

* Supports inbound and outbound geoblocking.

* Supports ipv4 and ipv6.

* Supports running on OpenWrt.

### **Reliability**:
- Downloaded ip lists go through validation which safeguards against application of corrupted or incomplete lists to the firewall.

<details> <summary>Read more:</summary>

- Supports the 'MaxMind' commercial source which provides more accurate ip lists, both the free GeoLite2 database and the paid GeoIP2 database. Note that in order to use the MaxMind source, you need to have a MaxMind account.
- With nftables, utilizes nftables atomic rules replacement to make the interaction with the system firewall fault-tolerant and to completely eliminate time when geoip is disabled during an automatic update.
- All scripts perform extensive error detection and handling.
- All user input is validated to reduce the chance of accidental mistakes.
- Verifies firewall rules coherence after each action.
- Automatic backup of geoip-shell state (optional, enabled by default except on OpenWrt).
- Automatic recovery of geoip-shell firewall rules after a reboot (a.k.a persistence) or in case of unexpected errors.
- Supports specifying trusted ip addresses anywhere on the Internet which will bypass geoip blocking to make it easier to regain access to the machine if something goes wrong.
</details>

### **Efficiency**:
- Utilizes the native nftables sets (or, with iptables, the ipset utility) which allows to create efficient firewall rules with thousands of ip ranges.

<details><summary>Read more:</summary>

- With nftables, optimizes geoblocking for low memory consumption or for performance, depending on the RAM capacity of the machine and on user preference. With iptables, automatic optimization is implemented.
- Ip list parsing and validation are implemented through efficient regex processing which is very quick even on slow embedded CPU's.
- Implements smart update of ip lists via data timestamp checks, which avoids unnecessary downloads and reconfiguration of the firewall.
- For inbound geoblocking, uses the "prerouting" hook in kernel's netfilter component which shortens the path unwanted packets travel in the system and may reduce the CPU load if any additional firewall rules process incoming traffic down the line.
- Supports the 'ipdeny' source which provides aggregated ip lists (useful for embedded devices with limited memory).
- Scripts are only active for a short time when invoked either directly by the user or by the init script/reboot cron job/update cron job.

</details>

### **User-friendliness**:
- Good command line interface and useful console messages.

<details><summary>Read more:</summary>

- Extensive and (usually) up-to-date documentation.
- Sane settings are applied during installation by default, but also lots of command-line options for advanced users or for special corner cases are provided.
- Provides a utility (symlinked to _'geoip-shell'_) for the user to change geoblocking config (turn geoblocking on or off, configure outbound geoblocking, change country codes, change geoblocking mode, change ip lists source, change the cron schedule etc).
- Provides a command _('geoip-shell status')_ to check geoblocking status, which also reports if there are any issues.
- In case of an error or invalid user input, provides useful error messages to help with troubleshooting.
- All main scripts display detailed 'usage' info when executed with the '-h' option.
- Most of the code should be fairly easy to read and includes a healthy amount of comments.
</details>

### **Compatibility**:
- Since the project is written in POSIX-compliant shell code, it is compatible with virtually every Linux system (as long as it has the [pre-requisites](#pre-requisites)). It even works well on simple embedded routers with 8MB of flash storage and 128MB of memory (for nftables, 256MB is recommended if using large ip lists such as the one for US until the nftables team releases a fix reducing memory consumption).
- The code is regularly tested on Debian, Linux Mint and OpenWrt, and occasionally on Alpine Linux and Gentoo.
- While not specifically tested by the developer, there have been reports of successful use in LXC containers.

<details><summary>Read more:</summary>

- Supports running on OpenWrt.
- The project avoids using non-common utilities by implementing their functionality in custom shell code, which makes it faster and compatible with a wider range of systems.
</details>

## **Initial setup**
Once the installation completes, the installer will suggest to automatically start the interactive setup. If you ran the install script non-interactively or interrupted the setup at some point, you can manually (re)start interactive setup by running `geoip-shell configure`.

Interactive setup gathers the important config via dialog with the user and does not require any command line arguments. If you are not sure how to answer some of the questions, read [SETUP.md](SETUP.md).

Alternatively, some or all of the config options may be provided via command-line arguments.

**NOTE:** Some features are only accessible via command-line arguments. In particular, by default, initial setup only configures inbound geoblocking and leaves outbound geoblocking in disabled state. If you want to configure outbound geoblocking, read the section [Outbound geoblocking](#outbound-geoblocking).

_To find out more, run `geoip-shell -h` or read [NOTES.md](NOTES.md) and [DETAILS.md](DETAILS.md)_

## **Usage**

By default, ip lists will be updated daily around 4:15am local time (to avoid everyone loading the servers at the same time, the default minute is randomized to +-5 precision at the time of initial setup and the seconds are randomized at the time of automatic update).

If you want to change geoblocking config or check geoblocking status, you can do that via the provided utilities.
A selection of options is given here, for additional options run `geoip-shell -h` or read [NOTES.md](NOTES.md) and [DETAILS.md](DETAILS.md).

**Note** that when using the `geoip-shell configure` command, if direction is not specified, direction-specific options apply to **inbound** geoblocking. Direction-specific options are `-m <whitelist|blacklist|disable>`, `-c <country_codes>`, `-p <ports>`. To specify direction, add `-D <inbound|outbound>` before specifying options for that direction (for more details, read the section [Outbound geoblocking](#outbound-geoblocking)).

**To check current geoip blocking status:** `geoip-shell status`. For a list of all firewall rules in the main geoblocking chains and for a detailed count of ip ranges in each ip list: `geoip-shell status -v`.

**To configure geoblocking mode:**

`geoip-shell configure -m <whitelist|blacklist|disable>`

(`disable` unloads all ip lists and removes firewall geoblocking rules for both directions)

**To change countries in the geoblocking whitelist/blacklist:**

`geoip-shell configure -c <"country_codes">`

_<details><summary>Example:</summary>_
- to set countries to Germany and Netherlands: `geoip-shell configure -c "DE NL"`
</details>

**To geoblock or allow specific ports or ports ranges:**

`geoip-shell configure -p <[tcp|udp]:[allow|block]:[all|<ports>]>`

_(for detailed description of this feature, read [NOTES.md](NOTES.md), sections 10-12)_

**To enable or disable geoblocking** (only adds or removes the geoblocking enable rules, leaving all other firewall geoblocking rules and ip sets in place):

`geoip-shell <on|off>`

**To change ip lists source:** `geoip-shell configure -u <ripe|ipdeny|maxmind>`

**To have certain trusted ip addresses or subnets, either in your LAN or anywhere on the Internet, bypass geoblocking:**

`geoip-shell configure -t <["ip_addresses"]|none>`

`none` removes previously set trusted ip addresses.

**To have certain LAN ip addresses or subnets bypass geoip blocking:**

`geoip-shell configure -l <["ip_addresses"]|auto|none>`

LAN addresses can only be configured when geoblocking mode for at least one direction is set to `whitelist`. Otherwise there is no need to whitelist LAN addresses. Also whitelisting LAN addresses is typically only needed if the machine has no dedicated WAN network interfaces. Otherwise you should apply geoblocking only to those WAN interfaces, so traffic from your LAN to the machine will bypass the geoblocking filter without any special rules for that.

`auto` will automatically detect LAN subnets (only use this if the machine has no dedicated WAN interfaces). `none` removes previously set LAN ip addresses.

**To enable or change the automatic update schedule:** `geoip-shell configure -s <"schedule_expression">`

_<details><summary>Example</summary>_

`geoip-shell configure -s "1 4 * * *"`

</details>

**To disable automatic updates of ip lists:** `geoip-shell configure -s disable`

**To update or re-install geoip-shell:** run the -install script from the (updated) distribution directory.

**To temporarily stop geoip-shell:** `geoip-shell stop`. This will kill any running geoip-shell processes, remove geoip-shell firewall rules and unload ip sets. To reactivate geoblocking, run `geoip-shell configure`.

**To uninstall:**

On OpenWrt, if installed via an ipk package: `opkg remove <geoip-shell|geoip-shell-iptables>`

_<details><summary>Examples of using the `configure` command:</summary>_

- configuring **inbound** geoblocking on a server located in Germany, which has nftables and is behind a firewall (no direct WAN connection), whitelist Germany and Italy and block all other countries:

`geoip-shell configure -r DE -i all -l auto -m whitelist -c "DE IT"`

- configuring **inbound** geoblocking on a router (which has a WAN network interface called `pppoe-wan`) located in the US, blacklist Germany and Netherlands and allow all other countries:

`geoip-shell configure -m blacklist -c "DE NL" -r US -i pppoe-wan`

</details>

## **Outbound geoblocking**

When using the `geoip-shell configure` command, if direction is not specified, direction-specific options apply to the **inbound** geoblocking direction.

Direction-specific options are `-m <whitelist|blacklist|disable>`, `-c <country_codes>`, `-p <ports>`. To specify direction, add `-D <inbound|outbound>` before specifying options for that direction.

So to configure outbound geoblocking, use same commands as described in the [Usage](#usage) section above, except add the `-D outbound` option before any direction-specific options.

Examples:

**To enable and configure outbound geoblocking:**

`geoip-shell configure -D outbound -m <whitelist|blacklist>`.

**To configure geoblocking mode for both inbound and outbound directions:**

`geoip-shell configure -D inbound -m <whitelist|blacklist> -D outbound -m <whitelist|blacklist>`

To configure **inbound and outbound** geoblocking, whitelisting Germany and Italy and blocking all other countries for incoming traffic, blacklisting France for outgoing traffic:

`geoip-shell configure -D inbound -m whitelist -c "DE IT" -D outbound -m blacklist -c FR`

**To change protocols and ports outbound geoblocking applies to:**

`geoip-shell configure -D outbound -p <[tcp|udp]:[allow|block]:[all|<ports>]>`

## **Pre-requisites**
- **Linux**. Tested on Debian-like systems and on OPENWRT, should work on any desktop/server distribution and possibly on some other embedded distributions.
- **POSIX-compliant shell**. Works on most relatively modern shells, including **bash**, **dash**, **ksh93**, **yash** and **ash** (including Busybox **ash**). Likely works on **mksh** and **lksh**. Other flavors of **ksh** may or may not work _(please let me know if you try them)_. Does **not** work on **tcsh** and **zsh**.

- **nftables** - firewall management utility. Supports nftables 1.0.2 and higher (may work with earlier versions but I do not test with them).
- OR **iptables** - firewall management utility. Should work with any relatively modern version.
- for **iptables**, requires the **ipset** utility - install it using your distribution's package manager
- standard Unix utilities including **tr**, **cut**, **sort**, **wc**, **awk**, **sed**, **grep**, **pgrep**, **pidof** and **logger** which are included with every server/desktop linux distribution (and with OpenWrt). Both GNU and non-GNU versions are supported, including BusyBox implementation.
- **wget** or **curl** or **uclient-fetch** (OpenWRT-specific utility).
- for the autoupdate functionality, requires the **cron** service to be enabled.
- for the MaxMind source, requires the utilities: `unzip`, `gzip`, `gunzip` (`apt install unzip gzip`)

## **Notes**
For some helpful notes about using this suite, read [NOTES.md](NOTES.md).

## **In detail**
For specifics about each script, read [DETAILS.md](DETAILS.md).

## **OpenWrt**
For information about OpenWrt support, read the [OpenWrt README](OpenWrt-README.md).

## **Privacy**
geoip-shell does not share your data with anyone.
If you are using the ipdeny or the maxmind source then note that they are a 3rd party which has its own data privacy policy.

