# poc

Install the necessary software:

a. Install containerlab by following the instructions here: https://containerlab.srlinux.dev/install/

b. Install gobgp by following the instructions here: https://osrg.github.io/gobgp/doc/installation/

Create a containerlab topology file with the following contents:

```
# topo.yml
---
topology:
  nodes:
    - name: bgp1
      kind: gobgp
      image: gobgp/gobgp
      command: ["gobgpd", "-f", "/etc/gobgpd.conf"]
      config:
        asn: 65000
        router-id: 192.168.1.1
        listen: 0.0.0.0:179
        neighbors:
          - address: 192.168.1.2
            remote-as: 65001
    - name: bgp2
      kind: gobgp
      image: gobgp/gobgp
      command: ["gobgpd", "-f", "/etc/gobgpd.conf"]
      config:
        asn: 65001
        router-id: 192.168.1.2
        listen: 0.0.0.0:179
        neighbors:
          - address: 192.168.1.1
            remote-as: 65000
          - address: 192.168.1.3
            remote-as: 65002
    - name: bgp3
      kind: gobgp
      image: gobgp/gobgp
      command: ["gobgpd", "-f", "/etc/gobgpd.conf"]
      config:
        asn: 65002
        router-id: 192.168.1.3
        listen: 0.0.0.0:179
        neighbors:
          - address: 192.168.1.2
            remote-as: 65001
          - address: 192.168.1.4
            remote-as: 65003
    - name: bgp4
      kind: gobgp
      image: gobgp/gobgp
      command: ["gobgpd", "-f", "/etc/gobgpd.conf"]
      config:
        asn: 65003
        router-id: 192.168.1.4
        listen: 0.0.0.0:179
        neighbors:
          - address: 192.168.1.3
            remote-as: 65002
          - address: 192.168.1.5
            remote-as: 65004
    - name: bgp5
      kind: gobgp
      image: gobgp/gobgp
      command: ["gobgpd", "-f", "/etc/gobgpd.conf"]
      config:
        asn: 65004
        router-id: 192.168.1.5
        listen: 0.0.0.0:179
        neighbors:
          - address: 192.168.1.4
            remote-as: 65003
  links:
    - endpoints: ["bgp1:eth1", "bgp2:eth1"]
    - endpoints: ["bgp2:eth2", "bgp3:eth1"]
    - endpoints: ["bgp3:eth2", "bgp4:eth1"]
    - endpoints: ["bgp4:eth2", "bgp5:eth1"]
```


```
sudo containerlab deploy -t topo.yml
```

Once the topology is deployed, you can log in to any of the gobgp containers and add the server IPs to be broadcasted. For example, to add the IP addresses 10.0.0.1, 10.0.0.2, 10.0.0.3, 10.0.0.4, and 10.0.0.5, you can do the following:

```
sudo containerlab enter bgp1
```

Use the gobgp command to add the server IPs:

```
gobgp global rib add 10.0.0.1/32
gobgp global rib add 10.0.0.2/32
gobgp global rib add 10.0.0.3/32
gobgp global rib add 10.0.0.4/32
gobgp global rib add 10.0.0.5/32
```

Verify that the server IPs are being advertised to all the other containers using the gobgp command. For example, to check the routes received by bgp2, you can do the following:

a. Log in to bgp2 container:

```
sudo containerlab enter bgp2
gobgp global rib

```
