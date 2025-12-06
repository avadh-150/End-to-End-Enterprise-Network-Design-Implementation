Perfect — now that you’ve clarified **EXACTLY what features your project contains**, I will create a **clean, complete, professional, final documentation ONLY for:**

### ✅ VLAN

### ✅ HSRP

### ✅ SVI (Inter-VLAN Routing)

### ✅ DTP

### ✅ NAT (PAT)

### ✅ Site-to-Site IPsec VPN

### ✅ Full verification + troubleshooting

No extra junk.  
No unnecessary explanations.  
Just the features you actually configured.

This is the **final documentation you can submit** for your BCA project or keep for long-term reference.

---

# 📘 **ENTERPRISE NETWORK PROJECT DOCUMENTATION**

## **VLAN • HSRP • SVI • DTP • NAT • SITE-TO-SITE IPSEC VPN**

---

# **1. PROJECT OVERVIEW**

This project implements a multi-layer enterprise network with redundancy, segmentation, secure WAN connectivity, and dynamic Layer-3 routing. The design includes:

- Segregated VLANs for user networks
    
- SVI on multilayer switches for inter-VLAN routing
    
- HSRP for default gateway redundancy
    
- DTP for dynamic trunk formation
    
- NAT overload for internet access
    
- Secure Site-to-Site IPsec VPN between Main Site and Server Branch
    

---

# **2. VLAN DESIGN**

|VLAN|Purpose|Network|Gateway (VIP)|
|---|---|---|---|
|VLAN 10|Users|192.168.10.0/24|192.168.10.100|
|VLAN 20|Users|192.168.20.0/24|192.168.20.100|
|VLAN 30|Users|192.168.30.0/24|192.168.30.100|

SWITCH CONFIG:

```
vlan 10
vlan 20
vlan 30
```

ACCESS PORTS:

```
interface fa0/1
 switchport mode access
 switchport access vlan 10
```

---

# **3. DTP – Dynamic Trunking Protocol**

Used between multilayer switches and edge switches.

```
interface g0/1
 switchport mode dynamic desirable
```

OR forced trunking:

```
switchport mode trunk
switchport trunk encapsulation dot1q
```

---

# **4. SVI – INTER-VLAN ROUTING**

Configured on both HSRP routers (Master/Backup).

```
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
interface vlan 20
 ip address 192.168.20.1 255.255.255.0
interface vlan 30
 ip address 192.168.30.1 255.255.255.0
ip routing
```

Each router participates in HSRP and provides redundancy.

---

# **5. HSRP – HOT STANDBY ROUTER PROTOCOL**

### **Master Router:**

```
interface vlan 10
 standby 10 ip 192.168.10.100
 standby 10 priority 110
 standby 10 preempt

interface vlan 20
 standby 20 ip 192.168.20.100
 standby 20 priority 110
 standby 20 preempt

interface vlan 30
 standby 30 ip 192.168.30.100
 standby 30 priority 110
 standby 30 preempt
```

### **Backup Router:**

```
interface vlan 10
 standby 10 ip 192.168.10.100

interface vlan 20
 standby 20 ip 192.168.20.100

interface vlan 30
 standby 30 ip 192.168.30.100
```

HSRP Virtual IPs:

- 192.168.10.100
    
- 192.168.20.100
    
- 192.168.30.100
    

---

# **6. NAT – PAT FOR INTERNET ACCESS (SITE A)**

### **NAT Exemption ACL for VPN:**

```
ip access-list extended NAT-OUT
 deny ip 192.168.0.0 0.0.255.255 10.1.1.0 0.0.0.255
 permit ip 192.168.0.0 0.0.255.255 any
```

### **NAT Overload:**

```
ip nat inside source list NAT-OUT interface Serial0/0/0 overload
```

### **Interface Roles:**

```
g0/0 → inside
g0/1 → inside
s0/0/0 → outside
```

---

# **7. SITE-TO-SITE IPSEC VPN CONFIGURATION**

---

## **7.1 Phase-1 (ISAKMP Policy)**

```
crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key detect45 address 100.1.1.6
```

---

## **7.2 Phase-2 (IPsec Policy)**

```
crypto ipsec transform-set VPNset esp-aes esp-sha-hmac
```

---

## **7.3 Crypto ACL – “Interesting Traffic”**

```
access-list 130 permit ip 192.168.0.0 0.0.255.255 10.1.1.0 0.0.0.255
```

Site B must mirror this:

```
permit ip 10.1.1.0 0.0.0.255 192.168.0.0 0.0.255.255
```

---

## **7.4 Crypto Map**

```
crypto map VPNmap 10 ipsec-isakmp
 set peer 100.1.1.6
 set transform-set VPNset
 match address 130

interface Serial0/0/0
 crypto map VPNmap
```

---

# **8. VERIFICATION COMMANDS**

---

### **8.1 VLAN/SVI**

```
show vlan brief
show ip interface brief
ping gateway
```

---

### **8.2 HSRP**

```
show standby brief
```

Expected:

- Master has **Active**
    
- Backup has **Standby**
    

---

### **8.3 DTP**

```
show dtp interface g0/1
```

Should show _Desirable_ or _Trunking_.

---

### **8.4 NAT**

```
show ip nat translations
show ip nat statistics
```

No NAT for VPN subnet:

```
192.168.x.x → 10.1.1.x   (must NOT appear)
```

---

### **8.5 IPsec VPN**

#### Phase-1:

```
show crypto isakmp sa
```

Look for:

```
QM_IDLE / ACTIVE
```

#### Phase-2:

```
show crypto ipsec sa
```

You must see:

- ACTIVE SAs
    
- Packet counters increasing
    

Your output:

```
#pkts encaps: 17
#pkts decaps: 18
Status: ACTIVE
```

This confirms **VPN is UP**.

---

# **9. TROUBLESHOOTING TABLE**

|Issue|Cause|Fix|
|---|---|---|
|No HSRP failover|Priority or preempt missing|Add priority + preempt|
|VLAN host cannot reach gateway|Wrong VLAN access/trunk config|Fix switchports|
|Inter-VLAN routing fails|SVI shut / IP missing|Enable SVI|
|VPN not forming|Wrong key / ACL mismatch|Compare both sites|
|IPsec counters 0|NAT still applied|Fix NAT exemption|
|DTP not forming trunk|One side static, other dynamic|Set both to trunk|

---

# **10. FINAL SUMMARY**

Your project includes:

### ✔ VLAN segmentation

### ✔ Trunking with DTP

### ✔ SVI-based inter-VLAN routing

### ✔ Redundant gateways using HSRP

### ✔ NAT for internet access

### ✔ Secure Site-to-Site IPsec VPN

### ✔ Full verification + troubleshooting

All core enterprise features are implemented correctly and working.

---

# 👉 I have Face issues like 
this project i have face issue with this i have configure VLANs, DTP, HSRP, OSPF, NAT after this i ping from PC from Vlan 10 to main server branch but i face "request time out" but when i do this on L3 master switch ping to HTTP or any server it successfully reply and also i do this in main server branch site router i ping to any vlan line 10,20,30 successfully reply

**I FOUND THE SOLUTION**
in Configuration in NAT i Forgot to add "ip nat inside" in "gig0/1" interface i only add in "gig0/0" interface 
"IP NAT INSITE" configure in Both interface "gig0/1 and gig0/0"
that's why i face this problem.....