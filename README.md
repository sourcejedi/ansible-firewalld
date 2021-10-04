# firewalld #

This role configures `firewalld` with one zone, and sets the list of
allowed services and ports.

It supports change over time.  Removing an allowed service or port will
have the desired effect.

We create the requested configuration before `firewalld` is started.
This means the default configuration from the `firewalld` package is
not used at any point.  As a result, you can use this role even if you
have set up Ansible to connect through a non-default port.  Other
roles might require you to use the default ssh / port 22 connection
(or to run locally instead), because that port is allowed by the
default configurations from the `firewalld` package.

Please read this entire document, so you will know the full set of
disclaimers :-).

I would be interested to hear any alternative, which allows good
support for removing ports and for IPv6.  I.e. default rules or macros
for IPv6, and no strange need to duplicate rules for IPv4 v.s. IPv6.


## Requirements

This should work on any distribution that has a package for
`firewalld`.  It has been tested on Debian and Fedora.

`firewalld` recommends that it be used with NetworkManager.  I do not
guarantee it will work correctly for you without NetworkManager.
I use this role without NetworkManager, on Debian.  As far as I
can tell, the features I rely on *should* work.  However the
documentation is confusing, `firewalld` code assumes RedHat-specific
details, and some `firewall-cmd` queries will show
confusing results:

https://unix.stackexchange.com/questions/497697/firewall-cmd-says-no-firewall-zones-are-active-why

If you run a firewall inside a container like `systemd-nspawn`,
it might not be able to load the conntrack kernel modules it needs.
This issue might be particularly prominent during the transition
from iptables to nftables.

This role happens *not* to require the python bindings for `firewalld`.
This means that you do not need to force Ansible to use python3 on
recent systems.  (You should use python3 anyway though :-).

## Status

This README is too long.
In theory, this role could be extended to work around some of the issues.

### Services that should not be used

To see the list of pre-defined `firewalld` "services", look in
`/usr/lib/firewalld/services/`.

Do not enable the service "upnp-client", unless you know exactly
what it does.  This is explained in the link in the uPnP section
below.  In general, you should be very suspicious about the
definition of any "-client" service.

Also note that if there is a "helper" with the same name as an
enabled "service", `firewalld` will automatically enable the
"helper".  These are the conntrack modules *which parse the relevant
protocol inside the kernel*.  This is NOT good for your security.
The "helpers" are defined in `/usr/lib/firewalld/helpers/`.

You could also browse these files in the upstream source tree:

https://github.com/firewalld/firewalld/tree/master/config/

### Breaks uPnP, including uPnP port forwarding

Most Linux firewalls break uPnP, including uPnP port forwarding.
This includes `firewalld`.  For certain uses of uPnP there are
simple hacks you can use, but not in other cases.  I notice this
breaks functionality of Transmission.  If NAT-PMP port forwarding
is available, Transmission will use that instead.

https://unix.stackexchange.com/questions/543612/transmission-gnome-bittorrent-client-v-s-firewall-on-debian-10/

### Interaction with libvirt virtual machines

In recent versions of firewalld, your virtual machines are subject to
a different set of rules ("libvirt zone").  In general, this prevents
them connecting to the host machine.  This is probably fine for most
uses.  However, if you have been connecting to VMs using MDNS
(`ssh my-virtual-machine.local`), that will no longer work.  This
can easily be worked around, or adjusted.  See:

* [sourcejedi.libvirt_nss](https://github.com/sourcejedi/ansible-libvirt_nss)
* https://unix.stackexchange.com/questions/602512/how-to-connect-to-libvirt-vms-by-name-instead-of-ip-address/602513

### Error checking might not work as expected

Please carefully test the configurations you apply.  An error in the
role could cause a revert to the default zone.  This would probably
not lock you out, as long as you are connecting using the default port
for ssh.

This role includes some checks e.g. that the service names you specify
are valid, although this does not work on old versions of firewalld.
Ideally you would also check for errors by by looking in the system
log, e.g. `systemctl status firewalld`.  On old systems, you might
also see warnings about iptables --delete commands, particularly
regarding "LIBVIRT" - this seems to be normal.

In general, the `firewalld` software seems to be missing some
error checking or error reporting.  It seems a little strange to
write so much infrastructure, and not make it fail-fast.


### Memory usage

`firewalld` is said to cost 20MB of RAM.  This is for understandable
reasons, but does not seem to be strictly necessary.  (See
[issue 337](https://github.com/firewalld/firewalld/issues/337#issuecomment-389086797))


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


## License

The role itself is licensed GPLv3.  Please open an issue if this creates any problem.
