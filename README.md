# Ansible Role: Security (Basics)

[![CI](https://github.com/ispanos/ansible-role-security/workflows/CI/badge.svg?event=push)](https://github.com/ispanos/ansible-role-security/actions?query=workflow%3ACI)

**First, a major, MAJOR caveat**: the security of your servers is YOUR responsibility. If you think simply including this role and adding a firewall makes a server secure, then you're mistaken. Read up on Linux, network, and application security, and know that no matter how much you know, you can always make every part of your stack more secure.

That being said, this role performs some basic security configuration on RedHat and Debian-based linux systems. It attempts to:

  - Install software to monitor bad SSH access (fail2ban)
  - Configure SSH to be more secure (disabling root login, requiring key-based authentication, and allowing a custom SSH port to be set)
  - Set up automatic updates (if configured to do so)
  - Create new administrators and adds their public keys. All fields must be filled if you decide to create a new user this way.

There are a few other things you may or may not want to do (which are not included in this role) to make sure your servers are more secure, like:

  - Use logwatch or a centralized logging server to analyze and monitor log files
  - Securely configure user accounts and SSH keys (this role assumes you're not using password authentication or logging in as root)
  - Have a well-configured firewall (check out the `geerlingguy.firewall` role on Ansible Galaxy for a flexible example)

Again: Your servers' security is *your* responsibility.

## Requirements

For obvious reasons, `sudo` must be installed if you want to manage the sudoers file with this role.

On RedHat/CentOS systems, make sure you have the EPEL repository installed (you can include the `geerlingguy.repo-epel` role to get it installed).

No special requirements for Debian/Ubuntu systems.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    security_ssh_port: 22

The port through which you'd like SSH to be accessible. The default is port 22, but if you're operating a server on the open internet, and have no firewall blocking access to port 22, you'll quickly find that thousands of login attempts per day are not uncommon. You can change the port to a nonstandard port (e.g. 2849) if you want to avoid these thousands of automated penetration attempts.

    security_ssh_password_authentication: "no"
    security_ssh_permit_root_login: "no"
    security_ssh_usedns: "no"
    security_ssh_permit_empty_password: "no"
    security_ssh_challenge_response_auth: "no"
    security_ssh_gss_api_authentication: "no"
    security_ssh_x11_forwarding: "no"

Security settings for SSH authentication. It's best to leave these set to `"no"`, but there are times (especially during initial server configuration or when you don't have key-based authentication in place) when one or all may be safely set to `'yes'`. **NOTE: It is _very_ important that you quote the 'yes' or 'no' values. Failure to do so may lock you out of your server.**

[//]: # (`security_ssh_pub_key_authentication: "yes"` is the default.)
[//]: # (`security_ssh_use_pam: "no"` Not sure if I should disable it.)

    security_sshd_state: started

The state of the SSH daemon. Typically this should remain `started`.

    security_ssh_restart_handler_state: restarted

The state of the `restart ssh` handler. Typically this should remain `restarted`.

---

    security_machine_admins:
      - username: admin
        password: '$6$ZNjKXVPieEdp2$xodvDHyPSY0pSbaPWpDwvuD59Zuwco6ZKHWwalEc34WiJjgYH8BW9/GchU1OOI7C8RUY3NHEtAW0A/5.bnNPE0'
        ssh_pub_key_url: https://raw.githubusercontent.com/hashicorp/vagrant/master/keys/vagrant.pub
        shell: /bin/bash

Create server administrators.
Default username is "admin, password is "admin", created with `mkpasswd --method=sha-512`
`ssh_pub_key_url` is Vagrant's default [insecure public key](https://github.com/hashicorp/vagrant/tree/main/keys)

    security_sudoers_passwordless: []
    security_sudoers_passworded: []

A list of users who should be added to the sudoers file so they can run any command as root (via `sudo`) either without a password or requiring a password for each command, respectively.

---

    security_autoupdate_enabled: true

Whether to install/enable `yum-cron` (RedHat-based systems) or `unattended-upgrades` (Debian-based systems). System restarts will not happen automatically in any case, and automatic upgrades are no excuse for sloppy patch and package management, but automatic updates can be helpful as yet another security measure.

    security_autoupdate_blacklist: []

(Debian/Ubuntu only) A listing of packages that should not be automatically updated.

    security_autoupdate_reboot: false

(Debian/Ubuntu only) Whether to reboot when needed during unattended upgrades.

    security_autoupdate_reboot_time: "03:00"

(Debian/Ubuntu only) The time to trigger a reboot, when needed, if `security_autoupdate_reboot` is set to `true`. In 24h "hh:mm" clock format.

    security_autoupdate_mail_to: ""
    security_autoupdate_mail_on_error: true

(Debian/Ubuntu only) If `security_autoupdate_mail_to` is set to an non empty value, unattended upgrades will send an e-mail to that address when some error occurs. You may either set this to a full email: `ops@example.com` or to something like `root`, which will use `/etc/aliases` to route the message. If you set `security_autoupdate_mail_on_error` to `false` you'll get an email after every package install.

    security_fail2ban_enabled: true

Whether to install/enable `fail2ban`. You might not want to use fail2ban if you're already using some other service for login and intrusion detection (e.g. [ConfigServer](http://configserver.com/cp/csf.html)).

    security_fail2ban_custom_configuration_template: "jail.local.j2"

The name of the template file used to generate `fail2ban`'s configuration.

## Dependencies

None.

## Example Playbook

    - hosts: servers
      vars_files:
        - vars/main.yml
      roles:
        - ispanos.security

*Inside `vars/main.yml`*:

    security_machine_admins:
      - username: ispanos
        password: '$6$YmGrvyFWlS$F6TZeSuV3Opc11gUF4WYOSPAFFg0rmBpBJ0AXXVxsnPKVCm0guFjHmg/oUwib3/ZbPtkPWhrWcIDAayHoJrOi1'
        ssh_pub_key_url: https://github.com/ispanos.keys
        shell: /bin/bash

    security_sudoers_passworded:
      - ispanos

    security_sshd_state: restarted

## License

MIT (Expat) / BSD

## Author Information

This role was originally created in 2014 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
