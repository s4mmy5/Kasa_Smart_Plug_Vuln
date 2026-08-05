# Basic Information
Name: Kasa Smart Wi-Fi Plug Mini
Model: EP10P2
S/N: Y264328X03277
P/N: 0184600821
Wifi: 802.11b/g/n
FCC ID: 2AXJ4EP10
IC ID: 26583-EP10

![Image of Kasa Smart Wi-Fi Plug Mini as one of the first search results on Amazon](./assets/Amazon_Listing.png)

# FCC ID Information
FCC ID: 2AXJ4EP10
Filing: https://fccid.io/2AXJ4EP10

# Internals
SoC: RTL8710CF

## UART Pins
Power pins are P1, P2, P3.
TP1, TP2, TP3, TP4, TP5, TP6 pins are also available.

Ground: TP1, P3

Voltages:
P1: 1.9 V
P2: 0 V
P3: Ground
TP1: Ground
TP2: 4.5 V
TP3: 1.6 V
TP4: 4.9 V
TP5: 1.6 V
TP6: 1.6 V

## NEXT STEPS (notes)
1. Wireshark capture from gl.inet router works but traffic is encrypted with TLS.
2. Creating pre-master-key file for wireshark decryption might not be plausible. Another option might be using a MITM proxy and adding a trusted self-signed CA to Android phone.
3. Unable to determine the function of TP pins. Logic Analyzer may be needed.
