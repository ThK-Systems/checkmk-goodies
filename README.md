# checkmk-goodies
Goodies for [checkmk](https://checkmk.com)

### Disclaimer
- It is only tested under Debian Linux (12) 'bookworm' with check_mk 2.2.
- It comes with no warranty at all!!!
- Support will be not available.
- Contributions are welcome

### Install one or more scripts (e.g. 'gluster')
1. Download the script(s) you want to use (or clone the github-repo)
2. Put it/them into the folder `/usr/lib/check_mk_agent/local/` 
3. Make the script(s) executable (0755)
4. Rescan the host in check_mk for new services

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


## hdidle

__Requirements__:
- smartctl

__Description__

It monitors the idle state of one or more disks.

Set the disks to monitor at the head of the script

__Notes__

- It never goes to warning or critical state. It just monitors the idle state.
- It is only tested for ata devices