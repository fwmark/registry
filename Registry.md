# The Firewall Mark Registry

Bits are reserved for both packet mark (mark) and connection mark
(connmark) use, since the most common use of connmark is to copy the
connmark into the packet mark.

Bits are numbered from 0 to 31, least-significant bit (LSB) to
most-significant (MSB). For example, if only mark bit number 3 is set,
the overall packet mark is 0x8. For search engine discoverability, the
full mark value with individual bits set is also listed in the form
that people are likely to search for.

| Bit | Mark value | Software |
|-----|-----------|----------|
| 8 | 0x100 | [Cilium][cilium] |
| 9 | 0x200 | [Cilium][cilium] |
| 10 | 0x400 | [Cilium][cilium] |
| 11 | 0x800 | [Cilium][cilium] |
| 14 | 0x4000 | [Kubernetes][k8s] |
| 15 | 0x8000 | [Kubernetes][k8s] |
| 17 | 0x20000 | [Weave Net][weave] |
| 18 | 0x40000 | [Tailscale][ts] |
| 19 | 0x80000 | [Tailscale][ts] |

[cilium]: https://cilium.io/
[k8s]: https://kubernetes.io/
[ts]: https://www.tailscale.com/
[weave]: https://www.weave.works/oss/net/

# Non-bitwise users of packet marks

Some software treats the packet mark as a simple integer, and so
sets/clears all bits at once whenever it touches a packet. Such
software is likely to be broadly incompatible with "bitwise" users of
the packet mark.

| Mark value | Software |
|------------|----------|
| 0x1337 | [Istio][istio] |
| 0x1e7700ce | [AWS AppMesh][aws-appmesh] |

[aws-appmesh]: https://aws.amazon.com/app-mesh/
[istio]: https://istio.io/
