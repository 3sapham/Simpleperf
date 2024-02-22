# Portfolio 1
This whole portfolio was created and designed as part of an exam for the DATA2410 course at OsloMet. This README provides information on how to run Simpleperf and conduct the tests. Here, we explain the steps required to set up and run the tool, as well as provide guidance on how to execute the experiments to generate meaningful results.


# Simpleperf: A Python-based Network Throughput Measurement Tool
Simpleperf is a simple network throughput measurement tool based on iperf implemented in Python. It can run in two modes: server and client mode. It uses Python's built-in socket module, threading module, and time module to, e.g. create a TCP network socket and to communicate between the client and server amongst other things. Additionally, argparse, ipaddress, and re modules are used to implement optional arguments the server and client can be invoked with. The sys module is used for exiting the program during exceptions. For the sake of simplcity, we assume 1 KB = 1000 Bytes, and 1 MB = 1000 KB.

## Installation
To use simpleperf, you need to have Python 3 installed in your system. You can download Python from the official website: https://www.python.org/downloads/

## Usage
### Server Mode
Once the server is running, it will wait for incoming client connections. When a client connects, it will create a new thread to handle the transfer. The server will receive data in chunks of 1000 bytes from the client, track how much data was received during the transfer, and send an acknowledgement message back to the client before closing the connection. It will then calculate and print the rate based on how much data was received and how much time elapsed during the connection.

To start simpleperf in server mode with the default options, it should be invoked as: 

    python3 simpleperf.py -s

* `-s, --server`: run simpleperf in server mode; it should receive data and track the total number of bytes

To start simpleperf in server mode with the all the available options, it can be invoked as:

    python3 simpleperf.py -s -b <ip_address> -p <port_number> -f <data_format>

* `-b, --bind`: the IP address to bind the server to (default: `127.0.0.1`)
* `-p, --port`: the port number to listen for incoming connections (default: `8088`)
* `-f, --format`: the format of the output data, the available options are `MB`, `KB` and `B`,  (default: `MB`)

Once the server is running, it should print:

    ---------------------------------------------------------------------------
    A simpleperf server is listening on port XXXX
    ---------------------------------------------------------------------------

When a client connects, the server prints:

    A simpleperf client with <IP address:port> is connected with <server IP:port>

At the end of the transfer the server prints the results in the following format, and gracefully close the connection.

    ID           Interval        Received      Rate

    IP:port      0.0 - 25.0      X MB          Y Mbps

There are four columns here:
1. `ID`: The IP address and port connected
2. `Interval`: The total duration in seconds
3. `Transfer`: `X` is the total number of bytes received
4. `Rate`: `Y` is the rate at which data could be read in Mbps

### Client Mode
Once the client is connected to the server, it will create a new thread to send data in chunks of 1000 bytes to the server and a BYE message to indicate the end of the transfer. It will then close the connection after receiving the acknowledgement message from the server. Lastly, it will calculate and print the bandwidth based on how much data was sent in the elapsed time.

To run simpleperf in client mode, it should be invoked as:

    python3 simpleperf.py -c -I <server_ip> -p <server_port> -t <time> -f <data_format>

* `-c, --client`: run simpleperf in client mode; it should send data and track the total number of bytes
* `-I, --serverip`: the IP address of the server to connect to (default: `127.0.0.1`)
* `-p, --port`: the port number of the server to connect to (default: `8088`)
* `-t, --time`: the time in seconds to transfer data (default: `25`)
* `-f, --format`: the format of the output data, the available options are `MB`, `KB` and `B`,  (default: `MB`)

Once the client is connected to a server, it should print:

    ---------------------------------------------------------------------------
    A simpleperf client connecting to server <IP>, port XXXX
    ---------------------------------------------------------------------------

    Client connected with server_IP port XXXX

Once the transfer is done, the client prints the results in the following format, and exits the program:

    ID           Interval        Transfer      Bandwidth

    IP:port      0.0 - 25.0      X MB          Y Mbps


Here is the other available options that can be used to invoke the client:
* `-n, --num`: the number of bytes to transfer (default: `None`)
* `-i, --interval`: the interval in seconds to print statistics (default: `None`)
* `-p, --parallel`: the number of parallel connections to create (default: `1`)
> NOTE: `–n` and `–t` should not be invoked at the same time



