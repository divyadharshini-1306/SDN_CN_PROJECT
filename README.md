# 🔗 SDN Project: Link Failure Detection and Recovery using Ryu Controller

---

#  Project Overview

This project demonstrates how a Software Defined Network (SDN) behaves under:

* Normal conditions
* Link failure
* Link recovery

The controller dynamically installs flow rules using OpenFlow (Ryu), and behavior is verified using:

* Mininet (connectivity)
* Flow tables (`ovs-ofctl`)
* Wireshark (packet-level analysis)
* iperf (throughput analysis)

---

#  Network Topology

```
        h1s1        h1s2
          |            |
          s1 -------- s2
          |            |
        h2s1        h2s2
```

* 2 switches: s1, s2
* 4 hosts: h1s1, h2s1 (left), h1s2, h2s2 (right)
* Link between s1 and s2 is the **failure point**

---

# ⚙️ REQUIREMENTS

* Ubuntu (20.04 / 22.04)
* Mininet
* Docker
* Ryu Controller
* Wireshark

---

#  COMPLETE EXECUTION GUIDE (STRICT ORDER)

---

##  STEP 1 — OPEN TERMINAL 1 (Controller)

```bash
sudo mn -c
sudo docker rm -f $(docker ps -aq) 2>/dev/null
```

### Start Controller:

```bash
cd ~/SDN_CN_PROJECT/controller

sudo docker run -it --rm --network host \
-v $(pwd):/app \
-w /app \
-e PYTHONPATH=/app \
osrg/ryu \
ryu-manager my_controller.py
```

###  Expected:

```
 CONTROLLER READY
```

Keep this terminal running

---

##  STEP 2 — OPEN TERMINAL 2 (Mininet)

```bash
sudo mn --topo linear,2,2 --controller=remote,ip=127.0.0.1,port=6653
```

###  Expected:

```
mininet>
```

---

#  TESTING PHASE

---

##  TEST 1 — NORMAL CONNECTIVITY

```bash
pingall
```

### Expected:

```
*** Results: 0% dropped
```

---

##  TEST 2 — LINK FAILURE

```bash
py net.configLinkStatus('s1', 's2', 'down')
pingall
```

### Expected:

```
*** Results: ~66% dropped
```

---

##  TEST 3 — LINK RECOVERY

```bash
py net.configLinkStatus('s1', 's2', 'up')
h1s1 ping -c 5 h1s2
pingall
```

### Expected:

```
*** Results: 0% dropped
```

---

#  FLOW TABLE VERIFICATION

```bash
sh ovs-ofctl dump-flows s1
sh ovs-ofctl dump-flows s2
```

### Expected:

```
priority=1,in_port=...,dl_src=...,dl_dst=... actions=output:...
priority=0 actions=CONTROLLER:65535
```

✔ Confirms dynamic flow installation

---

#  WIRESHARK ANALYSIS

## Open Wireshark:

```bash
sudo wireshark
```

### Select:

```
Loopback: lo
```

### Start Capture → Generate traffic:

```bash
h1s1 ping -c 5 h1s2
```

### Apply filter:

```
openflow
```

### Expected:

* OFPT_PACKET_IN
* OFPT_PACKET_OUT
* OFPT_ECHO_REQUEST / REPLY

---

## Note:

FLOW_MOD may not appear due to fast execution, but is verified using flow tables.

---

# 📈 THROUGHPUT ANALYSIS (IPERF)

##  NORMAL CONDITION

```bash
h1s1 iperf -s &
h1s2 iperf -c h1s1
```

### Expected:

```
~13 Gbits/sec bandwidth
```

✔ High throughput → network working efficiently

---

##  LINK FAILURE

```bash
py net.configLinkStatus('s1', 's2', 'down')
h1s2 iperf -c h1s1
```

### Expected:

```
tcp connect failed (No route to host)
```

✔ Confirms network is broken

---

##  RECOVERY

```bash
py net.configLinkStatus('s1', 's2', 'up')
h1s2 iperf -c h1s1
```

### Expected:

```
~13 Gbits/sec bandwidth restored
```

✔ Confirms recovery

---

#  PERFORMANCE SUMMARY

| Scenario     | Packet Loss | Throughput       |
| ------------ | ----------- | ---------------- |
| Normal       | 0%          | ~13 Gbps         |
| Link Failure | High        | Connection fails |
| Recovery     | 0%          | ~13 Gbps         |

---

#  KEY CONCEPT

* Controller does NOT directly detect failure
* Failure → packet drops
* New packets → trigger `packet_in`
* Controller → installs new flows

 This is **Reactive SDN**

---

#  VALIDATION

The system was validated using:

* `ping` → connectivity
* `ovs-ofctl` → flow rules
* `Wireshark` → OpenFlow packets
* `iperf` → throughput

---

#  FINAL PROOF (SUBMISSION)

Include screenshots of:

1. pingall (normal)
2. pingall (failure)
3. pingall (recovery)
4. flow tables
5. Wireshark packets
6. iperf (normal, failure, recovery)

---

#  CONCLUSION

This project demonstrates:

* Dynamic SDN flow rule installation
* 
* <img width="937" height="395" alt="image" src="https://github.com/user-attachments/assets/1788eda5-b23f-43a6-9854-67de455a9f40" />

<img width="1051" height="575" alt="image" src="https://github.com/user-attachments/assets/d35c7578-c00c-4bf8-bbaf-b2759748d9ee" />


* pingall (normal)

<img width="687" height="261" alt="image" src="https://github.com/user-attachments/assets/d09fefc3-4b3f-4005-898f-4edbf6d36dba" />

* pingall (failure)

  <img width="941" height="271" alt="image" src="https://github.com/user-attachments/assets/60a90472-57b7-43e3-9f0d-eabb62f69f4e" />


  
* pingall (recovery)

  <img width="1033" height="400" alt="image" src="https://github.com/user-attachments/assets/149798eb-c024-4e11-b368-8670bb5637ba" />



  
* flow tables

  <img width="1530" height="240" alt="image" src="https://github.com/user-attachments/assets/8932d60f-3225-44e6-bd8c-96ccc404eae4" />

  <img width="1578" height="210" alt="image" src="https://github.com/user-attachments/assets/52cf75e2-ba38-4ba5-8e8f-355badb097cd" />


  


  
* Performance validation using throughput
* (normal):

  <img width="1186" height="356" alt="image" src="https://github.com/user-attachments/assets/cadda428-6834-4b38-b7cc-ece70ada10ff" />


  (failure)

  <img width="1625" height="302" alt="image" src="https://github.com/user-attachments/assets/eec49cff-8bbd-48fa-a6ed-e4508c1b6cfb" />

  (recovery)

  <img width="1203" height="318" alt="image" src="https://github.com/user-attachments/assets/01828b76-4ac4-4d17-9c38-2fc5492ef6c7" />


  
* wireshark output:

  <img width="1315" height="628" alt="image" src="https://github.com/user-attachments/assets/f510317b-5a29-4731-948c-bcfe1f563b8f" />

  

---



---

# 👤 AUTHOR

M.B.DIVYADHARSHINI
