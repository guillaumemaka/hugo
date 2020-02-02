---
type: post
title: "OpenLDAP: Create a custom LDAP schema"
date: 2013-07-17T11:30:00-04:00
updated: 2015-01-31T10:55:00-0400
comments: true
share: true
tags: [Linux,OpenLDAP]
image:
  feature: /images/abstract-6.jpg
---

This post will walk you throught how to create a custom LDAP schema and object

<!--more-->

# **1. Install OpenLDAP and Utils**
Run this commend in a terminal

{{< codecaption lang="shell" >}}
apt-get install slapd migrationtools libpam-ldap libnss-ldap
{{< /codecaption >}}

During the installation **slapd** ask you some basic configuration like the root of your directory, the distinguished name of the slapd manager, his password, ldap version (choose v3), the database frontend (choose hdb), and enable LDAPv2 compatibility (choose No) .

Ex: Your domain name is supinfo.com

{{< codecaption lang="cfg" >}}
root-dn: dc=supinfo,dc=com
admin-dn: cn=<username>,dc=supinfo,dc=com
{{< /codecaption >}}

# **2. Configuration**

1. edit /etc/ldap/ldap.conf like this

{{< codecaption lang="cfg" title="/etc/ldao/ldap.conf" >}}

 #  
 # LDAP Defaults  

 #  


 # See ldap.conf(5) for details  
 # This file should be world readable but not world writable.  


 BASE    <your-base-dn>
 URI ldap://<ip_address_OR_FQDN_of_your_ldap_server>  
 BINDDN <admin-dn>  

 #SIZELIMIT  12  
 #TIMELIMIT  15  
 #DEREF      never  

{{< /codecaption >}}

2. Edit /etc/libnss-ldap.conf and find these line

{{< codecaption lang="cfg" title="/etc/libnss-ldap.conf" >}}
 # The distinguished name of the search base.
 base <your-base-dn>


 # Another way to specify your LDAP server is to provide an
 uri ldapi:///<ip_address_OR_FQDN_of_your_ldap_server>


 # The LDAP version to use (defaults to 3
 # if supported by client library)
 ldap_version 3


 # The distinguished name to bind to the server with
 # if the effective user ID is root. Password is
 # stored in /etc/libnss-ldap.secret (mode 600)
 # Use 'echo -n "mypassword" > /etc/libnss-ldap.secret' instead
 # of an editor to create the file.
 rootbinddn <admin-dn>


 # Hash password locally; required for University of
 # Michigan LDAP server, and works with Netscape
 # Directory Server if you're using the UNIX-Crypt
 # hash mechanism and not using the NT Synchronization
 # service.
 pam_password crypt
 nss_base_passwd ou=People,<your-base-dn>?one
 nss_base_shadow ou=People,<your-base-dn>?one
 nss_base_group      ou=Group,<your-base-dn>?one
 nss_base_hosts      ou=Computers,<your-base-dn>?one
{{< /codecaption >}}

# 3. Create a custom LDAP schema

## **The Goal**

Clients will query the Directory server to retrieve policy objects that applies to them. These objects should at least have a serial number and an URI to the GPO file on the file server. As there is no standard LDAP object class to do that, you’ll have to write a custom schema.

Create a groupPolicyDescriptor (inheriting from top) class with two string attributes:

 *  id (32 characters)
 *  uri (255 characters)

The id will be a UUID hexadecimal string that will be used to GPD’s from one another.

The uri field will be used by the client to get the file.

You don’t have to write any GPO deployment tool for this project Just use a plain LDIF file to put groupPolicyDescriptor’s in your OU’'s for test purposes. There is no need to write a dedicated UUID for this part: Just use a random one.

# **4. Schema definition**

Resource: [Documentation][1] "Schema Specification"

   [1]: http://www.openldap.org/doc/admin22/schema.html

## _OIDs_

Each schema element is identified by a globally unique Object Identifier (OID). OIDs are also used to identify other objects. They are commonly found in protocols described by ASN.1. In particular, they are heavily used by the Simple Network Management Protocol (SNMP). As OIDs are hierarchical, your organization can obtain one OID and branch it as needed. For example, if your organization were assigned OID 1.1, you could branch the tree as follows:

[See][2] "8.2.1 Object Identifier - Table 8.2 Example OID hierarchy"

   [2]: http://www.openldap.org/doc/admin22/schema.html

## _Object Class Specification_

