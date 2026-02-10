# Chapter 1.2: Types of Computer Networks

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

Computer networks come in various sizes and cover different geographical areas. This chapter explores the different types of networks classified by their size, coverage area, and purpose. Understanding these classifications helps in designing and implementing appropriate network solutions.

---

## 🎯 Learning Objectives

- Classify networks based on geographical coverage
- Understand characteristics of each network type
- Compare and contrast different network types
- Identify real-world applications of each type

---

## 1. Network Classification by Size

### Overview Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    NETWORK SIZE COMPARISON                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────┐                                                                │
│  │ PAN │ ← Personal Area Network (1-10 meters)                         │
│  └──┬──┘                                                                │
│     │                                                                   │
│  ┌──▼──────────┐                                                        │
│  │    LAN      │ ← Local Area Network (10m - 1km)                      │
│  └──────┬──────┘                                                        │
│         │                                                               │
│  ┌──────▼─────────────────┐                                             │
│  │         MAN            │ ← Metropolitan Area Network (1-100km)      │
│  └──────────┬─────────────┘                                             │
│             │                                                           │
│  ┌──────────▼───────────────────────────────────────────┐               │
│  │                       WAN                            │ ← Wide Area   │
│  └──────────────────────────────────────────────────────┘   (Worldwide) │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Personal Area Network (PAN)

### Definition

A **Personal Area Network (PAN)** is the smallest network type, designed for interconnecting devices around an individual person within a range of about 10 meters.

### Characteristics

```
                    ┌─────────────┐
                    │ SMARTPHONE  │
                    └──────┬──────┘
                           │ Bluetooth
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼────┐      ┌─────▼─────┐     ┌─────▼─────┐
    │Smartwatch│      │ Wireless  │     │ Bluetooth │
    │         │      │ Earbuds   │     │ Speaker   │
    └─────────┘      └───────────┘     └───────────┘
    
         PERSONAL AREA NETWORK (PAN)
              Range: ~10 meters
```

### Types of PAN

| Type | Technology | Range | Data Rate |
|------|------------|-------|-----------|
| **Wired PAN** | USB, FireWire | ~5m | Up to 480 Mbps |
| **Wireless PAN (WPAN)** | Bluetooth | ~10m | Up to 3 Mbps |
| **Infrared PAN** | IrDA | ~1m (line of sight) | Up to 4 Mbps |
| **NFC** | Near Field Communication | ~10cm | 424 Kbps |

### Applications

- Wireless headphones and earbuds
- Fitness trackers and smartwatches
- Wireless keyboards and mice
- File transfer between phone and laptop
- Smart home personal devices

---

## 3. Local Area Network (LAN)

### Definition

A **Local Area Network (LAN)** connects devices within a limited geographical area such as a home, office, school, or building.

### Characteristics

```
                         ┌─────────────────────┐
                         │   COMPANY OFFICE    │
┌────────────────────────┴─────────────────────┴────────────────────────┐
│                                                                        │
│    ┌──────────┐                              ┌──────────┐             │
│    │   PC 1   │                              │   PC 2   │             │
│    └────┬─────┘                              └────┬─────┘             │
│         │                                        │                    │
│         │         ┌──────────────────┐          │                    │
│         └─────────┤     SWITCH       ├──────────┘                    │
│                   └────────┬─────────┘                               │
│                            │                                          │
│         ┌──────────────────┼──────────────────┐                       │
│         │                  │                  │                       │
│    ┌────▼────┐       ┌─────▼─────┐      ┌─────▼─────┐                 │
│    │ Printer │       │  Server   │      │  Router   │ ──► Internet   │
│    └─────────┘       └───────────┘      └───────────┘                 │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

### Key Features

| Feature | Description |
|---------|-------------|
| **Ownership** | Privately owned and managed |
| **Coverage** | Single building or campus |
| **Speed** | High (100 Mbps to 10 Gbps) |
| **Error Rate** | Very low |
| **Cost** | Low initial and maintenance cost |
| **Technology** | Ethernet, Wi-Fi |

### LAN Configurations

**Ethernet LAN (Wired):**
```
    PC ─────┬───── Switch ─────┬───── PC
            │                  │
            PC                 Server
