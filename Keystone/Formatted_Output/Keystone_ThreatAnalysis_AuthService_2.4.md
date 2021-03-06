Keystone Threat Modeling : Auth Controller V3
=========================================
### Table of contents

[System Overview](#system)

[Implementation Overview](#implementation)

[System Assumptions](#assumption)

[Data Flow Diagrams](#dfd)

[Entry Points](#entry)

[Assets](#asset)

[Threats](#threats)


<a name="system"/>
###System Overview
####Application version
Keystone Havana Stable Release
   
####Application Description
The V3 Auth Controller routes and glues the requests from service_v3 middleware to the appropriate internal services. It also performs filtering on incoming and outgoing requests.  Authenticated users are issued bearer token.

####Additional Info
  

<a name="implementation"/>
###Implementation Overview
####Major Components
Keystone/auth /controllers.py
   
####Dependent components
Keystone/auth/routers.py

Keystone/auth/plugins/{authentication_plugin}

TOKEN API, IDENTITY API, TOKEN PROVIDER API, TRUST API, POLICY
####Description
Each authentication requests is a POST request, must include user credential, authentication method, and optionally ‘scope’. The scope is either project or domain, not both. Scope is also identifiable from trust_id. The authentication token is returned in X-Subject-Token header

POST /auth/tokens authenticate_for_token

GET  /auth/tokens validate_token (validates token)

HEAD /auth/tokens check_token

DELETE /auth/tokens revoke_token (disables a token by modifying 'enable' flag)

 More info: https://github.com/openstack/identity-api/blob/master/v3/src/markdown/identity-api-v3.md

<a name="assumption"/>
###System Assumptions (External Dependencies)
 - The communication channel between the end user and Keystone is SSL protected.
 - Only V3.0 API is covered in this doc.

###Security Objective

1. Issuance of integrity protected Token
2. Validate the integrity of Token
3. Avalability of service
4. Authentication of user (desired property is mutual authentication)
5. Issuance of integrity protected token revocation list
6. Auditablity of requset/response
7. User data integrity check .. * username, tenant, domain, token_id length check .. * whitelist/black list check

<a name="dfd"/>
###Data Flow Diagrams 
####Top level
![enter image description here][1]
####Authenticate
![enter image description here][2]
####Validate Token
![enter image description here][3]
####Check Token
![enter image description here][4]
####Delete Token
![enter image description here][5]

(unique is hash algorithm dependent)
 
<a name="entry"/>
###Entry Points
####Name ID-01: Upstream router
#####Description
Request from upper pipeline.
#####Accessible To
8) Keystone Process user

10)System admin 

####Name ID-02: External Identity Provider
#####Description
User Identity request resolved by external Provider. 
Sets REMOTE_USER variable
#####Accessible To
8) Keystone Process user 

10)System admin

13)External Identity Provider User

####Name ID-03: Other Internal Services
#####Description
IDENTITY API, TOKEN PROVIDER API, TOKEN  API


----------
<a name="asset"/>
###Assets
Full assets list is documented in url
[Asset Library][6]

1) User

1.1) User Secret (password, Token)

8) Token

8.2) Scope (project, Domain, Trust)

26) Configuration (Expiry_time)

27) REMOTE_USER

35) Context 

36) Auth Mecahnsim (Auth Chaining)

----------
<a name="threats"/>
###Threats
####AuthController -001
Threats:
> Spoofing: Unauthorized access

Threat Agent:
> Internet Attacker - Unauthorized

Attack Vectors:
> REMOTE_USER asset context exploited

Security Weakness:
> Context generation weakness. No credible one found, but possible exploitation area. Need to check, we can pass the REMOTE_USER in query string. Can it escape somehow the query string parameter in context? Special checks: no one tries to flatten the context. Is REMOTE_USER is set by other sources  than authentication purpose? Not all external authentication mechanism do set REMOTE_USER variable , what to expect for such cases.

Counter Measures:
> Follow best practices

Extra:
>  Probability: Zero (current state)

>   Impact: High

>   Related Info:

>   Comments:
     1. http://lists.openstack.org/pipermail/openstack-dev/2013-October/017886.html
     2.http://www.freeipa.org/page/Environment_Variables#X.509_Authentication
     

####AuthController-002
Threats:
> Spoofing

Threat Agent:
> Internet Attacker-Unauthorized or Authorized,

Attack Vectors:
> Exploiting the weakness of supported authentication mechanism.

Security Weakness:
> V3 support multiple authentication mechanism. Support for both weak and strong authentication at the same time to get authentication token could resulted in a threat in which an attacker could 
use the weaker authentication mechanism to get a TOKEN. An attacker will always choose the lower grade authentication mechanism to attack the target.  For example, supporting password and one time password both at the same time. The attacker will choose the lower strength auth method to achieve his goal. Weakness could generate from supporting multiple authentication.


Counter Measures:
> Security best practice to decide, resources available based on authentication method strength. User (or a domain) can be binded to allowable authentication mechansims. User can only perform authentication using the allowed mechanism. Current authenticaiton mechanims binding is too wide (System wide). 