{{< codecaption lang="cfg" >}}
ObjectClassDescription = "(" whsp
    numericoid whsp      ; ObjectClass identifier
    [ "NAME" qdescrs
    [ "DESC" qdstring ]
    [ "OBSOLETE" whsp ]
    [ "SUP" oids ]       ; Superior ObjectClasses
    [ ( "ABSTRACT" / "STRUCTURAL" / "AUXILIARY" ) whsp ]
                                ; default structural
    [ "MUST" oids ]      ; AttributeTypes
    [ "MAY" oids ]       ; AttributeTypes
    whsp ")"
{{< /codecaption >}}

> Where:
>
> **whsp** = white space
**numericoid** = Object IDentifier [See][3] "8.2.1. Object Identifiers"

   [3]: http://www.openldap.org/doc/admin22/schema.html

## _Attribute Types Specification_
{{< codecaption lang="cfg" >}}
AttributeTypeDescription = "(" whsp
            numericoid whsp              ; AttributeType identifier
          [ "NAME" qdescrs ]             ; name used in AttributeType
          [ "DESC" qdstring ]            ; description
          [ "OBSOLETE" whsp ]
          [ "SUP" woid ]                 ; derived from this other
                                         ; AttributeType
          [ "EQUALITY" woid              ; Matching Rule name
          [ "ORDERING" woid              ; Matching Rule name
          [ "SUBSTR" woid ]              ; Matching Rule name
          [ "SYNTAX" whsp noidlen whsp ] ; Syntax OID
          [ "SINGLE-VALUE" whsp ]        ; default multi-valued
          [ "COLLECTIVE" whsp ]          ; default not collective
          [ "NO-USER-MODIFICATION" whsp ]; default user modifiable
          [ "USAGE" whsp AttributeUsage ]; default userApplications
          whsp ")"

      AttributeUsage =
          "userApplications"     /
          "directoryOperation"   /
          "distributedOperation" / ; DSA-shared
          "dSAOperation"          ; DSA-specific, value depends on server

{{< /codecaption >}}
> Where:
>
> **whsp** = white space
**numericoid** = Object IDentifier [See][4] "8.2.1. Object Identifiers"
**noidlen** = oid{lengh}
**SYNTAX** = [See][4] "Attribute Type Specification - Table 8.3: Commonly Used Syntaxes"

   [4]: http://www.openldap.org/doc/admin22/schema.html

## _Example_

> Create a groupPolicyDescriptor (inheriting from top) class with two string attributes:
>
>   * id (32 characters)
>   * uri (255 characters)

### **1. Attributes Definition**

{{< codecaption lang="cfg" >}}
objectidentifier gpoSchema 1.3.6.1.4.1.X.Y
objectidentifier gpoAttrs gpoSchema:3
objectidentifier gpoOCs gpoSchema:4

attributetype ( gpoAttrs:1
      NAME 'id'
      DESC 'GPO Unique Identifier'
      EQUALITY caseIgnoreMatch
      SUBSTR caseIgnoreSubstringsMatch
      SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{32} )

attributetype ( gpoAttrs:2
      NAME 'uri'
      DESC 'GPO Unique Resource Identifier'
      EQUALITY caseIgnoreMatch
      SUBSTR caseIgnoreSubstringsMatch
      SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{255} )
 {{< /codecaption >}}

### **2. Object Definition**

{{< codecaption lang="cfg" >}}
objectClass ( gpoOCs:1
    NAME 'groupPolicyDescriptor'
    DESC 'Describe a Group Object Policy'
    SUP ( top ) AUXILIARY
    MUST ( id $ uri ) )
{{< /codecaption >}}

> Replace X and Y by arbitrary number

# **5. Install the schema**

Create the file /etc/ldap/schema/gpo.schema, with the following line

{{< codecaption lang="cfg" title="/etc/ldapschema/gpo.schema" >}}
objectidentifier gpoSchema 1.3.6.1.4.1.X.Y
objectidentifier gpoAttrs gpoSchema:3
objectidentifier gpoOCs gpoSchema:4

attributetype ( gpoAttrs:1
              NAME 'id'
          DESC 'GPO Unique Identifier'
          EQUALITY caseIgnoreMatch
          SUBSTR caseIgnoreSubstringsMatch
          SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{32} )
    attributetype ( gpoAttrs:2
              NAME 'uri'
          DESC 'GPO Unique Resource Identifier'
          EQUALITY caseIgnoreMatch
          SUBSTR caseIgnoreSubstringsMatch
          SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{255} )
objectClass ( gpoOCs:1
            NAME 'groupPolicyDescriptor'
        DESC 'Describe a Group Object Policy'
        SUP ( top ) AUXILIARY
        MUST ( id $ uri ) )
{{< /codecaption >}}

Create a directory /tmp/ldap_schema

{{< codecaption lang="shell" >}}
mkdir /tmp/ldap_config
{{< /codecaption >}}

Create a file ~/test.conf

{{< codecaption lang="cfg" >}}
include /etc/ldap/schema/core.schema
include /etc/ldap/schema/cosine.schema
include /etc/ldap/schema/nis.schema
include /etc/ldap/schema/inetorgperson.schema
include /etc/ldap/schema/gpo.schema
{{< /codecaption >}}

Execute
{{< codecaption lang="shell" >}}
slaptest -f ~/test.conf -F /tmp/ldap_config
{{< /codecaption >}}

This will create a new "cn=config" directory in /tmp/ldap_config_. If you examine its contents, you'll see:

{{< codecaption lang="shell" >}}
ls /tmp/ldap_config/cn\=config
cn=module{0}.ldif  cn=schema.ldif          olcDatabase={1}bdb.ldif
cn=schema      olcDatabase={0}config.ldif  olcDatabase={-1}frontend.ldif
{{< /codecaption >}}

Note the cn=schema directory. This directory will contain the converted files, so let's go there:

{{< codecaption lang="shell" >}}
ls /tmp/ldap_config/cn\=config/cn\=schema
cn={0}core.ldif    cn={2}nis.ldif        cn={4}gpo.ldif
cn={1}cosine.ldif  cn={3}inetorgperson.ldif
{{< /codecaption >}}

As you can see, there is now a gpo.ldif file, which is what has been converted from the Gpo schema file.

To finish, we need to copy the new file in the OpenLDAP schema directory and fix permissions, and restart the slapd daemon
{{< codecaption lang="shell" >}}
cp /tmp/ldap_config/cn\=config/cn\=schema/cn={4}gpo.ldif /etc/ldap/slapd.d/cn\=config/cn\=schema/  
chown openldap:openldap /etc/ldap/slapd.d/cn\=config/cn\=schema/cn={4}gpo.ldif
/etc/init.d/slapd restart
{{< /codecaption >}}

# **6. Implementing the schema**

Well we have our custom schema, so let use it. Assume we want a computer ***smith-computer*** launch a script at boot.  
Create a ldif file name it ***smith-computer.ldif***, and put these lines:

{{< codecaption lang="cfg" title="smith-computer.ldif" >}}
dn: cn=smith-computer,dc=supinfo,dc=local
objectClass: top
objectClass: device
objectClass: groupPolicyDescriptor
ou: Computers
cn: smith-computer
id: 00000000000002
uri: \\sysvol\scripts\hosts.sh
{{< /codecaption >}}

Run this command to add the new computer

{{< codecaption lang="shell"  title="shell" >}}
ldapadd -x -W -D "cn=admin,dc=supinfo,dc=local" -f smith-computer.ldif
{{< /codecaption >}}

> Replace the argument for the -D option by your admin dn  

This command will prompt you for the LDAP administrator password

{{< codecaption lang="shell"  title="shell" >}}
Enter LDAP Password:
adding new entry "cn=smith-computer,dc=supinfo,dc=local"
{{< /codecaption >}}

To verify that the new computer has been add, run this command

{{< codecaption lang="shell"  title="shell" >}}
ldapsearch -x "(cn=smith-computer)"
{{< /codecaption >}}

*shell result*

{{< codecaption lang="shell"  title="shell" >}}
# extended LDIF
#
# LDAPv3
# base <...> (default) with scope subtree
# filter: (cn=smith-computer)
# requesting: ALL
#

# smith-computer, supinfo.local
dn: cn=smith-computer,dc=supinfo,dc=local
objectClass: top
objectClass: device
objectClass: groupPolicyDescriptor
ou: Computers
cn: smith-computer
id: 00000000000002
uri: \\sysvol\scripts\hosts.sh

# search result
search: 2
result: 0 Success
{{< /codecaption >}}

That's it !

# **What's Next…**  

Program a Shell or C script to walktrough the LDAP Tree to find each object contain the groupPolicyDescriptor, extract the "uri" value and excute the script you found in the "uri".
