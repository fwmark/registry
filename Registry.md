# The Firewall Mark Registry

Bits are reserved for both packet mark (mark) and connection mark
(connmark) use, since the most common use of connmark is to copy the
connmark into the packet mark.

Bits are numbered from least-significant bit (LSB) to most-significant
(MSB). For example, if only mark bit number 3 is set, the overall
packet mark is 0x4.

| Bit | Software |
|-----|----------|
| 15 | Kubernetes |
| 16 | Kubernetes |
| 18 | Weave Net |
| 19 | Tailscale |
| 20 | Tailscale |
