# ansible-role-postfix-dovecot
An Ansible role that automates the installation and configuration of Postfix and Dovecot with MySQL authentication on Ubuntu. The MySQL schema is derived from the following
[Digital Ocean tutorial](https://www.digitalocean.com/community/tutorials/how-to-configure-a-mail-server-using-postfix-dovecot-mysql-and-spamassassin).
You can view the MySQL schema used in [schema.sql](schema.sql).

## Role Variables

#### Required Variables
* **dovecot_ssl_cert** - the path to the SSL certificate used by Dovecot. Note that if you need to provide a certificate chain,
it must be concatenated after the certificate in the same file.
* **dovecot_ssl_key** - the path to the SSL key used by Dovecot.
* **postfix_ssl_cert** - the path to the SSL certificate used by Postfix. This should include the intermediary CA as well if applicable.
* **postfix_ssl_key** - the path to the SSL key used by Postfix.
* **postfix_dovecot_mysql_password** - the password to the user that has permission to query the database on the SQL database server used for authentication.

#### Optional Variables
* **solr** - [true/false], default is false, enable Solr backend for indexing mail's content
* **sql** - [true/false], default is true, disable mysql backend
* **postfix_tls** - [true/false], default is false, enable smtps (smtp over ssl, port 465)
* **postfix_dovecot_mysql_host** - the FQDN or IP address to the MySQL server for authentication. This defaults to `127.0.0.1`.
* **postfix_mydomain** - Define mydomain in postfix configuration
* **postfix_dovecot_mysql_db_name** - the database name on the MySQL server used for authentication. This defaults to `servermail`.
* **postfix_dovecot_mysql_user** - the user that has permission to query the database on the MySQL server used for authentication. This defaults to `usermail`.
* **postfix_dovecot_mysql_password_scheme** - the password scheme used to encrypt passwords in the database. This defaults to `SHA512-CRYPT`.
* **postfix_default_domain** - the value to set the default domain used by Postfix, particularly when Postfix determines the sender's domain when sending bounce messages. This sets the contents of `/etc/mailname`.
* **postfix_inet_protocols** - the protocol that Postfix should listen on. To have only IPv4, set this value to `ipv4`. This defaults to `all`.
* **postfix_submission_smtpd_client_restrictions** - a list of client restrictions on the mail submission port (587). For more information visit the [Postfix documentation](http://www.postfix.org/postconf.5.html#smtpd_client_restrictions).
This defaults to `permit_sasl_authenticated` and `reject`.
* **postfix_smtpd_tls_auth_only** - whether to only allow SASL authentication over SSL/TLS. This defaults to `yes`.
* **postfix_smtpd_recipient_restrictions** - a list of restrictions of recipients of incoming email. For more information visit the [Postfix documentation](http://www.postfix.org/postconf.5.html#smtpd_recipient_restrictions).
This defaults to `permit_sasl_authenticated`, `permit_mynetworks`, and `reject_unauth_destination`.
* **postfix_smtpd_relay_restrictions** - a list of relay restrictions. For more information visit the [Postfix documentation](http://www.postfix.org/postconf.5.html#smtpd_relay_restrictions).
This defaults to `permit_mynetworks`, `permit_sasl_authenticated`, and `defer_unauth_destination`.
* **postfix_mynetworks** - a list of trusted SMTP clients. For more information visit the [Postfix documentation](http://www.postfix.org/postconf.5.html#mynetworks).
This defaults to `127.0.0.0/8`, `[::ffff:127.0.0.0]/104`, `[::1]/128`.
* **postfix_mydestination** - a list for the Postfix configuration value of `mydestination`. For information on visit the [Postfix documentation](http://www.postfix.org/postconf.5.html#mydestination).
This defaults to `localhost`.
* **postfix_alias_maps** - define another alias lookup for user
* **postfix_transport_maps** - define transport maps (routing table)
* **postfix_sender_maps** - define sender maps (to permit and control Impersonate)
* **postfix_mysql_alias_query** - the query used to find the destination of an alias when the source is supplied. This defaults to `SELECT destination FROM virtual_aliases WHERE source='%s';`.
* **postfix_mysql_domains_query** - the query used to determine if a domain is valid. This defaults to `SELECT 1 FROM virtual_domains WHERE name='%s';`.
* **postfix_mysql_users_query** - the query used to determine if an email address is valid. This defaults to `SELECT 1 FROM virtual_users WHERE email='%s';`.
* **postfix_mailbox_command** - Specifiy mailbox command
* **dovecot_mysql_password_query** - the query used to authenticate a user on the MySQL server used for authentication. This defaults to `SELECT email as user, password FROM virtual_users WHERE email='%u';`.
* **dovecot_protocols** - a list of protocols to be enabled. This defaults to `lmtp` and `imap`. To enable POP3, add `pop3` to this variable. (note: `apt install dovecot-pop3d` on the target to use pop3)  
* **dovecot_mail_privileged_group** - the group that owns the folder defined in `dovecot_mail_location`.
This gives Dovecot's mail process the ability to write in the folder. This defaults to `mail`.
* **dovecot_disable_plaintext_auth** - determines if authentication without SSL is enabled. This defaults to 'yes'.
* **dovecot_auth_mechanisms** - a list of authentication mechanisms allowed by Dovecot. This defaults to `plain` and `login`.
For more informationm read Dovecot's [Authentication Mechanisms](http://wiki2.dovecot.org/Authentication/Mechanisms) documentation.
* **dovecot_force_imaps** - determines whether or not to disable IMAP and force IMAPS. This defaults to `true`.
* **dovecot_force_pop3s** - determines whether or not to disable POP3 and force POP3S. This defaults to `true`.
Note that to also enable POP3S, you need to add pop3 to the `dovecot_protocols` list variable.
* **dovecot_ssl** - determines whether or not SSL is enforced across all protocols. This defaults to `required`.
For more information, read Dovecot's [SSL Configuration](http://wiki.dovecot.org/SSL/DovecotConfiguration) documentation.
* **dovecot_listen** - a list of IP or host addresses where Dovecot listens for connections. This defaults to `*` (all IPv4) and '::' (all IPv6).
* **dovecot_mail_plugins** - list of plugins to load. Plugins specific to IMPA, LDA, etc.
* **dovecot_user** - specifiy the user for the dovecot service
* **ldap** - specify ldap configuration to bind
  * **base** - LDAP base
  * **uris** ldap(s) servers to bind
  * **auth_bind** - Use authentication binding for verifying password's validity
  * **auth_bind_userdn** - specify the user bind
  * **pass_attrs** - LDAP Special fields which can be returned
  * **pass_filter** - Filter for password lookups
* **mail_location** - specify the Location for users' mailboxes, default value is : `/var/mail/vhosts/%d/%n`
* **dovecot_dsync** - Default is false, enable dsync synchro between dovecot cluster (need notify and replication dovecot_mail_plugins)
* **doveadm_password** - Password for replicator (Used only if dsync is enabled)
* **dovecot_default_vsz_limit** - Default VSZ (virtual memory size) limit for service processes (Default is `256M`)
* **dovecot_log_path** - Define log path of dovecot - defaut : syslog
* **dovecot_auth_verbose** - Log unsuccessful authentication attempts and the reasons why they failed
* **dovecot_INBOX_path** - Optional, Path of the INBOX (need if use dsync)(Exemple : `~/Maildir/.INBOX`)
* **dovecot_INDEX_path** - Optional : Path of dovecot index, by default in the mail directory
* **tomcat_loglevel** - Define tomcat log level when solr is enabled (level available : [ SEVERE , WARNING , INFO , CONFIG , FINE, FINER , FINEST ])

## Requirements

This role must be run with sudo/become or as root, otherwise the role will fail.

## Example Playbook

_requirements.yml_
```yaml
roles:
  - name: stackfocus.postfix-dovecot
    version: v1.1.0
```

_site.yml_
```yaml
- hosts: all
  become: yes
  gather_facts: true
  roles:
    - stackfocus.postfix-dovecot
  vars:
    postfix_dovecot_mysql_db_name: mailserver
    postfix_dovecot_mysql_user: mailuser
    postfix_dovecot_mysql_password: mailpass
    postfix_default_domain: example.com
    dovecot_protocols:
      - imap
      - pop3
      - lmtp
    dovecot_mail_privileged_group: vmail
    dovecot_ssl_cert: /etc/ssl/certs/dovecot.pem
    dovecot_ssl_key: /etc/ssl/private/dovecot.pem
    postfix_ssl_cert: /etc/ssl/certs/postfix.pem
    postfix_ssl_key: /etc/ssl/private/postfix.pem
    dovecot_mail_plugins:
    - name: fts
      value: solr
    - name: fts_solr
      value: url=http://localhost:8080/solr/ debug
    solr: true
```


```
$ ansible-galaxy install -r requirements.yml
$ ansible-playbook -i inventory site.yml --ask-become-pass
```

* To enable and control Impersonate:
  * Define one (or more) postfix_sender_maps table to map uid with mail permission
  * Add reject_sender_login_mismatch in postfix_smtpd_recipient_restrictions (or in other postfix_smtpd_*_restrictions
  * Can be tested via `postmap  -q mail_address database_type:file_path`
