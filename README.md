# firewalld #

This role configures firewalld with one zone, and sets the list of allowed services.
It supports change over time.  Removing an allowed service will have the desired effect.

firewalld is said to cost 20MB of RAM.  This is for understandable reasons,
but it is not strictly necessary.

https://github.com/firewalld/firewalld/issues/337#issuecomment-389086797