Extra:
>  Probability: Low

>   Impact: High

>   Related Info:

>   Comments:

####AuthController-003 
> Threats: DoS using Token Validation 

Threat Agent:
> Internet Attacker - Unauthorized

Attack Vectors:
> Token validation request for UUID tokens

Security Weakness:
> Currently there is no white listing check on Token_ID (UUID Token) during token validation. All tokenIDs are considered in correct form, which could result in expensive DB lookup to check the correctness of the Token_ID.

Counter Measures:
> Whitelisting

Extra:
>  Probability: Medium

>   Impact: Low

>   Related Info:

>   Comments:

####AuthController-004
Threats:
> Spoofing: Unauthorized access using Bearer Token Weakness

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


####AuthController-005
Threats:
> Spoofing: Unauthorized access using the weakness
default auth mechanisms : User/Password authentication

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

####AuthController-006
Threats:
> Spoofing/ Elevation of Priviledge: Unauthorized access using the weakness of
default auth mechanisms : token authentication

Threat Agent:
> 

Attack Vectors:
> 

Security Weakness:
> currently a unscoped token can be scoped and scoped token can be unscoped. This is true for both V2 and V3. A malicious entity in possession of a scoped token can create a new token with a changed scope ( a new token with a 
changed role, to which the token owner is a member)

Counter Measures:
> 

Extra:
>  Probability: 

>   Impact: 

>   Related Info:

>   Comments:

####AuthController-007
Threats:
> Spoofing: unauthorized access: 
Impedance mismatch among several authentication mechanism context

Threat Agent:
> 

Attack Vectors:
> 

Security Weakness:
> Keystone currently supports multiple authentication mechanism and authentication chaining. The granularity and scope of each authentication mechanism can be different.

Counter Measures:
> 

Extra:
>  Probability: Medium

>   Impact: 

>   Related Info:

>   Comments:
https://www.isecpartners.com/media/11958/common%20flaws%20of%20distributed%20identity%20and%20authentication%20systems.pdf

####AuthController-007
Threats:
> Spoofing/Elevation of Priviledges: Token Scoping is too large

Threat Agent:
> 

Attack Vectors:
> 

Security Weakness:
> Currently, Token is binded to user, project and role, not with the 
endpoint. A single token is valid for all services/endpoints. Besides, a scoped
token can be used to get an unscoped token and later on scoped token 
makes all User assets vulnerable if it is exposed in any place. Besides,
without bind parama (with deployment shortcomings), the token is not bind to
the credential owner. Anyone, having the token can use it. 

Counter Measures:
> 

Extra:
>  Probability: medium

>   Impact: High

>   Related Info:

>   Comments:

####AuthController-008
Threats:
> Spoofing/DoS: Unsuccessful authentication attempt

Threat Agent:
> Internet Attacker - Unauthorized

Attack Vectors:
> An attacker can issue an unlimited number of authentication request, possibly 
resulting an DoS attack against  the system

Security Weakness:
> By default, the system has no authentication failure handling mechanism e.g., Rate limiting

Counter Measures:
> When a number of unsuccessful authentication attempts has been reached, the system
should have mechanism to restrict (to avoid password guessing attack, DoS attack).

Extra:
>  Probability:  Medium

>   Impact: Medium

>   Related Info:

>   Comments:

####AuthController-009
Threats:
> Spoofing: via poor quality secrects (user secrects)

Threat Agent:
> Internet Attacker - Unauthorized

Attack Vectors:
> An attacker can expoloit a weak secret to easily gain access to the system (e.g., password strenth 
weakness, token_id(MD5 hashed id) weakness) 

Security Weakness:
> Lack of measurement of quality of secret. 

Counter Measures:
> Secrect should be controlled against quality parameters.

Extra:
>  Probability:  Medium

>   Impact: Medium

>   Related Info:

>   Comments:

#### Issues:

Authentication:

1. Issues with identity scope in case of external authentication. There could be many external auth mechanims, how does identity scope in one mechanism interoperate with identity scope with Keystone. For example,  PKIX use distingushed name for subject, Kerberos use SPN or UPN, OAuth use opaaque identifier. How these will interoperate with keystone user_id setting in REMOTE_USER variable. In addition, some mechanism allow wild card , others has black listing in names.

2. Token Bind - token can be binded to e.g., Kerberos. I.e., Kerberos SPN is placed inside the token. The consumer of the token can then verify that whether the requester (current owner) of the token is the owner of the SPN by performing additional checks. It makes unbounded bearer token bounded to the owner. Issues exists with deployment in OpenStack. See. http://www.jamielennox.net/blog/2013/10/22/keystone-token-binding/








  [1]: images/DFD_Auth_Router.png
  [2]: images/DFD_AUTH_CONTROLLER_authenticate_v3.png
  [3]: images/DFD_AUTH_CONTROLLER_Validate_V3.png
  [4]: images/DFD_AUTH_CONTROLLER_Check_V3.png
  [5]: images/DFD_AUTH_CONTROLLER_revoke_V3.png
  [6]: Keystone_asset_library.md
