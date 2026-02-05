# checkmk-goodies
Goodies for [checkmk](https://checkmk.com)

### Disclaimer
- It is only tested under Debian Linux (12/13) 'bookworm'/'trixie' with check_mk 2.2, 2.3, 2.4.
- It comes with no warranty at all!!!
- Support will be not available.
- Contributions are welcome

### Install one or more scripts (e.g. 'gluster')
1. Download the script(s) you want to use (or clone the github-repo)
2. Put it/them into the folder `/usr/lib/check_mk_agent/local/` 
3. Make the script(s) executable (0755)
4. Rescan the host in check_mk for new services

---

## crowdsec

__Requirements__:
- crowdsec

__Description__

This local plugin monitors **CrowdSec**. 

It exposes two services: **`crowdsec lapi`** (LAPI reachability) and **`crowdsec decisions`** (number of active decisions). 

LAPI reachability is determined via `cscli lapi status` and reflects whether the local system can successfully interact with the Local API.  

Only **active decisions** are monitored, as they represent enforced security actions and are meaningful for monitoring.  
Decisions are counted using `cscli decisions list`.  Warning and critical thresholds for decisions can be configured via environment variables.  
The plugin uses the native Checkmk local check format and does not rely on Nagios-style pipes.  
It is designed to be simple, robust, and semantically quiet in production environments.

__Configuration__

You can write the configuration to `/etc/check_mk_agent.d/crowdsec.conf`
- `CROWDSEC_DECISIONS_WARN=5`
- `CROWDSEC_DECISIONS_CRIT=20`

---

## docker_containers

__Requirements__:
- docker

__Description__

This Python-based Checkmk local plugin monitors Docker containers using cgroup v2 data and `docker inspect`, providing state, memory, and CPU services per container.

Memory and swap usage are reported in megabytes, with thresholds derived from container limits, while the service output additionally shows host-level RAM and swap usage for better context.  

CPU usage is calculated via cgroup deltas, respects CPU limits from `cpu.max`, and indicates throttling when the container is actively constrained.  

The script is designed to be deterministic, fast, and suitable for frequent polling (e.g. every five minutes) without generating unnecessary alert noise.


__Configuration__

You can write the configuration to `/etc/check_mk_agent.d/docker_container_memory.conf`

- `CONTAINERS` **(mandatory!!!)** - Space-separated list of Docker container names to monitor.
- `MEM_WARN`  - Global warning threshold for RAM usage in percent of the container memory limit.
- `MEM_CRIT`  - Global critical threshold for RAM usage in percent of the container memory limit.
- `SWAP_WARN` - Global warning threshold for swap usage in percent of the effective swap limit.
- `SWAP_CRIT` - Global critical threshold for swap usage in percent of the effective swap limit.
- `CPU_WARN`  - Global warning threshold for CPU usage in percent of the container CPU limit.
- `CPU_CRIT`  - Global critical threshold for CPU usage in percent of the container CPU limit.

For all threshold options, per-container overrides are supported by appending the container name as a suffix:

- `MEM_WARN.<container>`
- `MEM_CRIT.<container>`
- `SWAP_WARN.<container>`
- `SWAP_CRIT.<container>`
- `CPU_WARN.<container>`
- `CPU_CRIT.<container>`

If a per-container option is not defined, the global value is used.  
If neither is present, the script’s built-in default is applied.

---

## gluster
(Partially based on the gluster checkmk plugin ( https://exchange.checkmk.com/p/gluster ) from lpwevers.)

__Requirements__:
- glusterfs (of course)
- pidof

__Description__

It monitors glusterfs demons (running-state) and peers (connection-state) and volumes (needed healings).
Also, the needed healings of a volume are available as a metric for this volume.

At the head of the script, you can configure some time periods and other values.

__Note__

*The changes to the checkmk plugin from lpwevers are motivated as follows: The plugin immediately sets the service level to 'Critical', as soon as at least one file needs to be healed.*
*However, this seems to be a normal and regular process of glusterfs - at least if you use it under proxmox, as I do - which in my observation should not immediately trigger an alert in the monitoring.*

*The 'gluster' script therefore takes over all checks (except for the quotas) from lpwevers´ plugin and supplements the volume healing check with the option of first triggering a warning and then a critical alert after a certain (and configurable) time or a certain (and configurable) count of healings.*

To use this script, you have to disable lpwevers´ plugin.

---

## zram

__Requirements__:
- zram-tools

__Description__

This Checkmk local plugin monitors **zram-based swap** on Linux systems with a clear separation between **capacity, usage, and efficiency**.  

It is designed for environments where zram acts as a **pressure buffer**, not as a primary memory resource.  

The plugin exposes two services: `zram used` and `zram compression`, each representing a single, well-defined aspect of zram behavior.  

The main service, `zram used`, tracks the **logically used (uncompressed) data volume** and is the only stateful check.  Alerting is based on **relative fullness** of zram, using configurable percentage thresholds rather than fixed sizes. This makes the check independent of the actual zram configuration and robust across hosts.  

The `zram compression` service shows the current **compression ratio** as an informational metric.  Physical RAM consumption of zram is shown in the service output for context but is not used for alerting.  
