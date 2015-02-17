### DESCRIPTION

I wrote this module in need of managing bunch of servers running dnsmasq. 

It features some advanced features like:

* Basic dnsmasq management (service, installation)
* Cross-OS support (Debian, Ubuntu, RHEL, FreeBSD)
* Loads of options in basic config (ie TFTP)
(If you need any additional option that does not supported in this module, just ping me)
* Support for DHCP configuration.
* Support for adding static DHCP records (MAC -> IP binding)
* Support for adding static DNS records (IP -> hostname binding)
* Support for DHCP options
* Support for dnsmasq tagging system
* And much more

### DEPENDENCIES

* puppet >= 2.6
* puppetlabs/concat >= 1.0.0
* puppetlabs/stdlib

### BUILD STATUS
[![Build Status](https://travis-ci.org/rlex/puppet-dnsmasq.svg?branch=master)](https://travis-ci.org/rlex/puppet-dnsmasq)

### Basic class

Will install dnsmasq to act as DNS and TFTP (if specified) server

Example basic class config. Please refer to table below to see all possible variables

```puppet
class { 'dnsmasq':
  interface         => 'lo',
  listen_address    => '192.168.39.1',
  no_dhcp_interface => '192.168.49.1',
  domain            => 'int.lan',
  port              => '53',
  expand_hosts      => true,
  enable_tftp       => true,
  tftp_root         => '/var/lib/tftpboot',
  dhcp_boot         => 'pxelinux.0',
  dns_forward_max   => 1500,
  domain_needed     => true,
  bogus_priv        => true,
  no_negcache       => true,
  no_hosts          => true,
  resolv_file       => '/etc/resolv.conf',
  cache_size        => 1000,
  restart           => true,
}
```

Please refer to [dnsmasq man page](http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html) to get exact syntax and options

Core variables:

Variable      | Type          | Default | Description
------------- | ------------- | ------------ | -----------
$auth_sec_servers | string | undef | sec servers
$auth_server  | string  | undef | desc
$auth_ttl     | string | undef | desc
$auth_zone    | string | undef | desc
$bogus_priv | boolean | true | Desc
$cache_size | boolean | 1000 | Desc
$config_hash | array | undef | Desc
$dhcp_boot | bool | true | Desc
$dhcp_leasefile | boolean | true | Desc
$dhcp_no_override | boolean | false | Desc
$domain | string | undef | Desc
$domain_needed | boolean | false | Desc
$dns_forward_max | string | undef | Desc
$enable_tftp | boolean | undef | Desc
$expand_hosts | bool | true | Desc
$interface | string | undef | Desc
$listen_address | string | undef | Desc
$local_ttl | string | undef | Desc
$max_ttl | string | undef | Desc
$max_cache_ttl | string | undef | Desc
$neg_ttl | string | undef | Desc
$do_dhcp_interface | string | undef | Desc
$no_hosts | boolean | false | Desc
$no_negcache | boolean | false | Desc
$no_resolv | boolean | false
$port | string | 53 | Desc
$read_ethers | boolean | false | Desc
$reload_resolvconf | boolean | true | Desc
$resolv_file | boolean | false | Desc
$restart | boolean | true | Restart on config change
$run_as_user | string | undef | force dnsmasq under specific user
$save_config_file | boolean | true | Backup original config file
$service_enable | boolean | true | Start dnsmasq at boot
$service_ensure | string | running | Ensure service state
$strict_order | boolean | true | Desc
$tftp_root | string | /var/lib/tftpboot | Only if $dhcp_boot = true


There is also optional variables to override system-provided paths and names:

Variable      | Type          | Default | Desc
------------- | ------------- | ------- | --------
$dnsmasq_confdir | string | undef | Desc
$dnsmasq_conffile | string | undef | Desc
$dnsmasq_hasstatus | string | undef | Desc
$dnsmasq_logdir | string | undef | Desc
$dnsmasq_package | string | undef | Desc
$dnsmasq_package_provider | string | undef | Desc
$dnsmasq_service | string | undef | Desc

### DHCP server configuration

Will add DHCP support to dnsmasq.
This can be used multiple times to setup multiple DHCP servers.
Parameter "paramset" is optional, this one makes use of tagging system in dnsmasq

```puppet
dnsmasq::dhcp { 'dhcp':
  paramset   => 'hadoop0' #optional
  dhcp_start => '192.168.1.100',
  dhcp_end   => '192.168.1.200',
  netmask    => '255.255.255.0',
  lease_time => '24h'
}
```

### Static DHCP record configuration

Will add static DHCP record to DHCP server with hostname.
Please be aware that example-host will also be used as DNS name.

```puppet
dnsmasq::dhcpstatic { 'example-host':
  mac => 'DE:AD:BE:EF:CA:FE',
  ip  => '192.168.1.10',
}
```

### Host record configuration

Will add static A, AAAA and PTR record

```puppet
dnsmasq::hostrecord { "example-host-dns,example-host-dns.int.lan":
  ip  => '192.168.1.20',
}
```

### A record configuration

Will add static A record, this record will always override upstream data

```puppet
dnsmasq::address { "example-host-dns.int.lan":
  ip  => '192.168.1.20',
}
```

### CNAME records 
Will add canonical name record.
Please note that dnsmasq cname is NOT regular cname and can be only for targets
which are names from DHCP leases or /etc/hosts, so it's more like alias for hostname

```puppet
dnsmasq::cname { "mail":
  hostname => "post"
}
```

### SRV records
Will add srv record which always overrides upstream data.
Priority argument is optional.

```puppet
dnsmasq::srv { "_ldap._tcp.example.com":
  hostname => "ldap-server.example.com",
  port     => "389",
  priority => "1",
}
```

### MX records
Will create MX (mail eXchange) record which always override upstream data

```puppet
dnsmasq::mx { "maildomain.com":
  hostname   => "mailserver.com",
  preference => "50",
}
```

### PTR records 
Allows you to create PTR records for rDNS and DNS-SD.

```puppet
dnsmasq::ptr { "_http._tcp.dns-sd-services":
  value => '"New Employee Page._http._tcp.dns-sd-services"'
}
```

### TXT records
Allows you to create TXT records

```puppet
dnsmasq::txt { "_http._tcp.example.com":
  value => "name=value,paper=A4"
}
```
(this actually should be done via array, will fix later)

### DHCP option configuration

Will add dhcp option. Can be used for all types of options, ie:

* numeric ( dnsmasq::dhcpoption { '53': ... }
* ipv4-option ( dnsmasq::dhcpoption { 'option:router': ... }
* ipv6-option ( dnsmasq::dhcpoption { 'option6:dns-server': ... }

Can be used multiple times.

```puppet
dnsmasq::dhcpoption { 'option:router':
  content => '192.168.1.1',
  paramtag => 'sometag', #optional
}
```

### DHCP booting (PXE)

Allows you to setup different PXE servers in different subnets.
paramtag is optional, you can use this to specify subnet for bootserver, 
using tag you previously specified in dnsmasq::dhcp  
Can be used multiple times.

```puppet
dnsmasq::dhcpboot { 'hadoop-pxe':
  paramtag   => 'hadoop0', #optional
  file       => 'pxelinux.0', 
  hostname   => 'newoffice', #optional
  bootserver => '192.168.39.1' #optional
}
```

### Per-subnet domain

Allows you to specify different domain for specific subnets.
Can be used multiple times.

```puppet
dnsmasq::domain { 'guests.company.lan':
  subnet => '192.168.196.0/24',
}
```

### DNS server
Configure the DNS server to query external DNS servers
```puppet
dnsmasq::dnsserver { 'dns':
  ip => '192.168.1.1',
}
```

Or, to query specific zone
```puppet
dnsmasq::dnsserver { 'forward-zone':
  domain => "dumb.domain.tld",
  ip => "192.168.39.1",
}
```

### DNS-RR records 
Allows dnsmasq to serve arbitrary records, for example:
```puppet
dnsmasq::dnsrr { 'example-sshfp':
    domain => 'example.com',
    type   => '44',
    rdata  => '2:1:123456789abcdef67890123456789abcdef67890'
}
```

###Running in Docker containers
When running in a Docker container, dnsmasq tries to drop root privileges. This causes the following error:
```
dnsmasq: setting capabilities failed: Operation not permitted
```

In this case you can use the run\_as\_user to provide the appropriate user to run as:
```puppet
class { 'dnsmasq':
  interface         => 'lo',
  listen_address    => '192.168.39.1',
  no_dhcp_interface => '192.168.49.1',
  ....
  run_as_user       => 'root',
}
```
