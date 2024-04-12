# checkmk-goodies
Goodies for [checkmk](https://checkmk.com)

## Disclaimer
- It is tested under Debian Linux bookworm (12) with check_mk 2.2.
- It comes wit no warranties at all!!!
- Support is not available.

## Install a single script (e.g. gluster)
1. Download the script(s) you need to use (or clone the github-repo).
2. Put it into the folder `/usr/lib/check_mk_agent/local/` 
3. Make the script executable (0755)
4. Rescan the host in check_mk for new services

## gluster
Based on https://exchange.checkmk.com/p/gluster

It changed status in cause of a necessary healing after a certain period of time has passed. 
A warning is generated first and then a critical status.

The time periods can be adjusted in the head of the script.

*The change to the checkmk plugin from lpwevers is motivated as follows: The plugin immediately sets the service level to 'Critical', as soon as at least one file needs to be healed.*
*However, this seems to be a normal and regular process of glusterfs - at least if you use it under proxmox, as I do - which in my observation should not immediately trigger an alert in the monitoring.*

*The 'gluster' script therefore takes over all checks (except for the quotas) from lpwevers' plugin and supplements the volume healing check with the option of first triggering a warning and then a critical alert after a certain time or a certain count of healings.*