```

**Wireless LAN (WLAN):**
```
    Laptop )))                   ((( Tablet
              )))             (((
                  ))) ┌──┐ (((
                      │AP│        ← Access Point
                      └──┘
              )))             (((
    Phone  )))                   ((( Smart TV
```

### Advantages of LAN

1. **Fast data transfer** - High bandwidth within local network
2. **Resource sharing** - Printers, files, internet connection
3. **Centralized data** - Easy backup and management
4. **Cost effective** - Shared resources reduce costs
5. **Security** - Easier to implement within limited area

---

## 4. Metropolitan Area Network (MAN)

### Definition

A **Metropolitan Area Network (MAN)** spans a city or large campus, connecting multiple LANs within a metropolitan area.

### Characteristics

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        CITY/METROPOLITAN AREA                           │
│                                                                         │
│   ┌─────────────┐              ┌─────────────┐                         │
│   │   BRANCH    │              │    HEAD     │                         │
│   │   OFFICE    │              │   OFFICE    │                         │
│   │   (LAN)     │              │   (LAN)     │                         │
│   └──────┬──────┘              └──────┬──────┘                         │
│          │                            │                                 │
│          │         ┌────────┐         │                                 │
│          └─────────┤  MAN   ├─────────┘                                 │
│                    │BACKBONE│                                           │
│          ┌─────────┤ (Fiber)├─────────┐                                 │
│          │         └────────┘         │                                 │
│          │                            │                                 │
│   ┌──────┴──────┐              ┌──────┴──────┐                         │
│   │  WAREHOUSE  │              │   DATA      │                         │
│   │   (LAN)     │              │   CENTER    │                         │
│   └─────────────┘              │   (LAN)     │                         │
│                                └─────────────┘                         │
│                                                                         │
│                  Coverage: 5-50 km (City-wide)                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Technologies Used

| Technology | Description | Speed |
|------------|-------------|-------|
| **Fiber Optic** | High-speed backbone | Up to 100 Gbps |
| **Cable TV Network** | Existing CATV infrastructure | Up to 1 Gbps |
| **WiMAX** | Wireless MAN standard | Up to 1 Gbps |
| **Metro Ethernet** | Ethernet over metro area | Up to 10 Gbps |

### Applications

- City-wide Wi-Fi networks
- Cable TV distribution
- Connecting university campuses
- Government and public services network
- Inter-connecting bank branches

### Real-World Examples

1. **Cable TV Networks** - Distribute content across cities
2. **City-wide Wi-Fi** - Free public internet access
3. **University Networks** - Connecting multiple campus buildings
4. **Smart City Infrastructure** - IoT sensors and monitoring

---

## 5. Wide Area Network (WAN)

### Definition

A **Wide Area Network (WAN)** spans large geographical areas, often countries or continents, connecting multiple LANs and MANs.

### Characteristics

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         GLOBAL WAN EXAMPLE                              │
│                                                                         │
│     NORTH AMERICA                           EUROPE                      │
│    ┌─────────────┐                     ┌─────────────┐                 │
│    │   NEW YORK  │                     │   LONDON    │                 │
│    │   OFFICE    │                     │   OFFICE    │                 │
│    │   (LAN)     │                     │   (LAN)     │                 │
│    └──────┬──────┘                     └──────┬──────┘                 │
│           │                                   │                         │
│           │    ┌─────────────────────────┐    │                         │
│           └────┤  UNDERSEA FIBER CABLE   ├────┘                         │
│                │  (WAN CONNECTION)       │                              │
│           ┌────┤  Leased/Satellite Link  ├────┐                         │
│           │    └─────────────────────────┘    │                         │
│           │                                   │                         │
│    ┌──────┴──────┐                     ┌──────┴──────┐                 │
│    │ LOS ANGELES │                     │   TOKYO     │                 │
│    │   OFFICE    │                     │   OFFICE    │                 │
│    │   (LAN)     │                     │   (LAN)     │                 │
│    └─────────────┘                     └─────────────┘                 │
│                                             ASIA                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### WAN Technologies

| Technology | Description | Typical Use |
|------------|-------------|-------------|
| **Leased Lines** | Dedicated point-to-point | Corporate WAN |
| **MPLS** | Label-based routing | Enterprise networks |
| **Frame Relay** | Packet-switched (legacy) | Legacy networks |
| **ATM** | Cell-based switching | Telecom backbone |
| **VPN** | Encrypted tunnel over internet | Secure remote access |
| **Satellite** | Wireless via satellites | Remote locations |

### Comparison: LAN vs WAN

| Aspect | LAN | WAN |
|--------|-----|-----|
| Coverage | Building/Campus | Countries/Continents |
| Ownership | Private | Public/Private mix |
| Speed | Very High (1-10 Gbps) | Lower (few Mbps to Gbps) |
| Error Rate | Very Low | Higher |
| Cost | Low | High |
| Setup Time | Quick | Long |
| Congestion | Less | More |

### The Internet as a WAN

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          THE INTERNET                                   │
│                   (World's Largest WAN)                                 │
│                                                                         │
│   ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐         │
│   │  ISP 1  │─────│  ISP 2  │─────│  ISP 3  │─────│  ISP 4  │         │
│   └────┬────┘     └────┬────┘     └────┬────┘     └────┬────┘         │
│        │               │               │               │               │
│    ┌───┴───┐       ┌───┴───┐       ┌───┴───┐       ┌───┴───┐          │
│    │ Home  │       │Company│       │School │       │Govt   │          │
│    │ LAN   │       │ LAN   │       │ LAN   │       │ LAN   │          │
│    └───────┘       └───────┘       └───────┘       └───────┘          │
│                                                                         │
│    Billions of interconnected networks worldwide                        │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Other Network Types

### 6.1 Campus Area Network (CAN)

```
┌─────────────────────────────────────────────────────────┐
│                 UNIVERSITY CAMPUS                        │
│                                                          │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐             │
│  │Library  │    │ Admin   │    │ Science │             │
│  │Building │    │Building │    │   Lab   │             │
│  │ (LAN)   │    │ (LAN)   │    │  (LAN)  │             │
│  └────┬────┘    └────┬────┘    └────┬────┘             │
│       │              │              │                   │
│       └──────────────┼──────────────┘                   │
│                      │                                  │
│              ┌───────┴───────┐                          │
│              │  Campus Core  │                          │
│              │    Network    │                          │
│              └───────┬───────┘                          │
│                      │                                  │
│              ┌───────┴───────┐                          │
│              │   To WAN/     │                          │
│              │   Internet    │                          │
│              └───────────────┘                          │
└─────────────────────────────────────────────────────────┘
```

- Larger than LAN, smaller than MAN
- Connects multiple buildings in a campus
- Common in universities and corporate campuses

### 6.2 Storage Area Network (SAN)

```
┌─────────────────────────────────────────────────────────┐
│                  STORAGE AREA NETWORK                    │
│                                                          │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                 │
│  │Server 1 │  │Server 2 │  │Server 3 │                 │
│  └────┬────┘  └────┬────┘  └────┬────┘                 │
│       │            │            │                       │
│       └────────────┼────────────┘                       │
│                    │                                    │
│            ┌───────┴───────┐                            │
│            │  SAN Switch   │ ◄── High-speed Fiber       │
│            │(Fibre Channel)│                            │
│            └───────┬───────┘                            │
│                    │                                    │
│       ┌────────────┼────────────┐                       │
│       │            │            │                       │
│  ┌────┴────┐  ┌────┴────┐  ┌────┴────┐                 │
│  │ Disk    │  │ Tape    │  │ Disk    │                 │
│  │ Array 1 │  │ Library │  │ Array 2 │                 │
│  └─────────┘  └─────────┘  └─────────┘                 │
│                                                          │
│     Purpose: High-speed access to storage               │
└─────────────────────────────────────────────────────────┘
```

- Dedicated network for storage devices
- Uses Fibre Channel or iSCSI
- Provides block-level storage access

### 6.3 Virtual Private Network (VPN)

```
┌─────────────────────────────────────────────────────────┐
│                   VPN OVER INTERNET                      │
│                                                          │
│  ┌─────────────┐                    ┌─────────────┐     │
│  │ Home Office │                    │ Corporate   │     │
│  │   (LAN)     │                    │ Network     │     │
│  └──────┬──────┘                    └──────┬──────┘     │
│         │                                  │            │
│    ┌────┴────┐                        ┌────┴────┐       │
│    │   VPN   │                        │   VPN   │       │
│    │  Client │                        │  Server │       │
│    └────┬────┘                        └────┬────┘       │
│         │                                  │            │
│         │      ╔═══════════════════╗       │            │
│         └──────║  ENCRYPTED TUNNEL ║───────┘            │
│                ║   over Internet   ║                    │
│                ╚═══════════════════╝                    │
│                                                          │
│       Creates secure "private" link over public network │
└─────────────────────────────────────────────────────────┘
```

- Secure connection over public internet
- Encrypts all traffic in a "tunnel"
- Used for remote work and site-to-site connections

---

## 7. Network Types Comparison

### Complete Comparison Table

| Type | Range | Ownership | Speed | Cost | Example |
|------|-------|-----------|-------|------|---------|
| **PAN** | ~10m | Personal | Low-Med | Very Low | Bluetooth devices |
| **LAN** | ~1km | Private | Very High | Low | Office network |
| **CAN** | ~5km | Private | High | Medium | University |
| **MAN** | ~100km | Public/Private | High | High | City network |
| **WAN** | Global | Public | Varies | Very High | Internet |
| **SAN** | Local | Private | Very High | High | Data center |

### Visual Scale Comparison

```
PAN      LAN        CAN         MAN              WAN
 │        │          │           │                │
 ●────────●──────────●───────────●────────────────●─────────────►
 │        │          │           │                │
10m      1km        5km        100km           Global

Examples:
├────────┼──────────┼───────────┼────────────────┼────────────►
Bluetooth Office   University  City Network    Internet
Headset  Building  Campus      Cable TV        Global Corp
```

---

## 📊 Summary Table

| Concept | Key Points |
|---------|------------|
| **PAN** | Personal devices, ~10m range, Bluetooth/USB |
| **LAN** | Building/campus level, high speed, Ethernet/WiFi |
| **MAN** | City-wide, connects LANs, fiber/WiMAX |
| **WAN** | Global coverage, connects distant LANs, Internet |
| **CAN** | Campus network, bridges LAN and MAN |
| **SAN** | Storage-specific, high-speed, data centers |
| **VPN** | Virtual secure tunnel over public network |

---

## ❓ Quick Revision Questions

1. **Q:** Arrange these networks in order of increasing geographical coverage: WAN, PAN, MAN, LAN.
   <details>
   <summary>Answer</summary>
   PAN → LAN → MAN → WAN
   </details>

2. **Q:** What technology is commonly used in PAN?
   <details>
   <summary>Answer</summary>
   Bluetooth is the most common technology used in Personal Area Networks. Other technologies include USB, IrDA, and NFC.
   </details>

3. **Q:** What is the main difference between LAN and WAN in terms of ownership?
   <details>
   <summary>Answer</summary>
   LANs are typically privately owned and operated by a single organization. WANs often use public telecommunications infrastructure (like leased lines from telecom providers) though may be privately managed.
   </details>

4. **Q:** Give two examples of MAN applications.
   <details>
   <summary>Answer</summary>
   1. Cable TV networks distributing content across a city
   2. City-wide WiFi networks for public internet access
   (Also: University campus networks, connecting bank branches in a city)
   </details>

5. **Q:** What is a Storage Area Network (SAN) and where is it used?
   <details>
   <summary>Answer</summary>
   A SAN is a dedicated high-speed network that connects servers to storage devices. It provides block-level storage access and is primarily used in data centers for applications requiring fast, reliable access to large amounts of data.
   </details>

6. **Q:** How does a VPN create a private network over the public internet?
   <details>
   <summary>Answer</summary>
   A VPN creates an encrypted "tunnel" through the public internet. All data transmitted through this tunnel is encrypted, making it private and secure even though it travels over public infrastructure.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Basic Concepts](01-basic-concepts.md) | [Table of Contents](../README.md) | [Network Topologies →](03-network-topologies.md) |

---

*© 2026 Computer Networking Study Notes. For educational purposes.*
