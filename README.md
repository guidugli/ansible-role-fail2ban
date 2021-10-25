Ansible Role: fail2ban
=========

An Ansible Role that install and configure fail2ban on RHEL/CentOS, Fedora and Debian/Ubuntu.


Requirements
------------

Ansible server needs to have netaddr package installed. Install it using `pip install netaddr` or install corresponding package for your distribution.

Role Variables
--------------

**Available variables are listed below, along with default values (see defaults/main.yml):**

    #f2b_bantime_increment: true

"bantime.increment" allows to use database for searching of previously banned ip's to increase a default ban time using special formula, default it is banTime * 1, 2, 4, 8, 16, 32...

    #f2b_bantime_rndtime: 120

"bantime.rndtime" is the max number of seconds using for mixing with random time to prevent "clever" botnets calculate exact time IP can be unbanned again:

    #f2b_bantime_maxtime: 3660

"bantime.maxtime" is the max number of seconds using the ban time can reach (don't grows further)

    #f2b_bantime_factor: 1

"bantime.factor" is a coefficient to calculate exponent growing of the formula or common multiplier, default value of factor is 1 and with default value of formula, the ban time grows by 1, 2, 4, 8, 16 ...

    #f2b_bantime_formula: 'ban.Time * (1<<(ban.Count if ban.Count<20 else 20)) * banFactor'

"bantime.formula" used by default to calculate next value of ban time, default value bellow, the same ban time growing will be reached by multipliers 1, 2, 4, 8, 16, 32...

    #f2b_bantime_multipliers: '1 5 30 60 300 720 1440 2880'

"bantime.multipliers" used to calculate next value of ban time instead of formula, coresponding previously ban count and given "bantime.factor" (for multipliers default is 1); 
following example grows ban time by 1, 2, 4, 8, 16 ... and if last ban count greater as multipliers count, always used last multiplier (64 in example), for factor '1' and original ban time 600 - 10.6 hours

> f2b_bantime_multipliers: '1 2 4 8 16 32 64' 

    #f2b_bantime_overalljails: false

"bantime.overalljails" (if true) specifies the search of IP in the database will be executed cross over all jails, if false (dafault), only current jail of the ban IP will be searched

    #f2b_ignoreself: true

"ignoreself" specifies whether the local resp. own IP addresses should be ignored (default is true). Fail2ban will not ban a host which matches such addresses.

    #f2b_ignoreip: 127.0.0.1/8 ::1

"ignoreip" can be a list of IP addresses, CIDR masks or DNS hosts. Fail2ban will not ban a host which matches an address in this list. Several addresses can be defined using space (and/or comma) separator.

    #f2b_bantime: 30m

"bantime" is the number of seconds that a host is banned.

    #f2b_findtime: 30m

A host is banned if it has generated "maxretry" during the last "findtime" seconds.

    #f2b_maxretry: 5

"maxretry" is the number of failures before a host get banned.

    #f2b_maxmatches: '%(maxretry)s'

"maxmatches" is the number of matches stored in ticket (resolvable via tag <matches> in actions).

    #f2b_usedns: warn

"usedns" specifies if jails should trust hostnames in logs,   warn when DNS lookups are performed, or ignore all hostnames in logs
- yes:   if a hostname is encountered, a DNS lookup will be performed.
- warn:  if a hostname is encountered, a DNS lookup will be performed, but it will be logged as a warning.
- no:    if a hostname is encountered, will not be used for banning, but it will be logged as info.
- raw:   use raw value (no hostname), allow use it for no-host filters/actions (example user)

.

    #f2b_destemail: root@localhost

Destination email address used solely for the interpolations in `jail.{conf,local,d/*}` configuration files.

    #f2b_sender: root@<fq-hostname>

Sender email address used solely for some actions

    #f2b_mta: sendmail

E-mail action. Since 0.8.1 Fail2Ban uses sendmail MTA for the mailing. Change mta configuration parameter to mail if you want to revert to conventional 'mail'.

    #f2b_jails:
    #  - name: sshd
    #    settings:
    #      - { key: enabled, value: 'true' }
    #      - { key: mode, value: normal }
    #      - { key: port, value: 12345 }
    #  - name: apache-auth
    #    settings:
    #      - { key: enabled, value: 'true' }

Specify the name of the jail to config and the settings. Careful with settings as they will be added as is (check fail2ban documentation if the setting is correct).

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    fail2ban_cfg_dir: /etc/fail2ban

Fail2ban configuration directory

    fail2ban_packages:

Packages to install fail2ban

Dependencies
------------

No dependencies.

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: guidugli.fail2ban }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
