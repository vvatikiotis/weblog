---
author: Vassilis Vatikiotis
title: Manage LDAP entries with ruby
tags:
  - linux
  - ldap
---

# Manage LDAP entries with ruby

Say you want to add an attribute to some entries. Here's a sample:

```ruby
require 'net/ldap' # http://rdoc.info/gems/net-ldap/frames

base = 'ou=people,dc=my,dc=domain,dc=com'
filter = '(objectCLass=posixAccount)'

ldap = Net::LDAP.new
ldap.host = 'localhost'
ldap.port = 389
ldap.auth 'cn=admin,dc=my,dc=domain,dc=com', "passsword"

if ldap.bind
  ldap.search( :base => base, :filter => filter ) { |entry|
    ldap.add_attribute entry.dn, 'title', "Mrs"
    ldap.modify :dn => entry.dn,
                : operations => [[:replace, 'title', 'None']]
  }
end
```
