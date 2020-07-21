The Firewall Mark registry is an unofficial registry for software that
uses the packet or connection mark features of Linux's packet filter
system (Netfilter, sometimes colloquially called iptables or nftables
after the userspace tools).

[View the registry](Registry.md)

## Why do we need a registry?

In Netfilter, individual packets and connections can be marked with a
uint32 worth of data. These marks can then be used elsewhere in the
network stack to make policy decisions. For example, a Netfilter-based
firewall may mark packets that came from a certain interface, so that
the policy routing subsystem can then apply a special routing table to
those packets.

Typically, the packet mark is used bit-wise: rather than operate on
the literal value of the uint32, software reads and writes specific
bits of the mark. That means that there are 32 bits available for use
by software, and unfortunately picking the same bit(s) as someone else
will result in the two pieces of code interfering with each other.

This registry is an attempt to document the mark bits that are
currently in use, so that developers of new networking software can
(hopefully) avoid conflicts and hard to debug problems.

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
