# firewalld #

This role configures firewalld with one zone, and sets the list of
allowed services and ports.  It supports change over time.  Removing an
allowed service or port will have the desired effect.

firewalld is said to cost 20MB of RAM.  This is for understandable
reasons, but it is not strictly necessary.

https://github.com/firewalld/firewalld/issues/337#issuecomment-389086797

(I would be interested in any alternative, which has good support for
removing ports and IPv6.  I.e. default rules / macros for IPv6, and
no strange need to duplicate IPv4 v.s. IPv6 rules).
