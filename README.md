# wazuh-bastillebsd
Wazuh-bastilleBSD is a bastillebsd template used by deploy a testing Wazuh single infraestructure on FreeBSD. The principal goals are helps us to fast way install, configure and run wazuh-indexer (opensearch), wazuh-manager, logstash, filebeat and wazuh-dashboards (opensearch-dashboards + wazuh-kibana-app). Take on mind this container as is must be used by testing and it is not recommended for production because it has a minimal configuration for run wazuh. 

## Requeriments
Before of you can install wazuh using this template you need some initial configuration

#### Create a loopback interface
We can create it manually
```sh
# sysrc cloned_interfaces="lo1"
# sysrc ifconfig_lo1_name="bastille0"
# service netif cloneup
```
#### Enable Packet filter
We need add somes lines to /etc/rc.conf

```sh
# sysrc pf_enable="YES"
```
Edit /etc/pf.conf and modify like you want. Minimal working configuration could look at the following

```sh
ext_if="em0"
lo_if="bastille0"

set block-policy return
scrub in on $ext_if all fragment reassemble
set skip on lo

table <jails> persist
nat on $ext_if from <jails> to any -> ($ext_if:0)
rdr-anchor "rdr/*"

block in all
pass out quick keep state
antispoof for $ext_if inet
pass in inet proto tcp from any to any port ssh flags S/SA keep state

pass in quick on $ext_if inet proto { tcp udp } from any to any
pass out quick on $ext_if inet proto { tcp udp } from any to any

pass in quick on $lo_if inet proto { tcp udp } from any to any
pass out quick on $lo_if inet proto { tcp udp } from any to any
```
rdr-anchor section is necessary for use dynamic redirect from jails

Start packet filter

```sh
# service pf start
```
#### Bootstrap a FreeBSD version
Before you can begin creating containers, Bastille needs to "bootstrap" a release. It must be a version equal or lesser than your host version. In this example we will create a 13.1-RELEASE bootstrap

```sh
# bastille bootstrap 13.1-RELEASE
```
#### Create a lightweight container system
Create a container named wazuh with a private IP address 10.0.0.1

```sh
# bastille create wazuh 13.1-RELEASE 10.0.0.1
```
Now apply wazuh template to container

```sh
# bastille bootstrap https://github.com/alonsobsd/wazuh-bastillebsd
# bastille template wazuh wazuh-bastillebsd
```
