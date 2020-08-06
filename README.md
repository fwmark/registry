The firewall mark registry is a registry for software that uses the
packet or connection mark features of Linux's packet filter system
(Netfilter, sometimes colloquially called iptables or nftables after
the userspace tools).

There are two registries, one for bitwise users and one for whole-mark
users. For information on how they differ, and which you should use
for new applications, check further down the page.

Allocations apply to both packet mark and connection mark (connmark)
uses, since the most common use of connmark is copying back and forth
to/from packet marks.

Like most registries on the internet, this list is purely informative,
and non-binding. It's attempting to document current uses, to help
developers of new software pick some non-interfering values.

## Bitwise Mark Registry

Bits are numbered from 0 to 31, least-significant bit (LSB) to
most-significant (MSB). For example, if only mark bit number 3 is set,
the overall packet mark is 0x8. For search engine discoverability, the
full mark value with individual bits set is also listed in the form
that people are likely to search for.

| Bits       | Mark mask  | Software               | Source                                  |
|------------|------------|------------------------|-----------------------------------------|
| 0-12,16-31 | 0xFFFF1FFF | [Cilium][cilium]       | [Source code][cilium-src]               |
| 7          | 0x00000080 | [AWS CNI][aws-vpc-cni] | [Source code][aws-vpc-cni-src]          |
| 13         | 0x00002000 | [CNI Portmap][portmap] | [Documentation][portmap-src]            |
| 14-15      | 0x0000C000 | [Kubernetes][k8s]      | [Source code][k8s-src]                  |
| 16-31      | 0xFFFF0000 | [Calico][calico]       | [Documentation][calico-src]             |
| 17-18      | 0x60000    | [Weave Net][weave]     | [Source][weave-src1] [code][weave-src2] |
| 18-19      | 0xC0000    | [Tailscale][ts]        | [Source code][ts-src]                   |


[aws-vpc-cni]: https://github.com/aws/amazon-vpc-cni-k8s/
[aws-vpc-cni-src]: https://github.com/aws/amazon-vpc-cni-k8s/blob/v1.6.3/pkg/networkutils/network.go#L95
[cilium]: https://cilium.io/
[cilium-src]: https://github.com/cilium/cilium/blob/v1.8.2/bpf/lib/common.h#L380-L410
[k8s]: https://kubernetes.io/
[k8s-src]: https://github.com/kubernetes/kubernetes/blob/v1.18.6/pkg/kubelet/apis/config/v1beta1/defaults.go#L34-L35
[portmap]: https://github.com/containernetworking/plugins/
[portmap-src]: https://github.com/containernetworking/plugins/blob/v0.8.6/plugins/meta/portmap/README.md#usage
[ts]: https://www.tailscale.com/
[ts-src]: https://github.com/tailscale/tailscale/blob/v1.1.0/wgengine/router/router_linux.go#L20-L47
[weave]: https://www.weave.works/oss/net/
[weave-src1]: https://github.com/weaveworks/weave/blob/v2.7.0/npc/constants.go#L5
[weave-src2]: https://github.com/weaveworks/weave/blob/v2.7.0/net/ipsec/ipsec.go#L29
[calico]: https://www.projectcalico.org
[calico-src]: https://docs.projectcalico.org/reference/felix/configuration#iptables-dataplane-configuration

## Non-Bitwise Mark Registry

Some software treats the packet mark as a simple integer, and so
sets/clears all bits at once whenever it touches a packet. Such
software is likely to be broadly incompatible with "bitwise" users of
the packet mark.

| Mark value | Software                     | Source                           |
|------------|------------------------------|----------------------------------|
| Any        | [OpenShift][openshift-ovn]   | [Source code][openshift-ovn-src] |
| 0x00000800 | [Antrea][antrea]             | [Documentation][antrea-src]      |
| 0x1337     | [Istio][istio]               | [Documentation][istio-src]       |
| 0x1e7700ce | [AWS AppMesh][aws-appmesh]   | [Documentation][aws-appmesh-src] |

