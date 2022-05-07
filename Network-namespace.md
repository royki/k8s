# Network-Namespace

- Create Network-Namespace
- Create Bridge Network/Interface
- Create VETH Pairs(Pipe, Virtual Cable)
- Attach vEth to Namespace
- Attach other vEth to Bridge
- Assign IP Address
- Enable NAT - IP Masquerade (For external communications)

## Local Network/ Network-Namespace / Local-Private-Network

- List of network interface
  - ip link
  - ip a

- List of network-namespaces (local-private-network)
  - ip netns show
  - ip netns

- Add a network namespace
  - sudo ip netns add red (/var/run/netns/red)
  - sudo ip netns add blue (/var/run/netns/blue)
  - sudo cat /var/run/netns/blue

- Execute inside the network namespace to list interface, arp etc
  - sudo ip netns exec red ip link
  - sudo ip -n red link (same as above)
  - sudo ip netns exec red arp (list arp)

- List of arp
  - sudo arp

- Create virtual-interface or pipe
  - sudo ip link add veth-red type veth peer name veth-blue

- Delete virtual-interface or pipe
  - sudo ip -n red link del veth-red
  - If one side is deleted, other connected one will automatically be deleted.

- Attach virtual-interface to the network namespace
  - sudo ip link set veth-red netns red
  - sudo ip link set veth-blue netns blue

- Assign IP addresses to the virtual-interface (pipe)
  - ip -n red addr add 192.168.15.1 dev veth-red
  - ip -n blue addr add 192.168.15.2 dev veth-blue

- Now turn up the virtual-interface (devices)
  - ip -n red link set veth-red up
  - ip -n blue link set veth-blue up

- Ping from red namespace to blue namespace
  - ip netns exec red ping 192.168.15.2

## Attache multiple network namespaces using virtual-switch

- If we have multiple network namespaces, to connect them together, we need virtual-switch.
- To create a virtual network, create a virtual-switch
  - Create virtual-switch by Linux Bridge, Open vSwitch etc
  - ip link add v-net-0 type bridge
  - For the host machine it is just an interface

- Connect the virtual-switch to the namespaces
  - Need to create virtual-interface (pipe)
  - ip link add veth-red type veth peer name veth-red-br
  - ip link add veth-blue type veth peer name veth-blue-br

- Attach virtual-interface to the namespace & virtual-switch
  - attache to the namespace: ip link set veth-red netns red
  - attache to the virtual-switch: ip link set veth-red-br master v-net-0
  - attache to the namespace: ip link set veth-blue netns blue
  - attache to the virtual-switch: ip link set veth-blue-br master v-net-0

- Assign IP addresses to the virtual-interface (pipe)
  - ip -n red addr add 192.168.15.1 dev veth-red
  - ip -n blue addr add 192.168.15.2 dev veth-blue

- Now turn up the virtual-interface (devices)
  - ip -n red link set veth-red up
  - ip -n blue link set veth-blue up

- Assign an ip address to the virtual-switch
  - ip addr add 192.168.15.5/24 dev v-net-0

- This is a private network within a host machine (local-private-network)

## Access other machine/host from a local private network

- Localhost where all the namespaces are on. So we can connect a local private network to an external machine or network.
- So the Localhost is here the Gateway that connects between local private network & external network.
  - Add a route entry to the blue namespace to say route all traffic to the 192.168.1.0/24 through the gateway (using localhost) at 192.168.15.5
    - ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5
    - Add NAT gateway to tell the interfaces to direct the network. (NAT Enable on Host, acting as a gateway)
      - iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE
      - ip netns exec blue ping 192.168.1.3

- Connect local network-namespace(local-private-network) to the internet
  - Add a default Gateway to specify a host (as host can connect to any network, so host act as a default Gateway)
  - ip netns exec blue ip route add default via 192.168.15.5
  - ip netns exec blue ping 8.8.8.8

## Access local-private-network from external host/machine
  
- Using IP Route Entry or Post Forwarding
- Using Port Forwarding
  - iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT

- What is the Networking Solution used by this cluster?
- `/etc/cni/net.d/`
- How many weave agents/peers are deployed in this cluster?
- `kubectl get pods --all-namespaces`
- Identify the name of the bridge network/interface created by weave on each node
- ip link
- What is the POD IP address range configured by weave?
- `ip addr show weave`
- Deploy Weave on cluster
- `kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`

## Kubernetes Networking Model

- Every POD should have an IP Address
- Every POD should be able to communicate with every other POD in the same node.
- Every POD should be able to communicate with every other POD on other nodes without NAT.

## In Linux to create a private newtwork model

- ip link add v-net-0 type bridge
- ip link set dev v-net-0 up

- ip addr add 192.168.15.5/24 dev v-net-0

- ip link add veth-red type veth peer name veth-red-br

- ip link set veth-red netns red

- ip -n red addr add 192.168.15.1 dev veth-red

- ip -n red link set veth-red up

- ip link set veth-red-br master v-net-0

- ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5

- iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE
