# Centos Firewall
## Services 

### Adding a Service to specific Zones

The easiest method is to add the services or ports you need to the zones you are using. 
Again, you can get a list of the available services with the --get-services option:

```
firewall-cmd --get-services
```

```output
RH-Satellite-6 amanda-client amanda-k5-client bacula bacula-client ceph ceph-mon dhcp dhcpv6 dhcpv6-client dns docker-registry dropbox-lansync freeipa-ldap freeipa-ldaps freeipa-replication ftp high-availability http https imap imaps ipp ipp-client ipsec iscsi-target kadmin kerberos kpasswd ldap ldaps libvirt libvirt-tls mdns mosh mountd ms-wbt mysql nfs ntp openvpn pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy proxy-dhcp ptp pulseaudio puppetmaster radius rpc-bind rsyncd samba samba-client sane smtp smtps snmp snmptrap squid ssh synergy syslog syslog-tls telnet tftp tftp-client tinc tor-socks transmission-client vdsm vnc-server wbem-https xmpp-bosh xmpp-client xmpp-local xmpp-server
```

> You can get more details about each of these services by looking at their associated .xml file within the `/usr/lib/firewalld/services` directory. For instance, the SSH service is defined like this:

For instance, if we are running a web server serving conventional HTTP traffic, we can allow this traffic for interfaces in our "public" zone for this session by typing:

```
sudo firewall-cmd --zone=public --add-service=http
sudo firewall-cmd --reload
```
