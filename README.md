# checkmk-goodies
Goodies for checkmk

## Disclaimer
It is tested under Debian Linux bookworm (12) with check_mk 2.2.
It comes wit no warranties at all!!!
No support is available.

## Install a single script
1. Download the script(s) you need to use into the folder `/usr/lib/check_mk_agent/local/`
2. Make the script executable (0755)
3. Rescan the host in check_mk for new services

## gluster
Based on https://exchange.checkmk.com/p/gluster

Notifies only, when a healing is necessary after a certain period of time has passed. 
A warning is generated first and then a critical status.

The time periods can be adjusted in the head of the script.
