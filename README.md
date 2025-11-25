# CSC458 A2 — Programming Assignment 2: Bufferbloat 

# Overview
This assignment aims to study the dynamics of TCP in home networks. The figure below shows a ”typical” home network with a home router connected to an end-host. The Home Router is connected via cable or DSL to a headend router at the Internet access provider’s office. We are going to study what happens when we download data from a remote server to the end-host in this home network by doing the following:
1. Start a long-lived TCP flow, sending data from ```h1``` to ```h2```, using ```iperf```.
2. Send pings from ```h1``` to ```h2``` 10 times a second and record the RTTs.
3. Plot the following time series:

    - CWND for The long-lived TCP flow
    - RTT reported by ping
    - Queue size at the bottleneck

4. Spawn a webserver on ```h1```. Periodically download the ```index.html``` webpage (three times every five seconds) from ```h1``` and measure how long it takes to be fetched (on average).
5. Do this experiment twice for a router buffer size of 100 packets and 20 packets, respectively. 


# Installation
You may run `sudo ./scripts/setup.sh` to install the existing required dependencies for this assignment.
This file was modified to include the `statistics` package required in `bufferbloat.py`.


# Reproducing Results
Run the shell command `sudo ./scripts/run.sh`. This will produce 6 plots for router buffer sizes of 20 and 100:
- CWND for The long-lived TCP flow: ```bb-q20/cwnd-iperf.png``` and ```bb-q100/cwnd-iperf.png```
- Queue size at the bottleneck: ```bb-q20/q.png``` and ```bb-q100/q.png```
- RTT reported by ping: ```bb-q20/rtt.png``` and ```bb-q100/rtt.png```

Additionally, the output for various functions in ```bufferbloat.py``` will be written to text files under the ```bb-q20``` and ```bb-q100``` directories along with the plots. 
Please take a look at ```results.txt``` containing the mean and stddev for webpage fetch times, with the assistance of ```measure_time(net, h1, h2)``` helper function in ```bufferbloat.py```. Comments for each modification are included. 


# Assignment Report
For answers to the Questions and Theoretical Analysis from the handout, please refer to ```CSC458-A2-report.pdf``` included in this submission. As per the handout, the ***Questions*** section of the report will also be included in on this page below: 

**QUESTIONS**

**1. Why do you see a difference in webpage fetch times with small and large router buffers?**

With smaller buffers, the queue is shorter so the latency is also lower, and so there are faster webpage fetches since the RTT drops. With larger buffers, many packets can occupy the queue, which keeps throughput high but also makes packets wait longer in the queue, increasing delay. As a result, webpage fetch times go up. This can be seen from one of the results obtained:

    bb-q100: average=1.815298, stddev=0.242084
    bb-q20: average=0.543328, stddev=0.581217

<br> 

**2. Bufferbloat can occur in other places such as your NIC. Use ```ifconfig``` or ```ip link show``` on your Mininet VM to identify the transmit queue length ```txqueuelen``` of an interface such as ```enp0s1``` and report the output. For this queue size and a draining rate of 100 Mbps, what is the maximum time a packet might wait before leaving the NIC?**

The output of ```ifconfig enp0s1``` shows a ```txqueuelen``` of 1000.  
Assuming a packet size of 1500 bytes, the maximum time a packet might wait can be calculated as follows:

    Bits per packet = 1500 bytes × 8 = 12,000 bits
    Total bits in queue = 12,000 bits/packet × 1000 packets = 12,000,000 bits
    Draining rate = 100 Mbps = 100,000,000 bits/s
    Maximum queueing time = 12,000,000 / 100,000,000 = 0.12s = 120ms

Therefore, a packet could wait up to 120 ms before leaving the NIC.

<br>

**3. Analyze your plots of CWND, RTT, and queue size:**
<br>
**3a. Derive or express a symbolic equation showing how RTT varies with queue size.**

Based on the plots, one can see that the RTT increases linearly with the queue size. This can be expressed as the sum of base propagation delay and queuing delay as follows:

    RTT(Q) = RTT_0 + Q/C

Where Q represents the number of packets in the queue, and C represents the bandwidth of the bottleneck link capacity in packets/second. 

<br> 

**3b. Explain how CWND oscillations correspond to RTT spikes.**
  
Each time TCP additively increases the CWND, more data is injected into the network, causing the queue to fill. As the CWND grows in upward oscillations in the plot, the number of packets in the queue increases, which increases the RTT and produces corresponding spikes. When the queue reaches capacity and packet loss occurs, TCP reduces the CWND multiplicatively, and the RTT spikes fall as the CWND plot oscillates downward.

<br> 

**3c. Discuss how buffer size influences TCP performance and webpage fetch times.**
  
Based on the plots for ```bb-q100```, larger buffers allow TCP to achieve higher CWND sizes and higher throughput, but they also increase RTTs. This means more data can be sent, but webpage fetch times can be longer when the queue approaches full capacity. As seen in the plots for ```bb-q20```, shorter buffers cause packets to be dropped earlier, leading to more frequent reductions in CWND and smaller RTTs. As a result, webpages experience lower queuing delays and faster responses, but overall throughput may be slightly reduced. This behavior is also reflected in the results mentioned in my answer for question 1. 

<br>

**4. Identify and describe two ways to mitigate bufferbloat. For each, explain trade-offs and propose a Mininet experiment to evaluate quantitative effectiveness. Clearly document your experimental design, including parameter choices, measurement methodology, and justification for your conclusions, so that the TA can reproduce your results.**

i. Method 1: Smaller buffer with Explicit Congestion Notification (ECN)
- Trade-offs: Smaller buffers reduce queuing delay and lower RTT, but increase the chance of packet loss under congestion. ECN provides early feedback for congestion, but requires marking or dropping packets to signal the sender.
- Parameter choices: Enable ECN on the bottleneck link ```s0-eth2```. Compute metrics such as ```avg_throughput``` using the results from ```iperf```.
- Measurement methodology: Compare mean web fetch time, standard deviation, ```avg_throughput```, and RTT for ```maxq=20``` with ECN enabled versus ```maxq=20``` and ```maxq=100``` without ECN.
- Justification: Smaller buffers reduce RTT but may increase packet loss. Enabling ECN allows TCP to react to congestion earlier, marking packets instead of dropping them, which mitigates bufferbloat.

<br>

ii. Method 2: Active Queue Management (Random Early Detection, RED)
- Trade-offs: RED monitors the average queue length and drops packets probabilistically before the queue is full. This prevents long queues and reduces bufferbloat, keeping RTT low. Parameter tuning is difficult and may reduce throughput.
- Parameter choices: Enable RED on the bottleneck link ```s0-eth2```. Configure RED with ```min_threshold```, ```max_threshold```, ```drop_probability```, and ```avg_packet_size```. Compute ```avg_queue_length``` using ```count```.
- Measurement methodology: Compare mean web fetch time, standard deviation, RTT, queue occupancy, and CWND sizes for RED versus non-RED.
- Justification: By actively monitoring the average queue length, RED allows TCP to react to congestion earlier, reducing RTT and mitigating bufferbloat.
