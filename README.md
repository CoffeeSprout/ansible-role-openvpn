coffeesprout.openvpn
=========

Installs OpenVPN, creates the CA, creates client certificates, and supports certificate revocation.

Requirements
------------

Tested with FreeBSD 13 (not jail ready yet)
Tested with Debian 11

Role Variables
--------------

Too many to list; Have a look at defaults/main.yml
See also the minimal example listed below.
See also the test folder


Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: freebsd-test1
      user: nonpriv_user
      become: yes
      become_user: root
      vars:
        openvpn_default_server_ip: "10.2.1.0 255.255.255.0"
        openvpn_port: 443
        openvpn_proto: tcp
        openvpn_clients:
        - cn: user1
          validity: 3650
        - cn: user2-password
          validity: 365
          password: somesecurepassword
          ip: 10.2.1.55
        openvpn_client_push_routes:
        - "10.2.0.0 255.255.255.0"
      roles:
      - coffeesprout.openvpn

Certificate Revocation
----------------------

To revoke client certificates:

    - hosts: openvpn_servers
      become: yes
      vars:
        openvpn_revoke_clients:
        - cn: user1
          reason: key_compromise
        - cn: user2
          reason: affiliation_changed
      roles:
      - coffeesprout.openvpn
      tags:
      - revoke

Run with: `ansible-playbook playbook.yml --tags revoke`

License
-------

BSD

Author Information
------------------

