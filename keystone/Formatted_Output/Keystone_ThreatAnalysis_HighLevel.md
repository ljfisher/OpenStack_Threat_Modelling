
Keystone Threat Modeling - High Level
=========================================
### Table of contents

[System Overview](#system)

[Implementation Overview](#implementation)

[System Assumptions](#assumption)

[Data Flow Diagrams](#dfd)

[Attacker and Actors](#attackers)

[Entry Points](#entry)

[Assets](#asset)

[Components](#components)

[Threats](#threats)


<a name="system"/>
###System Overview
####Application version
   Keystone Havana Stable Release
   
####Application Description
   Keystone provides identity and access management for users of OpenStack Cloud.  For this, it provides a shim and most of the underlying components are pluggable. It provides authentication of users, can contain a repository of users (identities) and roles. It also performs account and entitlement management of users.  The entitlements are managed by assigning roles for users and enforcing access control on the target service (object).
   
   -	Username/user id identifies the user. A user is authenticated by a configured authentication mechanism. Before authentication, account needs to be provisioned. Keystone also performs account provisioning e.g., users, tenant, domain, role, and their assignments. External authentication mechanisms e.g., Apache HTTP authentication can be integrated with keystone.
-	Authorization (entitlement) is performed for each request based on assigned role within the token for a user and policy file defined in the system. The entitlement decision is decentralized i.e., made by the respective service.
-	Each successful and unsuccessful request is logged.
-	User and system data is protected by access control and sensitive data (only password) is protected by one way hashing.
-	Keystone provides catalog information for other services.

   
####Additional Info
  

<a name="implementation"/>
###Implementation Overview
####Major Components
   Pipe line middlewares:  url_nomalizer, token_auth, admin_token_auth, xml_body, Json_body, ec2_extension,  S3_extension,  crud_extension, admin_service, public_service (check the middleware from paste.ini for specific Pipe).  Currently this document focuses on [pipeline:api_v3], the keystone v3 API pipeline. The document also include some parts of [pipeline:public_api] – keystone v2 API pipeline.

Services/internal components: Identity, Assignment, Catalog, policy, Token, Token Provider, Trust

Drivers/persistence storage:  SQL, FILE, KVA, LDAP


####Dependent components
  Webservers:  WSGI (or any web server on top WSGI modules can run) 

Operating system and libraries:  oslo.config, dependencies. 

Crypto:  OpenSSL library, hashlib, passlib, UUID generator (Crypto library versions are considered)
Related info covered in:  
https://wiki.openstack.org/wiki/Security/Juno/Keystone#Notable_changes_since_Icehouse
####Description

-	Keystone consists of a combination of services: identity, assignment, catalog, policy, token and so on. Each of the service has specific drivers, which can be plugged using configuration profile.
-	Keystone performs access control: authentication and authorization (entitlement) of users. The authentication process can be performed using three mechanisms: username/password, token, and External authentication (including external authentication middleware or an external authentication mechanism which set REMOTE_AUTH variable). A successful authentication issues a token to the requester.
-	The authorization process consists of assignment of roles, policy definition file specific for specific OpenStack service, entitlement decision engine and enforcement points. 
-	Keystone provides identity provisioning service e.g., users, tenants, groups, roles, domains 
-   As an addition, Keystone provides catalog services   (description of     services and endpoints) for each user.

<a name="assumption"/>
###System Assumptions (External Dependencies)
####Application

- Keystone running in WSGI server which is accessible via public port 5000 and admin port 35357
- The document will cover both V2.0 (limited only for Token service) and V3.0 API.
- Keystone.conf, unless stated otherwise, the default values of the configuration file is used (keystone with devstack setting)
- PKI token (default token provider)
- SSL endpoint (Keystone server endpoint. We assumed keystone server to client transport channel is SSL protected.     OpenStack Security Guide and Keystone doc define steps for SSL endpoints)
- The communication channel between client and keystone service endpoint is SSL protected (from assumption 4).

####Persistence 
- The database server is MySQL and related driver is SQLAlchemy.
- Default config DB from keystone.conf and mysql.conf

####Platform
- Operating systems, webserver, and physical machines are hardened
- The system admin follows security best practice
- Keystone server is running in Ubuntu 12.04
- Dependencies (keystone dependent libraries) in requirements.txt  are trustworthy

###Security Objective

 -  An IdAM must provide
-	Identification (authentication) and entitlements of users
-	Entitlement decision (authorization) for user-initiated request
-	Non-repudiable auditing for each request
-	Protection of system and user data / state information.
-   In addition, for keystone case: 
    -    Integrity of catalog information 

<a name="dfd"/>
###Data Flow Diagrams 
####Keystone Components
![enter image description here][1]


<a name="attackers"/>
###Attackers and Actors


<a name="entry"/>
###Entry Points


####Name
  data


----------
<a name="asset"/>
###Assets
Full assets list is documented in url



----------
<a name="components"/>
###Components



----------
<a name="threats"/>
###Threats
####ComponentName-001
Threats:
> 

Threat Agent:
> 

Attack Vectors:
> 

Security Weakness:
> 

Counter Measures:
> 

Extra:
>  Probability:

>   Impact:

>   Related Info:

>   Comments:
     Link to Bug/mailing list or Tracking 

####ComponentName-002
Threats:
> 

Threat Agent:
> 

Attack Vectors:
> 

Security Weakness:
> 

Counter Measures:
> 

Extra:
> 


  [1]: images/DFD_Attacker_Keystone_level_1.png