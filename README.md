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

This role happens to *not* require the python bindings for `firewalld`.
(None of this role could be implemented using the Ansible `firewalld`
module).  This means that you do not need to force Ansible to use
python3 on Fedora 28 and above.

(You should probably start using python3 anyway though :-).

`firewalld` recommends that it be used with NetworkManager.  I do not
guarantee it will work correctly for you without NetworkManager.
I use this role [without NetworkManager, on Debian][1].  As far as I
can tell, the features I rely on *should* work.  However the
documentation is confusing, `firewalld` code assumes RedHat-specific
details, and some `firewall-cmd` queries will show confusing results.

[1] https://unix.stackexchange.com/questions/497697/firewall-cmd-says-no-firewall-zones-are-active-why


## Status

This README is too long.
In theory, this role could be extended to work around some of the issues.

### Services that should not be used

To see the list of pre-defined `firewalld` "services", look in
`/usr/lib/firewalld/services/`.

Do not enable the service "upnp-client", unless you know exactly
what it does.  This is explained in the link in the uPnP section
below.  Really, you should be very suspicious about the definition
of any "-client" service.

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
simple hacks you can use, but not in other cases.  I note this
breaks functionality of the Gnome app Transmission.  If NAT-PMP
port forwarding is available, Transmission will use that instead.

https://unix.stackexchange.com/questions/543612/transmission-gnome-bittorrent-client-v-s-firewall-on-debian-10/

### Lack of error checking

In general, the `firewalld` software seems to be missing some
error checking or error reporting.  It seems unfortunate to write
so much infrastructure and not make it fail-fast.

Please carefully test the configurations you apply.  Syntax errors
may cause a revert to the default zone.  Again, this will probably
not lock you out, as long as you are connecting using the default
port for ssh.

You should be able to see if there are any errors by looking in
the system log, e.g. `systemctl status firewalld`.


### Memory usage

`firewalld` is said to cost 20MB of RAM.  This is for understandable
reasons, but is not strictly necessary.

https://github.com/firewalld/firewalld/issues/337#issuecomment-389086797


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
