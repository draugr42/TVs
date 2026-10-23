# TVs: clean up smart TV network traffic and bloat

Context file for Claude Code. Drop it in the project root and run `claude` from the Mac, on the home WiFi.

## Owner and style
Sean: software engineering director, homeowner, technical. Skip the basics, be direct, call out risks. Recommend, don't survey.

## Goal
Smart TVs on the home WiFi generate a lot of traffic (ACR telemetry, ads, vendor services) and run bloated software. Reduce it, measure the result, and isolate the TVs from the rest of the network.

## Ground rules
- Only scan and probe devices on my own network.
- Read-only discovery first. Ask before changing any router, DNS, or firewall config, or any setting on a TV.
- Never expose admin UIs (router, Pi-hole, AdGuard) to the internet.
- The Mac is the workstation, not the DNS or firewall box. It sleeps and moves, and the house loses DNS when it does. Run DNS and filtering on a Pi, a mini PC, or the router.
- Measure before and after. Don't claim a change helped without numbers.

## Why the Mac can't see everything
WiFi is switched. A client sees only its own traffic plus broadcast and multicast, so the Mac cannot sniff the TVs' traffic by default. To see TV traffic we need a vantage point on the path:
1. Router or DNS logs (Pi-hole / AdGuard Home query log). Easiest, and usually enough.
2. Router packet capture or a mirror port (OpenWrt, pfSense, OPNsense, UniFi).
3. The Mac as a gateway or bridge for the TVs. A last resort, temporary only.

## Options, in priority order
1. **TV settings:** turn off ACR / viewing-information services, interest-based ads, voice and "smart" features. Free, but settings can revert after firmware updates.
2. **DNS filtering:** Pi-hole or AdGuard Home with vendor blocklists. Force port 53 (and DoT/DoH where possible) to the local resolver with a firewall redirect, because many TVs hardcode 8.8.8.8. Expect some breakage (Netflix, YouTube) and tune.
3. **Isolation:** put the TVs on an IoT VLAN or SSID with internet-only access and no LAN access. Allow mDNS reflection only if casting is needed. Needs VLAN-capable router or APs.
4. **Dumb TV:** block the TV from the internet or keep it off WiFi, and use an Apple TV or Nvidia Shield for streaming. This is the cleanest option and gives the best update support.
5. **QoS / rate limits:** fixes bandwidth hogging only, with no privacy benefit.

## Open questions (fill in)
- TV brands, models, and count:
- Router / APs (model, firmware, VLAN support, DNS settings):
- Goal: privacy, bandwidth, or both?
- Willing to add a streaming box (Apple TV, Shield)?
- Always-on box available for Pi-hole / AdGuard (Pi, mini PC)?

## Plan
1. **Inventory (read-only):** ARP table, `nmap -sn`, mDNS browse (`dns-sd -B _services._dns-sd._udp`), then `nmap -sV` on the TVs only. Record IP, MAC, vendor, open ports. Store the output in `inventory/`.
2. **Baseline:** collect DNS query logs or a capture for 24-48h. Summarize per TV: top domains, query counts, bytes if available. Store in `baseline/`.
3. **Apply the settings and DNS changes** (options 1 and 2), one at a time.
4. **Re-measure** and compare to baseline.
5. **Isolate** (option 3) if the router supports it.
6. **Reassess** whether a streaming box (option 4) is worth it.

## Repo layout (proposed)
```
inventory/      discovery output (gitignore anything with MACs if the repo goes public)
baseline/       before/after measurements
scripts/        discovery and analysis scripts
config/         AdGuard/Pi-hole lists, firewall and VLAN rules
docs/           setup notes, per-brand ACR settings
```

## First task for Claude
Write `scripts/discover.sh` (macOS: bash + `arp`, `nmap`, `dns-sd`), read-only, that outputs a table of devices on the subnet with IP, MAC, vendor, hostname, and open ports. Tell me what to install via Homebrew first (`brew install nmap`). Don't run anything that changes network state.
