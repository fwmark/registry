# The Firewall Mark Registry

Bits are reserved for both packet mark (mark) and connection mark
(connmark) use, since the most common use of connmark is to copy the
connmark into the packet mark.

Bits are numbered from least-significant bit (LSB) to most-significant
(MSB). For example, if only mark bit number 3 is set, the overall
packet mark is 0x4. For search engine discoverability, the full mark
value with individual bits set is also listed in the form that people
are likely to search for.

| Bit | Mark value | Software |
|-----|-----------|----------|
| 15 | 0x4000 | Kubernetes |
| 16 | 0x8000 | Kubernetes |
| 18 | 0x20000 | Weave Net |
| 19 | 0x40000 | Tailscale |
| 20 | 0x80000 | Tailscale |
