# procServMgr

## Name
procServMgr

## Description
procServMgr is intended to be a control script for running "soft" iocs with procServ. It is designed to be run from the command line and the cron facility.  

## Dependencies

- **ksh** - Korn Shell<br/>
    Ususally provided by the the pdksh or ksh packages

- **procServ**<br/>
    Project Page for procServ - https://sourceforge.net/projects/procserv

- **caRepeater**<br/>
    Provided by the EPICS distribution - https://epics.anl.gov/download/index.php

## procServMgr configuration file
The standard configuration file read by procServMgr is procServ.conf. You can overide this on the command line if you want to test an alternate configuation or the file is in a different place (like a DEV environment). The file is self documenting.

```
# procServ.conf
# 
# This is the configuration file for the procServMgr controller script.  
# 
#-------------------
# Help
#-------------------
# procServMgr is self documenting with the help option:
#  procServMgr -h
#
#
#-------------------
# Valid Hosts
#-------------------
# Valid opsbat hosts where procServMg is running in this fiefdom: 
# host0  RHEL 7WS
# host1  RHEL 6WS
# host2  RHEL 7WS
# host3  RHEL 9WS
# 
#-------------------
# Testing
#-------------------
# You can test the options using the test option.
# Examples:
#  procServMgr test
#  procServMgr -i iocname test
#
# You can also use your own private config file. You can use this file as a template.
# Examples:
#  procServMgr -c /tmp/myconfigfile test
#  procServMgr -c /tmp/myconfigfile start
#  procServMgr -c /tmp/myconfigfile stop
#
#-------------------
# Accessing your ioc
#-------------------
# You can use the softioc_console script to access your ioc.  You must run this command
# from the same fiefdom as the host (i.e. from an ops system for opsbat3).
#  softioc_console iocname
#
#-------------------
# Field Separator
#-------------------
# The Field Separator  is a colon ":" which is required between all fields even empty ones.
#
#-------------------
# Field Descriptions
#-------------------
# iocname - name of the softioc, which is also it's directory name in the ioc directory.
# hostname - the host where you want the soft to run.  Note the host must in the lisb above.
# port - a unique port on which to run.  THis must be above 20000.
# status - whether the ioc should be running or not.  Only enabled and disabled are valid.
# procServ options - options to pass directly to the procServ dameon.
# startup options - options to be passed to the softioc itself.
# 
#-------          :--------     :-----   :-------        :----------------               :---------------
#iocname          :hostname     :port    :status         :procServ options               :startup options
#-------          :--------     :-----   :-------        :----------------               :---------------
iocsofthrtbt 	:host0		:20000	:disabled	:-L /tmp/iocsoftthrtbt.log	:-x test options
iocsofthrtbt2	:host1		:20001	:disabled	:-L /tmp/iocsoftthrtbt2.log     :-x @test 	
iocsoftlemtest	:host3		:20003	:enabled	:-L /tmp/iocsoftlemtest.log     :                         
```

## Installation

THere are only two files required for procServMgr to run. The procServMgr script itself and the config file procServ.conf

1. Modify the proServMgr file and update the following variables to match your installation.
```
     SCRIPT_USER=someuser           - user that softIOC processes will run as.
     IOC_DIR=/cs/iocs               - Path to where the softIOC directories are stored.
     APP_DIR=/opt/procServ          - Path to procServ installation.
     CONFIG_DIR=$IOC_DIR/procServ   - Path to the directory containing the procServ.conf config file.
```
2. Modify the provided procServ.conf file to suit your neeeds.  Note that the field separators are ":" and should not be modified.
Example:
```
#-------          :--------     :-----   :-------        :----------------               :---------------
#iocname          :hostname     :port    :status         :procServ options               :startup options
#-------          :--------     :-----   :-------        :----------------               :---------------
iocsoftmag         :opsbat9      :20000   :enabled        :-L /cs/op/iocs/iocsoftmag/log  :
iocsoftinjdec      :opsbat9      :20001   :enabled        :                               :

```

## Usage
```
usage: procServMgr [-h] [-i iocname] [-c cfg_file] [-d ioc_dir]
       start|stop|restart|check|status|test

[-i] Specify the ioc. If not specified, the script assumes all iocs
[-c] Specify the configuration file.
[-d] Specify the IOC directory.
[-h] Print Help

start      - start iocs specified in config file
stop       - stop iocs specified in config file
check      - start any iocs not running that are configured to run
restart    - stop and start iocs specified in config file
status     - dump a short status screen showing current status
test       - show how each IOC would be invoked but don't do anything

Examples:
procServMgr start  - start all iocs enabled in the config file
procServMgr stop   - stop all iocs enabled in the config file
procServMgr status - show status for all iocs
procServMgr -c /tmp/testconfig.conf check - use alternate config file
procServMgr -i iocsofthrt start - start ioc iocsofthrt only
procServMgr -i iocsofthrt test - show  start command for iocsofthrt

CONFIGURATION FILES:
--------------------
/cs/op/iocs/procServ/procServ.conf
```

