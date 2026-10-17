# Home Segmentation Network

## Overview

This project documents my home lab for practicing Linux networking, routing, DHCP, NAT, firewall rules, troubleshooting, and network segmentation.

The goal is to build the network step by step and relate the concepts to real-world firewall and network administration.

## Phase 1 — Ubuntu Router and Firewall

Phase 1 uses Ubuntu Server as a router/firewall between a trusted network and a separate Lab/User network.

### Lab Design

```text
Trusted / Upstream Network
10.10.10.0/24
        |
     Ubuntu Server
        |
Lab / User Network
10.20.20.0/24
        |
    Lab Client


The Ubuntu server uses separate network interfaces for the trusted/upstream side and the Lab network.
A third network interface is currently unused and available for future testing.



