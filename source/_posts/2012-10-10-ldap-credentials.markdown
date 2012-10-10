---
layout: post
title: "Credentials in LDAP URLs when Anonymous Search is Disabled"
date: 2012-10-10 16:55
comments: true
categories: 
---
Cumin authenticates logins against LDAP using a two step process:

+ Search for LDAP entries based on the login name and the search filter in the LDAP URL
+ Attempt a simple bind operation using the distinguished names (dn) in returned entries and the password

If anonymous access is disabled by the LDAP server, the first step will fail.  In this case, Cumin needs to bind to the server with credentials before doing the search for user records.  Happily, the URL syntax allows for credentials in the URL and Cumin will use them if provided to bind before the search.

### Extra Attributes
The credentials for an initial bind can be set using LDAP extensions supported by the ldapurl module.  There are two additional attributes:

+ bindname (a formal dn corresponding to an LDAP entry)
+ X-BINDPW (a password that will work for that entry)

These attributes follow the filter specification and are separated by a comma.  Any comma (,) characters present in the bindname value must be html-escaped because the URL parser will see them as attribute separators.  Futhermore, the % character in the escape code for a comma must be escaped itself to prevent Python from seeing it as a string substituion sequence.  Simple, right?

### An Example

Let's assume the following:  

+ The LDAP server is at example.com:389
+ The search dn is ou=People,dc=example,dc=com
+ The search scope is sub
+ The filter compares uid to username (this is actually the default)
+ The dn for an initial bind is uid=joeuser,ou=People,dc=example,dc=com
+ The password for the initial bind is joepassword

The LDAP URL in the Cumin configuration file would like like this:

    auth: ldap://example.com:389/ou=People,dc=example,dc=com??sub?(&(uid=%%s))?bindname=uid=joeuser%%2cou=People%%2cdc=example%%2cdc=com,X-BINDPW=joespassword  

Note in particular the %%2c substrings in the URL.  These are the commas in the bindname value.  In the interest of full disclosure, let me  be the first to say "yuck".  But it works!

### Caution!
LDAP credentials specified as part of a URL in the cumin configuration file will be visible to anyone who has access to the configuration file.  When Cumin is installed as a package the configuration file will be located at `/etc/cumin/cumin.conf` and will only be readable by the `cumin` user.  This should be adequate to protect the credentials as long as the permissions on that file are not changed.

However, if Cumin is run from sources in a development instance, alternate configuration files are possible.  Care should be taken to protect those configuration files if they contain credentials.

[Wiki]:http://fedorahosted.org/grid/wiki/Cumin
The Cumin project wiki can be found [here][Wiki]
