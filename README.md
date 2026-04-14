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

 Open a terminal and run:

```bash
sudo mn -c
sudo docker rm -f $(docker ps -aq) 2>/dev/null
```

---

### Start Ryu Controller:

```bash
cd ~/SDN_CN_PROJECT/controller

sudo docker run -it --rm --network host \
-v $(pwd):/app \
-w /app \
-e PYTHONPATH=/app \
osrg/ryu \
ryu-manager my_controller.py
```

---

###  Expected Output:

```
 CONTROLLER READY
```

( Keep this terminal running (DO NOT CLOSE))

---

##  STEP 2 — OPEN TERMINAL 2 (Mininet)
 Open a **new terminal** (don’t reuse Terminal 1)

```bash
sudo mn --topo linear,2,2 --controller=remote,ip=127.0.0.1,port=6653
```

---

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

###  Expected Output:

```
*** Results: 0% dropped (all hosts reachable)
```

---

##  TEST 2 — LINK FAILURE

```bash
py net.configLinkStatus('s1', 's2', 'down')
pingall
```

---

###  Expected Behavior:

* Hosts on SAME switch → communicate
* Hosts across switches → FAIL

### Example:

* h1s1 → h2s1 ✔
* h1s1 → h1s2 ❌

---

###  Expected Output:

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

---

###  Expected Output:

```
*** Results: 0% dropped
```

👉 Network fully restored

---

#  FLOW TABLE VERIFICATION

## Run inside Mininet:

```bash
sh ovs-ofctl dump-flows s1
sh ovs-ofctl dump-flows s2
```

---

##  Expected Output Pattern:

```
priority=1,in_port="s1-ethX",dl_src=...,dl_dst=... actions=output:"s1-ethY"
priority=0 actions=CONTROLLER:65535
```

---

##  Interpretation:

* `priority=1` → dynamically installed rules
* `dl_src/dl_dst` → MAC-based forwarding
* `actions=output` → forwarding decision
* `priority=0` → default rule (table-miss)

 Confirms controller-installed flows

---

#  WIRESHARK ANALYSIS

---

##  STEP 3 — OPEN WIRESHARK (NEW WINDOW)

 Open another window:

```bash
sudo wireshark
```

---

##  Select Interface:

```
Loopback: lo
```

---

##  Start Capture:

Click:
▶️ Blue shark fin icon

---

##  STEP 4 — GENERATE TRAFFIC

Go back to Mininet terminal:

```bash
h1s1 ping -c 5 h1s2
```

---

##  STEP 5 — APPLY FILTER

After packets appear, apply:

```
openflow
```

---

##  Expected Packets:

* `OFPT_PACKET_IN`
* `OFPT_PACKET_OUT`
* `OFPT_ECHO_REQUEST / REPLY`

---

##  Note on FLOW_MOD

You may NOT see:

```
OFPT_FLOW_MOD
```

### Reason:

* Flow rules are installed very quickly
* Happens only once during first packet
* May not be captured

---

##  Correct Explanation:

> Flow installation is verified using ovs-ofctl dump-flows even if FLOW_MOD is not visible in Wireshark.

---

#  PERFORMANCE ANALYSIS

---

## 🔹 Metrics

| Scenario     | Packet Loss | Behavior          |
| ------------ | ----------- | ----------------- |
| Normal       | 0%          | Stable network    |
| Link Failure | ~66%        | Network partition |
| Recovery     | 0%          | Restored network  |

---

##  RTT Example:

```
rtt min/avg/max/mdev = 0.100/1.202/5.478/2.137 ms
```

---

##  Interpretation:

* Low RTT → efficient forwarding
* Recovery requires new packets
* Controller reacts dynamically

---

#  KEY CONCEPT

Controller does NOT directly detect failure.

Instead:

* Failure → packet drops
* New packets → packet_in
* Controller → installs new flows

 This is **Reactive SDN**

---

#  FINAL PROOF (SUBMISSION)

Include screenshots of:

1. pingall (normal)
2. pingall (link down)
3. pingall (recovery)
4. flow table output
5. Wireshark OpenFlow packets

---

#  CONCLUSION

This project demonstrates:

* SDN-based dynamic forwarding
* Link failure impact
* Automatic recovery using controller logic
* Verification at:

  * Network level
  * Flow level
  * Packet level

---

# 
---

# 👤 AUTHOR

M.B.DIVYADHARSHINI
