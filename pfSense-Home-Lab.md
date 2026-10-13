### Goal

- Build a segmented home lab network using pfSense as a firewall and router. Create isolated VLANs for Trusted, IoT, and Guest zones.

### What I Did

- Installed VirtualBox
- Downloaded pfSense CE Version 2.9
- Created a new pfSense VM in VirtualBox.
- Allocated 2048 MB of Memory.
- Created 20 GB virtual disk.
- The VM failed to boot intially with VERR_NEM_NOT_AVAILABLE and VERR_SYM_DISABLED. SVM was disabled in BIOS. I enabled SVM Mode in BIOS and the VM booted successfully afterward.  
- nstalled Ubuntu 26.04.1 as a LAN client.
- The VirtualBox GUI only showed Adapter 1 under the Network. I had to use VBoxManage from the command line to add the second adapter directly.
  
```VBoxManage modifyvm "pfSense-Firewall" --nic2 intnet --intnet2 "lan-net"```
  
- I then verified it worked with the following.

```VBoxManage showvminfo "pfSense-Firewall" | findstr "NIC"```

- This confirmed that both NIC 1 and NIC 2 were configured correctly.
- Created three VLANs in pfSense with 802.1Q tagging. The three VLANs are named Trusted (10), IoT (20), and Guest (30). They each with its own subnet and DHCP scope.
- Switched the DHCP backend from legacy ISC to Kea.
- Wrote firewall rules using an RFC1918 alias. Trusted is allow anywhere. IoT and Guest are block to RFC1918 and allow to internet.
- For testing, I added a fifth NIC to pfSense via VBoxManage and bound Trusted, IoT, and Guest to dedicated adapters rather than VLAN-tagged client interfaces.
  
```VBoxManage modifyvm "pfSense-Firewall" --nic5 intnet --intnet5 "lan-guest"```
  
- I then verified it worked with the following.
  
```VBoxManage showvminfo "pfSense-Firewall" | findstr "NIC"```
  
- The reason for this is because VirtualBox doesn't pass VLAN tags by default.
- The clients could ping public IPs, like 8.8.8.8, but couldn't resolved domain names.
- The DHCP scopes for the clients weren't handing out DNS servers.
- I added 1.1.1.1 and 8.8.8.8 to each VLAN's DHCP settings.
- This resolved the issue.
- I then confirmed each client was on the correct subnet using the following.

```ip a```

- I attempted to ping Trusted from Guest, which failed as intended.
- I confirmed internet access was working on each client after the DNS fix.

### Next Steps

- Deploy Snort as an IDS.
- Generate attack traffic with Kali to test detection rules.
