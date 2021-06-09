---
cards-deck: Default
---

**Table of contents**
- [[#Versions|Versions]]
- [[#Architecture|Architecture]]
	- [[#Controllers|Controllers]]
		- [[#Basic rate or Enhanced data rate|Basic rate or Enhanced data rate]]
		- [[#Low Energy (LE)|Low Energy (LE)]]
			- [[#Powe consumption|Powe consumption]]
			- [[#Speed|Speed]]
	- [[#Topology|Topology]]
		- [[#Scatternet|Scatternet]]
		- [[#BR/EDR example|BR/EDR example]]
		- [[#LE example|LE example]]
- [[#Physical|Physical]]
	- [[#Bluetooth radio layer|Bluetooth radio layer]]
		- [[#BR/EDR|BR/EDR]]
		- [[#LE|LE]]
		- [[#Transmisison power|Transmisison power]]
	- [[#Baseband layer|Baseband layer]]
		- [[#Logical transport types|Logical transport types]]
		- [[#Channel access BR/EDR|Channel access BR/EDR]]
		- [[#STATES|STATES]]
- [[#Comparisons|Comparisons]]
	- [[#[[Zigbee]]|[[Zigbee]]]]



# Bluetooth
#card
- Bluetooth is a **collection of protocols** that are a **cable replacement technology** (pc, headphones, printers) that implement a personal area network
	- Lower cost and lower power consumption than [[IEEE 802.11]]
- It was more succesfull with smartphones
- Limited coverage (one room) up to 10 meters
- Slow versions because it's not a replacement of [[Wi-Fi]], nowadays high speed versions uses [[Wi-Fi]] inside
^1623230385481

## Versions
#card
- 3.0, uses a parallel [[IEEE 802.11]] channel to increase up to 24 Mbps, that is negotiated with Bluetooth
- 4.0 in 2010, defines: classic Bluetooth, high speed Bluetooth and low energy Bluetooth (i.e. BLE ~1 Mbps)
- 5.0 in 2016, BLE with burst up to 2Mbps (important for IoT) and increase capacity for connectionless services (location relevant navigation)

Applications
#card #question 
- cable replacement (original idea)
- sync devices (e.g. smartbands)
- access to the internet
- mobile social networking (e.g. Immuni or crowd sensing in general)
^1623230385488

## Architecture
![[bluetooth architecture.png]]
- Host controller interface between hw/sw
- One host (logical layer), while one or more controllers for the physical layer for the different bluetooth versions
	- A bluetooth card contains the controller + L2CAP (logical link control and adaption protocol)
- Protocols (core in **bold**, others are adopted)
	- **Physical layer** defines
		- **Radio**, frequencies
		- **Baseband**, low level procedures
	- **Data Link layer** defines the **setup and controller**
		- Lower part
			- **LMP** (link management protocol for BR/EDR)
			- **LLP** (link layer protocol for BLE)
		- Upper part
			- **L2CAP** (Logical Link Control and Adaptation): higher-level protocol multiplexing (package segmentation)
	- **Application layer**
		-  Management/dipscovery
			- **SDP** for service discovery
			-  SMP, ATT/GATT (kind of a 1small [[ZigBee]] cluster library)
		- RFCOMM emulates serial port for cable replacement
		- Telephony AT, TCS

### Controllers
- Basic rate (headphones for smartphone)
- Low Energy
- Alternate MAC, for high speed transport link [[IEEE 802.11]] (2 Mbps -> 24 Mbps)

#### Basic rate or Enhanced data rate
- Operates at 2.4Ghz like  [[IEEE 802.15.4]] and [[IEEE 802.11]]
- 1Mbps for BR, 3Mbps for EDR
- Forms a **piconet**: group of devices synchronized to a common clock and **frequency hopping pattern** #todo
	- Bluetooth BR/EDR is **connection oriented**, so piconet must be formed before, in order to communicate
- Logical links transport **synchronous** (like [[IEEE 802.15.4]]), **asynchronous** and **isochronous** data (start async and then for a period sync)
	- Synchronization is used to ensure low energy for BLE, because it's required at mac level and using sniff mode
- BR uses at peak 25mA

#### Low Energy (LE)
- Compared to [[#Basic rate Enhanced data rate]]
	- smaller packets to reduce TX/RX power consumption
	- less channels for discovery and connection (searching over a smaller n)
	- simpler state machine
	- used to **expose the state** rather than transferring
- Frequency division multiple access (FDMA) over 40 physical channels + Time
division multiple access (TDMA).
- Physical channel divided with **advertising** and **connection** events, namely frame and its response
- other
	- latency 3ms
	- topology star
	- connections > 2billion
	- range 150m

##### Powe consumption
- less than 20mA at peak with an average of less than 5μA
	- even devices with coin cell battery!

##### Speed
- up to 1Mbps nominal, but in reality for the overhead it's smaller

### Topology
It's the piconet, formed by master and slaves
- **1 Master** than enforces the **synchronization** to enable the communication of the slaves, by **controlling the access** to the channels of the **slaves**
	- roles are not fixed
	- hence slaves can only communicate when the master says so
- Types
	- Point to point
	- Point to multipoint     
- BR/EDR
	-  Up to 7 slaves in a piconet, but other can be in **parked mode** to keep in sync with the master.
- LE   
	- No limit to active slaves in a piconet
	- Just in one piconet

#### Scatternet
![[bluetooth scatternet.png]]
- to extend the range by doing multi-hop communication
- there is no routing protocol defined by the standard
- no usefull applications

#### BR/EDR example
#exam 
![[bluetooth br edr topology.png]]
- Several piconets in an area
- A is the master, the other slaves
	- They can use different low level mechanism
- F is master, the other slaves
	- H is temporarly **deep low power mode** maybe using the sniff state
- E is a slave both in the piconet of A and F, so it need to keep in sync with both in different time
- D is a slave in A but a master in his piconet with J
- K is not connected to any piconet but scanning for it in order to communicate
- Slaves cannot communicate point to point, but need to pass through the master

#### LE example
#exam
![[bluetooth le topology.png]]
- communication
	-  connection-oriented channels (grey one)
		- acts like the master in BR/EDR, so the master A will enforce the sync of the slaves B and C by telling them when to communicate
	- connection-less advertising channels, transparent
		-  D is an **advertiser**, that is not being connected to a piconet but emitting periodically beacons to expose the state to any device in range that is scanning, in this case A is a **scanner**.
		- Be aware that D is not a slave, because there is no connection
		- I, H, J are not in a piconet, but they are advertising/scanners or just one of it.


## Physical

### Bluetooth radio layer

#### BR/EDR
random sequence of 79 channels is generated by the master

How it works in [[ZigBee]]?
bands in [[ZigBee]] are just 16 and it's always the same
- even if there will be overlap with bluetooth, there is the chance of inactive mode of one of the network
- frequencies are bigger
- different encoding

#### LE
- 40 channels
	- connection oriented only the first 37 channels used for **connection oriented** (master/slave)
	- 37, 38, 39 for connection, discovery and broadcast (search O(3) rather than O(40)), used by **advertisers** to emit beacon and by **scanners** for listening

#### Transmisison power
#low-priority
- class 1: Tx power < 20 dBm (100 mW)
	- Long range (100 m);
- class 2: Tx power < 4 dBm (2.5 mW)
	- Normal range (10 m);
- class 3: Tx power < 0 dBm (1 mW)
	- Short range (10 cm).

### Baseband layer
#low-priority
It creates the abstraction of physical channel that is a sequence of 79 frequencies, each piconet with a different sequence
- the slave connects and receive the sequence from the master
- Frequency hopping management (FH).
- management of the channel and of the link (to LMP/LLP)
	- Power control, Channel access, Link control (packets retransmission, sync/async link)
- Error correction

#### Logical transport types
#low-priority 
- broadcast can be used to sync the slaves

#### Channel access BR/EDR
#exam
![[Pasted image 20210609171555.png]]

#### STATES
- standby
- connection
- park
	- only keep in sync
	- extreme low-power mode

## Comparisons 
#card 
![[bluetooth vs other protocols.png]]
^1623230385496

### [[Zigbee]]
#card #question
- Bluetooth is widely available in different kind of devices, [[ZigBee]] is constrained over IoT devices
- Beginning of 2000 there was the need in the E-health sector and [[IEEE 802.15.4]] was not even there
- Only start topology
- Technically multi-hop is doable, but not known use-cases
^1623230585830

c
