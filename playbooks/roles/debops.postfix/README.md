## debops.postfix

[![Platforms](http://img.shields.io/badge/platforms-debian%20|%20ubuntu-lightgrey.svg)](#)

This role installs and manages [Postfix](http://postfix.org/), an SMTP
server.

`debops.postfix` role is designed to manage Postfix on different hosts in
a cluster, with different "capabilities". At the moment role can configure
Postfix to act as:

* a null client: Postfix sends all mail to another system specified
  either via DNS MX records or an Ansible variable, no local mail is enabled
  (this is the default configuration);
* a local SMTP server: local mail is delivered to local user accounts;
* a network SMTP server: network access is enabled separately from other
  capabilities, to avoid exposing misconfigured SMTP server by mistake and
  becoming an open relay;
* an incoming MX gateway: Postfix will listen on the port 25 (default SMTP
  port) and process connections using `postscreen` daemon with automatic
  greylisting and optional RBL checking;

More "capabilities" like user authentication, support for virtual mail,
spam/virus filtering and others will be implemented in the future.

This role can also be used as a dependency of other roles which then can
enable more features of the Postfix SMTP server for their own use. For
example, `debops.mailman` role enables mail forwarding to the configured
mailing lists, and `debops.smstools` role uses Postfix as mail-SMS gateway.

### Installation

To install `debops.postfix` using Ansible Galaxy, run:

    ansible-galaxy install debops.postfix

### Role dependencies

- `debops.pki`
- `debops.ferm`



### Role variables

List of default variables available in the inventory:

    ---
    
    # Active Postfix capabilities (see README.md). By default Postfix is configured
    # with local mail disabled, all mail is sent to local domain MX server
    postfix: [ 'null' ]
    
    
    # Configuration options for Postfix. Many options are configured automatically
    # using templates, here you can (mostly) add your own entries to Postfix lists
    # (look in Postfix manual for details), they will by added or replaced in
    # templates.
    
    # Mail host name configured in /etc/mailname
    postfix_mailname: '{{ ansible_fqdn }}'
    
    # How long to wait before notifying users about delivery problems
    postfix_delay_warning_time: '4h'
    
    # Address of mail host this host should relay all mail to instead of delivering
    # it directly. (Automatic configuration)
    postfix_relayhost: False
    
    # List of relay domains this host accepts
    postfix_relay_domains: []
    
    # On what interfaces postfix should listen to by default (not a list). (Automatic configuration)
    postfix_inet_interfaces: False
    
    # List of local domains accepted by postfix. (Automatic configuration)
    postfix_mydestination: []
    
    # List of networks postfix accepts by default. (localhost is always enabled)
    postfix_mynetworks: []
    
    # List of postfix transport maps. (Automatic configuration)
    postfix_transport_maps: []
    
    # List of postfix virtual alias maps. (Automatic configuration)
    postfix_virtual_alias_maps: []
    
    # Message size limit in megabytes
    postfix_message_size_limit: 50
    
    
    # TLS certificate configuration (see 'pki' role). By default postfix relies on
    # self-signed certificates, but signed or wildcard certificates can also be
    # enabled.
    postfix_pki: '/srv/pki'
    postfix_pki_type: 'selfsigned'
    postfix_pki_wildcard: '{{ ansible_domain }}'
    postfix_pki_name: '{{ ansible_fqdn }}'
    postfix_pki_cert: False
    postfix_pki_key: False
    
    
    # Firewall configuration. Set these variables to True to enable access for all
    # Internet hosts, or provide lists of allowed IP addresses or address ranges
    # for ports smtp (25), submission (587) or smtps (465). Set to False to
    # deny access from remote hosts.
    postfix_allow_smtp: True
    postfix_allow_submission: True
    postfix_allow_smtps: True
    
    
    # Postscreen blacklists
    postfix_postscreen_dnsbl_sites:
    
      # Spamhaus ZEN: http://www.spamhaus.org/zen/
      # Might require registration
      - 'zen.spamhaus.org*3'
    
      # Barracuda Reputation Block List: http://barracudacentral.org/rbl
      # Requires registration
      #- 'b.barracudacentral.org*2'
    
      # Spam Eating Monkey: http://spameatingmonkey.com/lists.html
      # Might require registration
      - 'bl.spameatingmonkey.net*2'
      - 'backscatter.spameatingmonkey.net*2'
    
      # SpamCop Blocking List: http://www.spamcop.net/bl.shtml
      - 'bl.spamcop.net'
    
      # Passive Spam Block List: http://psbl.org/
      - 'psbl.surriel.com'
    
      # mailspike: http://mailspike.net/usage.html
      # Might require contact
      - 'bl.mailspike.net'
    
    
    # Postscreen whitelists
    postfix_postscreen_dnswl_sites:
    
      # SpamHaus Whitelist: http://www.spamhauswhitelist.com/en/usage.html
      # Might require registration
      - 'swl.spamhaus.org*-4'
    
      # DNS Whitelist: http://dnswl.org/tech
      # Might require registration
      - 'list.dnswl.org=127.[0..255].[0..255].0*-2'
      - 'list.dnswl.org=127.[0..255].[0..255].1*-3'
      - 'list.dnswl.org=127.[0..255].[0..255].[2..255]*-4'
    
    
    # List of user-supplied smtpd restrictions, they will replace restrictions
    # automatically created by templates.
    postfix_smtpd_client_restrictions: []
    postfix_smtpd_helo_restrictions: []
    postfix_smtpd_sender_restrictions: []
    postfix_smtpd_relay_restrictions: []
    postfix_smtpd_recipient_restrictions: []
    postfix_smtpd_data_restrictions: []
    
    
    # List of default recipients for local aliases which have no recipients
    # specified, by default current $USER managing Ansible
    postfix_default_local_alias_recipients: ['{{ lookup("env","USER") }}']
    
    # Hash of local aliases which will be merged with default aliases in
    # vars/main.yml. Commented out example below.
    postfix_local_aliases:
      #'alias': [ 'account1', 'account2' ]
      #'other': [ 'user@email', '"|/dir/command"' ]
      #'blackhole': [ '/dev/null' ]
      #'default_recipients':
    
    
    # Custom configuration added at the end of /etc/postfix/main.cf (use text block)
    postfix_local_maincf: False
    
    # Custom configuration added at the end of /etc/postfix/master.cf (use text block)
    postfix_local_mastercf: False
    
    
    # This variable can be used in postfix dependency role definition to configure
    # additional lists used in Postfix main.cf configuration file. This variable
    # will be saved in Ansible facts and updated when necessary
    postfix_dependent_lists: {}
      # Examples:
    
      # Include these lists in transport_maps option
      #transport_maps: ['hash:/etc/postfix/transport']
    
      # Include this alias map if Postfix has 'local' capability
      #alias_maps:
      #  - capability: 'local'
      #    list: [ 'hash:/etc/aliases' ]
    
      # Include this virtual alias map if Postfix does not have 'local' capability
      #virtual_alias_maps:
      #  - no_capability: 'local'
      #    list: [ 'hash:/etc/postfix/virtual_alias_maps' ]
    
    # Here you can specify Postfix configuration options which should be enabled in
    # main.cf using postfix dependency role definition. Configuration will be saved
    # in Ansible facts and updated when necessary
    postfix_dependent_maincf: []
      # Examples:
    
      # Set this option in main.cf
      #- param: 'local_destination_recipient_limit'
      #  value: '1'
    
      # Enable this option only if 'mx' is in Postfix capabilities
      #- param: 'defer_transports'
      #  value: 'smtp'
      #  capability: 'mx'
    
      # Enable this option only if 'local' is not in Postfix capabilities
      #- param: 'relayhost'
      #  value: 'mx.example.org'
      #  no_capability: 'local'
    
      # If no value is specified, check if a list of the same name as param exists
      # in postfix_dependent_lists and enable it
      #- param: 'virtual_alias_maps'
    
    # This list can be used to configure services in Postfix master.cf using
    # postfix dependency variables. Configured services will be saved in Ansible
    # facts and updated when necessary
    postfix_dependent_mastercf: []
      # Examples:
    
      # Minimal service using 'pipe' command
      #- service: 'mydaemon'
      #  type: 'unix'
      #  command: 'pipe'
      #  options: |
      #    flagsd=FR user=mydaemon:mydaemon
      #    argv=/usr/local/bin/mydaemon.sh ${nexthop} ${user}
    
      # Optional parameters from master.cf:
      # private, unpriv, chroot, wakeup, maxproc
    
      # You can also specify 'capability' or 'no_capability' to define when
      # a particular service should be configured
    
    
    # At what hour DH parameters will be regenerated by a script run by cron
    postfix_cron_dhparams_hour: '3'
    
    # List of clients and networks which will have access to XCLIENT protocol
    # extension when 'test' postfix capability is enabled.
    postfix_smtpd_authorized_xclient_hosts: ['127.0.0.1/32']




### Detailed usage guide

List of Postfix capabilities in `postfix` variable - what Postfix can and
should do on a host. Set this to `False` and disable Postfix support, set it
to `[]` and have Ansible not do anything with Postfix (unsupported). Not all
combinations of these capabilities will work correctly (role is still in
beta stage).

- `null`: Postfix has no local delivery, all mail is sent to a MX for current
  domain. Configuration similar to that presented here:
  http://www.postfix.org/STANDARD_CONFIGURATION_README.html#null_client
  Default. You should remove this capability and replace it with others
  presented below.

- `local`: local delivery is enabled on current host.

- `network`: enables access to Postfix-related ports (`25`, `587`, `465`) in
  firewall, required for incoming mail to be acceped by Postfix.

- `mx`: enables support for incoming mail on port `25`, designed for hosts set up
  as MX. Automatically enables `postscreen` (without `dnsbl`/`dnswl` support),
  anti-spam restrictions.

- `submission`: enables authorized mail submission on ports `25` and `587` (user
  authentication is currently not supported and needs to be configured
  separately).

- `deprecated`: designed to enable obsolete functions of mail system,
  currently enables authorized mail submission on port `465` (when
  `submission` is also present in the list of capabilities).

- `postscreen`: allows to enable postscreen support on port `25` independently of
  `mx` capability.

- `dnsbl`: enables support for DNS blacklists in postscreen, automatically
  enables whitelists.

- `dnswl`: enables support for DNS whitelists in postscreen, without blacklists.

- `test`: enables "soft_bounce" option and XCLIENT protocol extension for
  localhost (useful in mail system testing).

- `defer`: planned feature to defer mail delivery.

- `auth`: planned feature to enable user authentication.


### Authors and license

`debops.postfix` role was written by:

- Maciej Delmanowski - [e-mail](mailto:drybjed@gmail.com) | [Twitter](https://twitter.com/drybjed) | [GitHub](https://github.com/drybjed)


License: [GNU General Public License v3](https://tldrlegal.com/license/gnu-general-public-license-v3-(gpl-3))


***

This role is part of the [DebOps](http://debops.org/) project. README generated by [ansigenome](https://github.com/nickjj/ansigenome/).

