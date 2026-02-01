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

You can write the configuration to `/etc/check_mk/check_mk_agent.d/crowdsec.conf`
- `CROWDSEC_DECISIONS_WARN=5`
- `CROWDSEC_DECISIONS_CRIT=20`

---

## docker_container_memory

__Requirements__:
- docker

__Description__

## Docker Container Memory (Local Check)

This Checkmk local agent plugin monitors memory usage of all running Docker containers relative to their configured memory limits.

The check runs on the Docker host and uses `docker stats --no-stream` to collect current memory usage and limits. For each container with a defined memory limit, a dedicated Checkmk service is created.

Reported metrics include current memory usage, warning and critical thresholds (derived from the container limit), and the maximum memory limit. All values are reported as numeric perfdata suitable for graphing.

Containers without an explicit memory limit are automatically ignored.

__Configuration__

You can write the configuration to `/etc/check_mk/check_mk_agent.d/docker_container_memory.conf`
- `WARN_PCT` (default: 85)
- `CRIT_PCT` (default: 95)

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