## procServMgr Testing
You can test the options and your sofIOC using the test option.
```
Examples:
   procServMgr test
   procServMgr -i iocname test

```
You can also use your own private config file. You can use the standard configuration file as a template.
```
Examples: 
   procServMgr -c /tmp/myconfigfile test
   procServMgr -c /tmp/myconfigfile start
   procServMgr -c /tmp/myconfigfile stop
```
## Accessing your IOC - softioc_console
You can use the _softioc_console_ script to access your ioc. You must run this command from the same enclave as the host since it will read the procServ.conf file and ssh to the appropraite host.
```
usage: softioc_console [-h] iocname

[-h] Print Help

Examples:
softioc_console [-h]
softioc_console iocsofthrt - start a console on ioc iocsofthrt
```
## Setting Up ProcServMgr on a Host

1. **Create a cron entry to run as the _SCRIPT_USER_.**

Ordinarily, procServMgr runs as and individual user via cron. This might be different depending on the environment. You need to consider this when creating the crontab entry.

The standard cron entry runs procServMgr every 5 minutes. This may need to be reduced depending on the circumstances. The PATH variable also needs to be modified to add the location of caRepeater if it is not in the standard PATH.

```
sudo vi /etc/crontab

PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/csite/pubtools/bin:/cs/prohome/bin

# procServMgr
*/5 * * * * someuser /usr/csite/pubtools/procServ/pro/bin/procServMgr check >> /cs/op/iocs/procServ/logs/softioc_`hostname`.log 2>&1
```

2.  **Raise the Limits for Number of Processes Per User**

The default limits on the number of processes/threads a standard user can run concurrently can cause problems as all soft IOCs are run as the same user. Raise the limits from 1024 to 8192. You must change it in the following two locations for a RHEL style system.

Note: To get a sense of the current number of processes/threads that are being run by vxuser, run this command.

`
ps -efL | grep vxuser | wc -l
`

```
sudo vi /etc/security/limits.conf
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#@student        -       maxlogins       4

*       soft    nproc           8192
*       hard    nproc           8192
# End of file

sudo vi /etc/security/limits.d/95-procserv.conf
*          soft    nproc     8192
```

3. **Fix UDP Name Resolution with Firewall Rules**

Running multiple IOCs on one host has an annoying side effect: Clients that are using that host's IP address in their EPICS_CA_ADDR_LIST with EPICS_CA_AUTO_ADDR_LIST=NO will only reach one of the IOCs - usually the one that was started last. All clients have to use broadcasts to reach all IOCs.

The same is true for CA Gateway machines that are set up in a way that makes multiple Gateway processes serve channels into the same network.

The reason is that the kernel delivers UDP broadcasts to all processes that are listening to the IP port, while UDP unicast messages will only be delivered to one of those processes.

A simple and effective trick to deliver messages to all IOCs instead of one is to use iptables/firewalld.  Below is an example setup for a RHEL6/7 system and alternately a RHEL9 system where firewalld must be disabled.

**RHEL 6/7**
```
# Enable iptables
   chkconfig iptables on
# Start iptables
   service iptables start
# Clar all rules first
   iptables -F
   iptables -X
   iptables -t nat -F
   iptables -t nat -X
   iptables -t mangle -F 
   iptables -t mangle -X
   iptables -P INPUT ACCEPT
   iptables -P FORWARD ACCEPT
   iptables -P OUTPUT ACCEPT
# Create the redirect rule substituting the Host IP Address and Network Broadcast Address
   iptables -t nat -A PREROUTING -d <HOST_IP_ADDRESS> -p udp --dport 5064 -j DNAT --to-destination <NETWORK_BROADCAST_ADDRESS>
# Save the configuration
   iptables-save > /etc/sysconfig/iptables
```

**RHEL 9**

```
systemctl stop firewalld
systemctl disable firewalld
systemctl mask firewalld
dnf install -y iptables-services iptables-utils``

# Enable iptables
   chkconfig iptables on
# Start iptables
   service iptables start
# Clar all rules first
   iptables -F
   iptables -X
   iptables -t nat -F
   iptables -t nat -X
   iptables -t mangle -F 
   iptables -t mangle -X
   iptables -P INPUT ACCEPT
   iptables -P FORWARD ACCEPT
   iptables -P OUTPUT ACCEPT
# Create the redirect rule substituting the Host IP Address and Network Broadcast Address
   iptables -t nat -A PREROUTING -d <HOST_IP_ADDRESS> -p udp --dport 5064 -j DNAT --to-destination <NETWORK_BROADCAST_ADDRESS>
# Save the configuration
   iptables-save > /etc/sysconfig/iptables

```


## License
MIT.

## Project status
procSrvMgr has been in use since 2008 withb only minor fixes.  Is is still being improved and devloped.
