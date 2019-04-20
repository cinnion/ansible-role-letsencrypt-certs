# letsencrypt-certs

An Ansible Role which allows for maintaining SSL certificates via
certbot or other clients on a single administrative machine, and
pushing them to the servers.

With redundant servers becoming a norm even for small businesses, it
is not unheard of to want to have the same certificate installed on a
service running on two, three or more services for load ballancing or
availability purposes. However, when combined with services such as
[Let's Encrypt](https://letsencrypt.org)'s CA authority and clients
such as certbot, there is still a disconnect when it comes to
certificate maintenance. The certificates need to be renewed at a
single point of authority and distributed from there. And that is
where this role comes onto the stage.

## Requirements

This role has been developed using Ansible 2.6, and presently only works with
RHEL/CentOS 6.x and 7.x. Adapting it to other distros, or even other
OSes should not be an issue; it is just that I do not presently use
anything else.

It also does not address the setup/execution of a client such as
[cerbot](https://certbot.eff.org/), nor does it address the initial
"installation" of the certificates and the software configuration
which goes with it. It mearly consolidates the renewal into one
location where the certificates can be easily backed up, and where a
single renewal for shared certificates can take place.

N.B. There is a near-term goal of hopefully adding support for the following:

* DELL DRAC 5 (using racadm)
* FreeBSD
  - FreeNAS
  - OPNSense

## Role Variables:

**NOTE:** This role uses the [ansible-merge-vars plugin](https://github.com/leapfrogonline/ansible-merge-vars)
to handle variables coming from the host variables as well as group variables
for an indeterminate number of groups. So for example, where
`certificates__to_merge` is used, this could also include variables such as
`ldap_certificates__to_merge` and `web_certificates__to_merge`, which will be
merged within the role, rather than having one variable which gets overwritten.

So, if a host has:

        certificates__to_merge:
            - certificate-a
            - certificate-b

and one group to which it belongs has:

        ldap_certificates__to_merge:
            - certificate-c

and a second group has:

        web_certificates__to_merge:
            - certificate-b
            - certificate-d

These will be combined into the variable `certificates`, which would be the same
as (order possibly varying):

        certificates:
            - certificate-a
            - certificate-b
            - certificate-c
            - certificate-d

-----

The following host/group-specific variables are used, and are defined either in
the host variables, or in the variables for associated groups.


        certificates__to_merge:
           - www.example.com

or

        certificates__to_merge:
          - www.example.com
          - foo.example.net

Defines the certificates to be distributed to a given host and/or group, merging
as discussed above. It may be defined in any location where host/group variables
may be defined. *The values given must be a list.*


        certificate_services__to_merge:
            - httpd

or

        ldap_certificate_services__to_merge:
            - slapd

The list of services which will be restarted if a new certificate is pushed to
the host. If no services are defined, then a default of `httpd` is used, but it should be
noted that if any service is specified in either the host or group variables and
`httpd` is needed, it must be explicitly specified.

**NOTE:** At this point, pushing a certificate for one service will result in
all services which use certificates being restarted. So the above would be
combined into the internal `certificate_services` variable, and `slapd` would be
restarted even if just a certificate associated with a web server was pushed.

## Dependencies

- [ansible-merge-vars](https://github.com/leapfrogonline/ansible-merge-vars)

## Example Playbooks

See the examples found in the `examples` directory.

## License

This software is open-sourced software licensed under the
[Apache 2.0 license](http://www.apache.org/licenses/LICENSE-2.0).

## Author Information

This role was created 2018 Dec 1 by [Douglas Needham](https://www.ka8zrt.com/).
