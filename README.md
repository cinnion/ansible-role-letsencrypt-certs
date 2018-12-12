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

The following host/group-specific variables are used, and are defined
outside of the scope of this role:

        certificates:
           - www.example.com

or

        certificates:
          - www.example.com
          - foo.example.net

Defines the certificates to be distributed to a given host or
group. It may be defined in any location where host/group variables
may be defined. *The value given must be a list.*

## Dependencies

None.

## Example Playbooks
### Playbook #1

```
    ---
    - hosts: ldaps
      roles: { role: letsencrypt-certs }
      notify: restart slapd

    - hosts: webservers
      roles: { role: letsencrypt-certs }
      notify: restart httpd

    ...
```

*Inside `group_vars/webservers`*

```
    ---
    certificates:
      - www.example.com
      - foo.example.net
    ...

### Playbook #2

```
---
- hosts: all
  gather_facts: False

  pre_tasks:
    - setup:
        gather_subset: min

  roles:
    - role: letsencrypt-certs
      when: ansible_facts['os_family'] == 'RedHat'
...
```

## License

This software is open-sourced software licensed under the
[Apache 2.0 license](http://www.apache.org/licenses/LICENSE-2.0).

## Author Information

This role was created 2018 Dec 1 by [Douglas Needham](https://www.ka8zrt.com/).

