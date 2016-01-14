ansible-firewall
=========

This role manages firewall rules for linux systems using **iptables** (netfilter).  
You will able to design you ruleset using role vars or overide the
whole template and install your own.

Supported Platforms
-------------------

* RHEL 5, 6
* Ubuntu
* Debian

It will likely run on other platforms, just drop in vars/ a new file to support your os variant, vars are parsed in the following order/format:
* {{ ansible_distribution }}_{{ ansible_distribution_major_version }}.yml
* {{ ansible_distribution }}.yml
* {{ ansible_os_family }}_{{ ansible_distribution_major_version }}.yml
* {{ ansible_os_family }}.yml"

Requirements
------------

None at this time.

Default Role Variables
--------------

By default this role initialize the firewall with an empty ruleset (allow any/any), however variables can be passed to this role
and configuration can be overridden, for additional informations please have a look to **defaults/main.yml**


Dependencies
------------

None at this time.

Example Playbook
----------------
1) Just install needed packages and setup an empty ruleset
```yaml
- hosts: all
  remote_user: root
  sudo: no
  - roles: 
    - { role: firewall }
```
2) Install needed packages, and setup a custom ruleset
```yaml
- hosts: all
  remote_user: root
  sudo: no
  vars:
    firewall_chain_input:
      - -m state --state RELATED,ESTABLISHED -j ACCEPT
      - -s 192.168.99.0/24 -d 192.168.202.119 -m comment --comment "allow foo to bar" -j ACCEPT
    firewall_chain_forward:
      - -s 10.3.22.19 -d 192.168.200.0/24 -p udp -m multiport --dports 13000,13111 -j REJECT
    firewall_chain_nat_prerouting:
      - -d 10.3.200.0/21 -p udp -m udp --dport 53 -j DNAT --to-destination 192.168.1.1
    firewall_chain_nat_postrouting:
      - -d 192.168.1.2/32 -o eth3 -p udp -j SNAT --to-source 192.168.1.200
      - -s 192.168.1.0/24 -j SNAT --to-source 192.168.1.254
    firewall_extra_chains:
      - { name: "LOGDROP", append: { "-j LOG --log-prefix 'iptables DROP:' --log-level 7", "-j DROP"} }
      - { name: "FOO", append: { "-j RETURN" } }
  - roles: 
    - { role: firewall }
```

3) Install needed packages, and override the whole template.
```yaml
- hosts: all
  remote_user: root
  sudo: no
  vars:
    firewall_script_template: templates/production_rules.sh.j2
  - roles: 
    - { role: firewall }
```

License
-------

GPLv2

Author Information
------------------

Alessio Cassibba (x-drum) http://blog.zerodev.it.
