# CSC458 A2 — Programming Assignment 2: Bufferbloat 

# Overview
This assignment aims to study the dynamics of TCP in home networks. The figure below shows a ”typical” home network with a home router connected to an end-host. The Home Router is connected via cable or DSL to a headend router at the Internet access provider’s office. We are going to study what happens when we download data from a remote server to the end-host in this home network by doing the following:
1. Start a long-lived TCP flow, sending data from h1 to h2, using iperf.
2. Send pings from h1 to h2 10 times a second and record the RTTs.
3. Plot the following time series:
a. CWND for The long-lived TCP flow
b. RTT reported by ping
c. Queue size at the bottleneck
4. Spawn a webserver on h1. Periodically download the index.html webpage (three times every five seconds) from h1 and measure how long it takes to be fetched (on average).
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
For answers to the Questions and Theoretical Analysis from the handout, please refer to ```CSC458-A2-report.pdf``` included in this submission. As per the handout, the answers




