# M-CORD utility #1: Setting flow rules on toward VTN.

This tools is to set up flow rules between SPGWU and eNB toward VTN, when there is no ONOS-Fabric. When ONOS-Fabric has been set up well, this tools is unnecessary since all rules will be set up automatically.

## 1. Before you use this,
At first, go to SPGWC VNF and insert below.
```
$ sudo pkill -9 -ef build
```
Likewise, go to SPGWU and insert above.

## 2. How to use?

##### First, access head node.

##### Then, go to SPGWC synchronizer
```
$ docker exec -it mcordng40_vspgwc-synchronizer_1 bash
```
##### Get source code from GitHub

```
# cd ~
# git clone git@github.com:woojoong88/mcord_flow_wo_fabric.git
```

##### Open setup_flow_playbook.yaml

Change some variables as below
```
---
- hosts: localhost
  vars:
    - sgi_as_ip: <eNB's IP address connected to sgi_network>
    - sgi_spgwu_ip: <SGPWU's IP address connected to sgi_network>
    - ip_addr_pool: "16.0.0.0"
    - ip_pool_mask: "255.0.0.0"
    - ip_pool_mask_int: "8"
    - enb_sw_ip: <IP address of br-int connected with eNB>
    - spgwu_sw_ip: <IP address of br-int connected with SPGWU>
    - enb_compute_name: <Compute node's name which runs eNB>
    - spgwu_compute_name: <Compute node's name which runs SPGWU>
  roles:
      - rule_setup
```

**Note:**
+ You can find `sgi_as_ip` and `sgi_spgwu_ip` variables with `nova list --all-tenants` command.
For example, go to Head node and put below
```
Go to Head node
$ source /opt/cord_profile/admin-openrc.sh
$ nova list --all-tenants
+--------------------------------------+-----------------+--------+------------+-------------+---------------------------------------------------------------------------------------------+
| ID                                   | Name            | Status | Task State | Power State | Networks                                                                                    |
+--------------------------------------+-----------------+--------+------------+-------------+---------------------------------------------------------------------------------------------+
| ba1c80ca-54c1-4c2c-b4ba-3e1ec734979b | mysite_venb-4   | ACTIVE | -          | Running     | s1u_network=111.0.0.4; sgi_network=115.0.0.4; management=172.27.0.6; s11_network=112.0.0.5  |
| 573ee863-f89e-4c5d-9699-faa3c27a5703 | mysite_vspgwc-6 | ACTIVE | -          | Running     | management=172.27.0.5; spgw_network=117.0.0.4; s11_network=112.0.0.4                        |
| cd6b93a8-f5d6-46e7-968a-419ecb574b0b | mysite_vspgwu-5 | ACTIVE | -          | Running     | spgw_network=117.0.0.5; s1u_network=111.0.0.5; sgi_network=115.0.0.5; management=172.27.0.7 |
+--------------------------------------+-----------------+--------+------------+-------------+---------------------------------------------------------------------------------------------+
```
In this situation, `sgi_as_ip: 115.0.0.4` and `sgi_spgwu_ip: 115.0.0.5`.

+ You can find `enb_compute_name` and `spgwu_compute_name` by using CORD UI. Go to `http://<Head node IP>/xos` and click `Instance` tab.

+ You can find `enb_sw_ip` with `ip a list br-int` command. First, go to the compute node which contains eNB. Then, insert below.
```
Go to Head node
$ ssh ubuntu@<Compute node's name which runs eNB>
$ ip a list br-int
16: br-int: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
    link/ether 00:1e:67:d2:ee:b3 brd ff:ff:ff:ff:ff:ff
    inet 10.6.1.2/24 scope global br-int
       valid_lft forever preferred_lft forever
    inet 172.27.0.1/24 scope global br-int
       valid_lft forever preferred_lft forever
    inet6 fe80::21e:67ff:fed2:eeb3/64 scope link
       valid_lft forever preferred_lft forever
```
In this situation, 10.6.1.2 is the IP address which is for `enb_sw_ip`.

+ You can find `spgwu_sw_ip` with `ip a list br-int` command. First, go to the compute node which contains SPGWU. Then, insert below.
```
Go to Head node
$ ssh ubuntu@<Compute node's name which runs SPGWU>
$ ip a list br-int
16: br-int: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
    link/ether 00:1e:67:d2:ec:7e brd ff:ff:ff:ff:ff:ff
    inet 10.6.1.3/24 scope global br-int
       valid_lft forever preferred_lft forever
    inet 172.27.0.1/24 scope global br-int
       valid_lft forever preferred_lft forever
    inet6 fe80::21e:67ff:fed2:ec7e/64 scope link
       valid_lft forever preferred_lft forever
```
In this situation, 10.6.1.3 is the IP address which is for `spgwu_sw_ip`.

+ After that, save this file.

##### Run
```
Go to Head node
$ docker exec -it mcordng40_vspgwc-synchronizer_1 bash
# cd ~/mcord_flow_wo_fabric
# ansible-playbook setup_flow_playbook.yaml
```

##### Validation
First, go to the VTN UI, `http://<Head node IP>/vtn/onos/ui`, and go to Devices tab. Then, you can see one or more switches. Select one of switches and click `Flow` icon. If you can find priority `7001`, then flows are successfully set up for the selected switch. You should repeat above for all switches.

## 3. After run this software,
First, go to SPGWU VNF and insert below.
```
Go to SPGWU VNF
$ sudo su
# cd /root/ngic/dp
# source ../setenv.sh
# make clean
# make
# ./udev.sh > results &
```

You should see below output then go further.
```
DP: RTE NOTICE enabled on lcore 1
DP: RTE INFO enabled on lcore 1
DP: RTE NOTICE enabled on lcore 0
DP: RTE INFO enabled on lcore 0
DP: RTE NOTICE enabled on lcore 3
DP: RTE INFO enabled on lcore 3
DP: RTE NOTICE enabled on lcore 5
DP: RTE INFO enabled on lcore 5
DP: RTE NOTICE enabled on lcore 4
DP: RTE INFO enabled on lcore 4
API: RTE NOTICE enabled on lcore 4
API: RTE INFO enabled on lcore 4
DP: RTE NOTICE enabled on lcore 6
DP: RTE INFO enabled on lcore 6
DP: RTE NOTICE enabled on lcore 2
DP: RTE INFO enabled on lcore 2
```

Next, go to SPGWC VNF and insert below.
```
Go to SPGWC VNF
$ sudo su
# cd /root/ngic/cp
# source ../setenv.sh
# make clean
# make
# ./run.sh > results &
```

Finally, go to SPGWU VNF again and open `/root/ngic/dp/results`. If you can see below output at the bottom of this file, all VNFs are set up successfully.
```
DP: MTR_PROFILE ADD: index 2 cir:125000, cbd:3072, ebs:3072
DP: MTR_PROFILE ADD: index 3 cir:250000, cbd:3072, ebs:3072
DP: MTR_PROFILE ADD: index 4 cir:500000, cbd:3072, ebs:3072
```