# firewalld #

This role configures `firewalld` with one zone, and sets the list of
allowed services and ports.

It supports change over time.  Removing an allowed service or port will
have the desired effect.

The config is created before `firewalld` is started.  Usually this does
not matter, because the default zone allows ssh / port 22.  It might be
useful in case you use a different ssh port.

`firewalld` is said to cost 20MB of RAM.  This is for understandable
reasons, but it is not strictly necessary.

https://github.com/firewalld/firewalld/issues/337#issuecomment-389086797

(I would be interested to hear any alternative, which allows good
support for removing ports and for IPv6.  I.e. default rules or macros
for IPv6, and no strange need to duplicate rules for IPv4 v.s. IPv6).


## Requirements

This should work on any distribution that has a package for
`firewalld`.  It has been tested on Debian and Fedora.

This role happens to *not* require the python bindings for `firewalld`.
(None of this role could be implemented using the Ansible `firewalld`
module).  This means that you do not need to force Ansible to use
python3 on Fedora 28 and above.

(You should probably start using python3 anyway though :-).


## Example playbook

    - hosts: all
      roles:
       - role: firewalld
         firewalld__services:
          - dhcpv6-client
          - mdns
          - ssh
         firewalld__ports:
          # apt-cacher-ng
          - protocol: tcp
            port: 3142

To see the list of `firewalld` "services", look in
`/usr/lib/firewalld/services/`.

Note that if there is a "helper" with the same name as an enabled
"service", `firewalld` will automatically enable the "helper".
These are the conntrack modules *which parse the relevant protocol
inside the kernel*.  This is not desirable for a security feature.
The "helpers" are defined in `/usr/lib/firewalld/helpers/`.

You could also browse these files in the upstream source tree:

https://github.com/firewalld/firewalld/tree/master/config/


## License

The role itself is licensed GPLv3.  Please open an issue if this creates any problem.
