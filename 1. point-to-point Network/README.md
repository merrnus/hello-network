# Project 1: Point-to-Point Network
<img width="2752" height="1536" alt="Gemini_Generated_Image_f6jaeyf6jaeyf6ja" src="https://github.com/user-attachments/assets/e09317ee-d93e-436c-bd67-63ef8a6e8042" />

## 🌑 The Dark Room

We have a message.

It needs to reach another computer.

No router.
No switch.
No internet.

Just two hosts and a connection between them.

So, how do they communicate?

This project starts with that simple question.

---

# 🧩 The Four Parts of Communication

Before looking at the technical details, let's reduce the problem to four basic parts:

| Part              | Question                           |
| ----------------- | ---------------------------------- |
| **Source**        | Where does the message come from?  |
| **Destination**   | Where does it go?                  |
| **Communication** | How do the two hosts connect?      |
| **Protocols**     | How do they understand each other? |

For this project:

* **Source:** macOS Host
* **Destination:** Kali Linux
* **Communication:** VMware Host-Only Network
* **Protocols:** ARP and ICMP

Now let's look at each part.

---

# 1. 📤 Source

The source is the device that sends the message.

In this project, the source is the **macOS Host**.

The destination is a Kali Linux virtual machine running inside VMware Fusion.

```text
[ macOS ]
  Source
     │
     │
     ▼
[ Kali Linux ]
 Destination
```

But having two devices is not enough.

They need a way to reach each other.

---

# 2. 📥 Destination

Our destination is the **Kali Linux virtual machine**.

We can inspect its network interfaces with:

```bash
ip a
```

The output contains several useful pieces of information.

### `lo`

```text
127.0.0.1
```

This is the loopback interface.

It allows the system to communicate with itself and is useful for testing the local TCP/IP stack.

### `eth0`

This is the network interface used by Kali for network communication.

The output shows its IP address and, next to:

```text
link/ether
```

its MAC address.

This gives us two different types of addresses:

* **IP address** — logical addressing
* **MAC address** — link-layer addressing

---

# 3. 🔌 Communication

Now we know the source and the destination.

But they still need a path between them.

For this project, we use VMware Fusion's:

> **Host-Only Network**

This creates a private virtual network between the macOS Host and the Kali Linux VM.

```text
┌──────────────┐
│    macOS     │
│    Source    │
└──────┬───────┘
       │
       │ Host-Only
       │ Network
       │
┌──────▼───────┐
│ Kali Linux   │
│ Destination  │
└──────────────┘
```

The important part is that this network does not need internet access.

The goal is simple:

> **Get two hosts to communicate directly.**

---

# 4. 🤝 Protocols

The connection exists.

The source can reach the destination.

But there is another problem:

> **How do they understand each other?**

This is where protocols come in.

In this project, we focus on two of them:

* **ARP**
* **ICMP**

---

## ARP — "Who has this IP?"

Suppose the source knows the destination's IP address.

That is not enough for local network communication.

The source also needs the destination's MAC address.

ARP helps us find it.

The source can ask the local network:

> **"Who has this IP address? Tell me your MAC address."**

This request is sent as a broadcast.

The device that owns the IP address replies with its MAC address.

The result is then stored in the ARP table.

In simple terms:

```text
IP address
    ↓
   ARP
    ↓
MAC address
```

So ARP answers a simple question:

> **"I know where the device is logically. Which device is it on this local network?"**

---

## ICMP — "Are you there?"

Now we have the destination's addressing information.

Next, we want to know if the destination is reachable.

This is where ICMP comes in.

We can trigger an ICMP Echo Request using:

```bash
ping 172.16.110.128
```

If the destination is reachable, it sends an ICMP Echo Reply back.

```text
macOS
  │
  │ ICMP Echo Request
  ▼
Kali Linux
  │
  │ ICMP Echo Reply
  ▼
macOS
```

And just like that, our first communication works.

---

# 🧱 Where does this fit in the OSI Model?

So far, we looked at the problem using our four-part model:

```text
Source
   ↓
Destination
   ↓
Communication
   ↓
Protocols
```

Now we can look at the same system from a more technical perspective.

This project mainly uses the first three layers of the OSI model.

### Layer 3 — Network

This layer handles logical addressing.

**IP addresses** work here.

```text
"Where should this data go?"
```

### Layer 2 — Data Link

This layer handles communication on the local network.

**MAC addresses** are used here.

```text
"Which device should receive this data on this network?"
```

### Layer 1 — Physical

This is where data is carried as physical signals.

Depending on the technology, these can be electrical signals, light, or radio waves.

In a real Ethernet network, cables and network interfaces are part of this layer.

In our project, VMware provides the virtual network connection.

---

# 🏠 IP vs. MAC

A simple way to think about the difference:

### IP = Logical Address

It tells us which network or logical destination a device belongs to.

It can change.

### MAC = Link-Layer Address

It is associated with a network interface and is used for communication on the local network.

It can also be changed in software, so it is better not to think of it as a permanent identity.

In short:

```text
IP  → "Where?"
MAC → "Which interface on this local network?"
```

---

# 🧪 Lab

### Environment

* **Host:** macOS
* **Virtual Machine:** Kali Linux
* **Virtualization:** VMware Fusion
* **Network:** Host-Only
* **Goal:** Communicate without using the internet

### Step 1 — Inspect the network interfaces

On Kali Linux:

```bash
ip a
```

Look for:

* `lo` → Loopback
* `eth0` → Network interface
* `inet` → IP address
* `link/ether` → MAC address

### Step 2 — Test the connection

From the macOS terminal:

```bash
ping 172.16.110.128
```

If the connection is working, the communication looks roughly like this:

```text
Source
   ↓
Find destination MAC using ARP
   ↓
Communicate over the local network
   ↓
Send ICMP Echo Request
   ↓
Destination
   ↓
Send ICMP Echo Reply
   ↓
Source
```

---

# 🎯 Result

We started with two hosts.

No router.
No switch.
No internet.

We then:

1. Identified the **source**.
2. Identified the **destination**.
3. Created a **communication path**.
4. Used **ARP** to resolve the destination IP to a MAC address.
5. Used **ICMP** to test the connection.
6. Successfully exchanged packets between the two hosts.

This gave us a small but complete environment for understanding the basic building blocks of network communication.

---

# 📚 What I Learned

* How a Host-Only network works
* The difference between IP and MAC addressing
* How ARP resolves local IP addresses to MAC addresses
* How ICMP and `ping` work
* How the first three OSI layers relate to network communication
* How to break a network problem into four basic parts: **Source, Destination, Communication, and Protocols**

---

# 🚀 Next

We have two hosts communicating directly.

But what happens when we put another device between them?

```text
Source → ? → Destination
```

That's where the next project begins.