[antrea]: https://github.com/vmware-tanzu/antrea
[antrea-src]: https://github.com/vmware-tanzu/antrea/blob/v0.8.2/docs/policy-only.md#routing
[aws-appmesh]: https://aws.amazon.com/app-mesh/
[aws-appmesh-src]: https://docs.aws.amazon.com/app-mesh/latest/userguide/appmesh-getting-started.html
[istio]: https://istio.io/
[istio-src]: https://github.com/cilium/istio/blob/46d7da707bb8b162ad561a704697eb8aff418463/tools/deb/istio-iptables.sh#L69
[openshift-ovn]: https://github.com/ovn-org/ovn-kubernetes
[openshift-ovn-src]: https://github.com/ovn-org/ovn-kubernetes/blob/2462a724e54b626b69eb1813660f9ed5c0f783f0/go-controller/pkg/node/gateway_iptables.go#L192

## Why do we need a registry?

In Netfilter, individual packets and connections can be marked with a
uint32 worth of metadata. These marks can then be used elsewhere in
the network stack to make policy decisions. For example, a
Netfilter-based firewall may mark packets that came from a certain
interface, so that the policy routing subsystem can then apply a
special routing table to those packets.

Typically, the packet mark is used bit-wise: rather than operate on
the literal value of the uint32, software reads and writes specific
bits of the mark. That means that there are 32 bits available for use
by software, and unfortunately picking the same bit(s) as someone else
will result in the two pieces of code interfering with each other.

The registry is an attempt to document the mark bits that are
currently in use, so that developers of new networking software can
(hopefully) avoid conflicts and hard to debug problems.

## What's with bitwise and non-bitwise marks?

Originally, packet marks could only be set as entire values, i.e. a
whole 32-bit unsigned integer. This allowed for 2^32 distinct values,
but each packet could only be marked with a single value.

Later, bitwise matching and setting operations were introduced. In the
bitwise mode, only 32 values are available, but they can OR together
gracefully.

Empirically, both styles are in use in the wild - sometimes both
within the same ecosystem. For example, Kubernetes and most of its
ecosystem use bitwise marking, but Istio uses non-bitwise ("whole
mark") markings for packets that it intercepts.

## Which should I use?

It depends. Unfortunately, neither option is perfect, so it often
comes down to what you need or want to interoperate with.

As a rule: software using bitwise marking is "compatible" with other
software that uses bitwise marking, as long as they use different
bits. "Compatible" is in quotes because the behaviors triggered by the
other software's marking may still interfere with your
functionality. Alas, there's no way to be sure.

Software using non-bitwise marking is not compatible with either
bitwise or other non-bitwise marking software on the same system, as
they'll all clobber each others' values. The only exception is if all
the data paths that trigger packet marking are completely distinct
from each other, such that every packet only ever gets zero or one
mark applied to it.

For example, if one piece of software only marks outbound packets, and
another only marks inbound packets, these can coexist peacefully... As
long as they don't use connmarks, which would cause crosstalk between
input and output.

The answer, then, is that there is no good answer. If you need maximum
interoperability with other software:
 - Use bitwise marks, and pick bits that don't conflict with other
   software. (and send us a PR to register your use!)
 - For each other piece of software you need to work with, manually
   verify that your use of marks and theirs don't interfere with each
   other.
 - For other software using non-bitwise marks, additionally verify
   that packets can only ever receive your mark XOR the other's
   mark. Datapaths where both might apply will almost definitely break
   stuff.

If you need no interoperability at all, or you're confident that your
datapath could never reasonably have other behavior applied to it, use
non-bitwise marks and pick a random number for your mark value (and
send us a PR to register it!). However, be cautious in assuming that
no other software could ever reasonably want to alter the processing
of "your" packets. People are inventive, and by using non-bitwise
marks you're effectively locking yourself into not permitting those
uses.

## Contribution Guidelines

The registry is "open", meaning anyone can register mark bits by
sending a pull request.

- Try to avoid using the same bits as someone else. If that's not
  possible (e.g. if you're documenting what proprietary software is
  doing and you can't alter it, or all bits have been claimed
  already), do your best to pick bits that are least likely to be
  already in use by users of your software.
- Add yourself to AUTHORS in the same pull request as your initial
  registration.
