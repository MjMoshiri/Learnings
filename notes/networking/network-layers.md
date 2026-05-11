---
title: "Network Layers"
tags: [ "networking", "osi-model", "tcp-ip-model" ]
prerequisites: []
status: "draft"
last_updated: "2025-08-19"
todos:
  - "DONE: Detail each layer of the OSI model."
  - "Detail each layer of the TCP/IP model."
  - "Add a section comparing the OSI and TCP/IP models."
  - "Include a diagram illustrating the layers."
---

## Summary

Network layers are a conceptual framework used to standardize the functions of a telecommunication or computing network. By dividing the complex set of functions and protocols into a series of layers, it simplifies network design, troubleshooting, and implementation. The two most common models are the OSI (Open Systems Interconnection) model and the TCP/IP model.

## Core Concepts

The OSI (Open Systems Interconnection) model is a conceptual framework that standardizes the functions of a telecommunication system into seven abstract layers. Each layer serves the layer above it and is served by the layer below it.

### The 7 Layers of the OSI Model

1.  **Layer 7: Application**
    -   **Function:** The highest layer, responsible for providing network services directly to the end-user's applications. This includes protocols for email (SMTP), web browsing (HTTP), and file transfers (FTP).
    -   **Data Unit:** Data.
    -   **Example Protocols:** HTTP, FTP, SMTP, DNS.

2.  **Layer 6: Presentation**
    -   **Function:** Translates, encrypts, and compresses data. It ensures that data sent from the application layer of one system can be read by the application layer of another system.
    -   **Data Unit:** Data.
    -   **Example Functions:** Data encryption/decryption, character encoding translation (e.g., ASCII to EBCDIC), data compression.

3.  **Layer 5: Session**
    -   **Function:** Manages sessions between applications. It establishes, maintains, and terminates connections (sessions) between a local and remote application.
    -   **Data Unit:** Data.
    -   **Example Protocols:** NetBIOS, PPTP.

4.  **Layer 4: Transport**
    -   **Function:** Provides reliable data transfer between end systems. It handles flow control, error checking, and segmentation or reassembly of data. The two most common protocols are TCP (connection-oriented) and UDP (connectionless).
    -   **Data Unit:** Segment (TCP), Datagram (UDP).
    -   **Example Protocols:** TCP, UDP.

5.  **Layer 3: Network**
    -   **Function:** Responsible for logical addressing and routing. It determines the best path to move data from a source to a destination across multiple networks. This is where IP (Internet Protocol) operates.
    -   **Data Unit:** Packet.
    -   **Example Protocols:** IP, ICMP, OSPF.

6.  **Layer 2: Data Link**
    -   **Function:** Provides node-to-node data transfer and handles error correction from the physical layer. It is divided into two sublayers: the Media Access Control (MAC) layer and the Logical Link Control (LLC) layer. This is where switches operate.
    -   **Data Unit:** Frame.
    -   **Example Protocols:** Ethernet, PPP, MAC addresses.

7.  **Layer 1: Physical**
    -   **Function:** The lowest layer, responsible for the physical transmission of raw data bits over a communication medium. This includes cables (copper, fiber), hubs, and the electrical and physical specifications for the devices.
    -   **Data Unit:** Bit.
    -   **Example Hardware:** Ethernet cables, Hubs, Repeaters.


## Use Cases

-   **Standardization:** Allows different vendors to create interoperable hardware and software.
-   **Troubleshooting:** Helps network engineers isolate problems to a specific layer.
-   **Learning:** Provides a structured way to learn and teach networking concepts.

## References

(TODO: Add references here)