# Performance evaluation
This project aims to measure the bandwidth and latency in a virtual network using `simpleperf`, `ping`, and `iperf` tools. The virtual network is created using Mininet, and the experiments are conducted on this network. The results of these experiments are analyzed and discussed in the report. 

## Experimental setup
To install Mininet on a recent release of Ubuntu or Debian, use the following command if it has not been installed already:

    sudo apt-get install mininet

With problems about openvswitch, make sure it is installed with:

    sudo apt-get install openvswitch-switch
    sudo service openvswitch-switch start

Or try:

    sudo apt-get install openvswitch-testcontroller

To run the various tests, xterm and iperf needs to be installed, run these commands to install if it has not been already: 

    sudo apt-get install xterm
    sudo apt-get install iperf3


## The tests
The tests involve measuring the bandwidth and latency in a virtual network using the tools mentioned above. The tests are conducted on the topology provided in the `portfolio_topology.py` file. To run the script, use the command:

    sudo python3 portfolio_topology.py
 
There are 9 hosts (h1 to h9) and 4 routers (r1 to r4). See the `portfolio_topology.py` file for the IP addresses and details.
> NOTE: When running ping and simpleperf in Mininet, use the IP addresses

### Test case 1: Measuring bandwidth with iperf in UDP mode
In this test case, run three separate iperf tests in UDP mode with -b XM between the client-server pairs h1-h4, h1-h9 and h7-h9 respectively. Where X is the optional rate to measure the bandwidth with for the different pairs. E.g., open terminals for h1 and h4

    xterm h1 h4

Run server on h4 terminal: 

    iperf –s –u

Run client on h1 terminal: 

    iperf -c <server-ip> -u -b 40M

* -u specifies UDP mode

* -b specifies the rate at which you want to send (40 Mbps here for example)

### Test case 2: Link latency and throughput
In this case, measure the RTT (round-trip time) and bandwidth of each of the three individual links between routers R1-R2 (L1), R2-R3 (L2), and R3-R4 (L3), running ping with 25 packets and simpleperf for 25 seconds. 
E.g., ping R2 from R1 terminal:

    ping 10.0.1.2 -c 25

* -c specifies how many packets

simpleperf:

Server: 

    python3 simpleperf.py -s -b 10.0.1.2 -p 8088

Client: 

    python3 simpleperf.py -c -I 10.0.1.2 -p 8088 -t 25

Repeat for the other pairs.

### Test case 3: Path latency and througput
In this case, measure the latency and throughput of each of the three individual paths between the hosts h1-h4, h7-h9 and h1-h9, running ping with 25 packets and simpleperf for 25 seconds.

Same as above, changing the hosts.

### Test case 4: Effects of multiplexing and latency
In this experiment, look at the effects of multiplexing. Different pairs of hosts will simultaneously communicate using simpleperf and ping to measure the latency and throughput with the same parameters as the previous cases (25 packets and 25 seconds). The pairs of hosts that will simultaneously communicate are 
* h1-h4 and h2-h5 
* h1-h4, h2-h5 and h3-h6
* h1-h4 and h7-h9
* h1-h4 and h8-h9

Run ping and simpleperf in the same way as above, but opening two terminals for each of the clients (to run ping and simpleperf simultaneously). E.g., 

Open terminals for h1 (twice), h2 (twice) and h4 and h5

    xterm h1 h1 h2 h2 h4 h5

Run simpleperf server on h4 and h5, then write ping and simpleperf commands in each client terminal, before quickly hitting `ENTER` on all the client hosts so they start roughly at the same time.

### Test case 5: Effects of parallel connections
In this case, measure the throughput when three pairs of hosts are communicating simultaneously using simpleperf. The hosts h1, h2 and h3 are connected to R1 and will simultaneously talk to hosts (h4, h5 and h6) connected to R3. However, h1 will open two parallel connections to communicate with h4, while h2 and h3 will just open one normal connection each. 

Same as the case above, but only running simpleperf. Open terminals for h1 h2 h3 h4 h5 h6, run simpleperf server on h4, h5 and h6, write simpleperf client command on h1, h2 and h3, furthermore `-P 2` on h1, then quickly hit `ENTER` on all the client hosts so that they roughly start at the same time.

## My measurements
The measurement results I got are organized into separate folders inside the `measurements` folder, each corresponding to each test scenario. Within each folder are the files containing the raw measurement data. The results are profoundly analyzed and discussed in the report `teresapham_s345368_portfolio1.pdf`. 

