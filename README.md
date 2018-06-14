![SAP HANA Academy](https://yt3.ggpht.com/-BHsLGUIJDb0/AAAAAAAAAAI/AAAAAAAAAVo/6_d1oarRr8g/s100-mo-c-c0xffffffff-rj-k-no/photo.jpg)
# LDAP #
### Tutorial Video Playlist ### 
[LDAP](https://www.youtube.com/playlist?list=PLkzo92owKnVy851u716gxj4jRiSi7gZkY)

## What's New? ##
In the first video, the concepts of LDAP as it relates to SAP HANA authorization and authentication are explained. 

### Tutorial Video ### 
[![What's New](https://img.youtube.com/vi/9OGphP_1npY/0.jpg)](https://www.youtube.com/watch?v=9OGphP_1npY "What's New")

## Create LDAP Provider ##

To configure a connection to an LDAP server in SAP HANA, you need to create an LDAP provider in the (tenant) database with the CREATE LDAP PROVIDER or ALTER LDAP PROVIDER statements.

Access to the LDAP server takes place using an LDAP server user with permission to perform searches as specified by the user look-up URL. The credential of this user is stored in the secure internal credential store.

Communication between SAP HANA and the LDAP server can be secured using the TLS/SSL protocol or Secure LDAP protocol (LDAPS).
```
CREATE LDAP PROVIDER my_ldap_provider
  CREDENTIAL TYPE 'PASSWORD' USING 'user=CN=MyTechnicalUser,CN=Users,DC=MyDC,DC=LOCAL;password=Password1'
  USER LOOKUP URL 'ldap://server.domain:389/CN=Users,DC=MyDC,DC=LOCAL??sub?(&(objectClass=user)(sAMAccountName=*))'
  ATTRIBUTE DN 'distinguishedName'
  ATTRIBUTE MEMBER_OF 'memberOf';
  
-- you need to enable a provider before it can be used (can be included in the CREATE statement)
ALTER LDAP PROVIDER my_ldap_provider
  ENABLE PROVIDER;

-- when not using LDAPS (USER LOOKUP URL), SSL should be used to secure network traffic between the LDAP and SAP HANA server. 
ALTER LDAP PROVIDER my_ldap_provider
  SSL ON;

-- when creating multiple providers, you can assign a particular provider as default
ALTER LDAP PROVIDER my_ldap_provider
  DEFAULT ON;
```  
You can query the and system views for the configuration of LDAP providers and LOOKUP URLs. 
```  
SELECT * FROM LDAP_PROVIDERS;
SELECT * FROM LDAP_PROVIDER_URLS;
```  
To validate that SAP HANA can connect and bind to the LDAP server using the specified credentials of the access lookup account, you can use the VALIDATE LDAP PROVIDER statement. For more options, see below LDAP User Authentication. 
```
VALIDATE LDAP PROVIDER my_ldap_provider; 
```

### Tutorial Video ### 
[![LDAP - Create Provider](https://img.youtube.com/vi/e4beKQRhPQg/0.jpg)](https://www.youtube.com/watch?v=e4beKQRhPQg "LDAP - Create Provider")

### Documentation ### 
* [CREATE LDAP PROVIDER Statement (Access Control)](https://help.sap.com/viewer/4fe29514fd584807ac9f2a04f6754767/latest/en-US/3b722036ba4941df8712508a3b231643.html)
* [VALIDATE LDAP PROVIDER Statement (Access Control)](https://help.sap.com/viewer/4fe29514fd584807ac9f2a04f6754767/latest/en-US/4181217e3e104c57a5090431c1cd70b7.html)
* [Secure Communication Between SAP HANA and an LDAP Directory Server](https://help.sap.com/viewer/b3ee5778bc2e4a089d3299b82ec762a7/latest/en-US/b9086809b9bb466cbd15542430f2ebe6.html)

## LDAP Group Authorization ##

You can use LDAP group membership to authorize existing SAP HANA database users. 
To implement LDAP group authorization, you need to 
- Map LDAP groups to SAP HANA catalog roles using the CREATE ROLE or ALTER ROLE statements
- Configure SAP HANA users for LDAP group authorization

In the example below, we create a role and alter it to map it to an LDAP group. 
```
CREATE ROLE backup_admin_role;
GRANT BACKUP ADMIN TO backup_admin_role;

ALTER ROLE backup_admin_role 
  ADD LDAP GROUP 'CN=HANABackupAdmin,CN=Users,DC=MyDC,DC=LOCAL';

SELECT * FROM ROLE_LDAP_GROUPS;
```
Next, we create a user with LDAP authorization. Upon first connection, this user will be granted the mapped roles. 

Authorization is either LOCAL or LDAP. 

This can be verified by querying the USERS and GRANTED_ROLES system views before and after the initial connection of this user. Initially only the PUBLIC role will have been granted. After the first connection, the user will also have been granted the mapped LDAP group role(s). 
```
CREATE USER UserA PASSWORD Password1 AUTHORIZATION LDAP;

SELECT USER_NAME, CREATOR, CREATE_TIME, LAST_SUCCESSFUL_CONNECT, AUTHORIZATION_MODE 
  FROM USERS
 WHERE USER_NAME = 'USERA';
 
SELECT GRANTEE_TYPE, ROLE_NAME, GRANTOR, IS_GRANTABLE
  FROM GRANTED_ROLES 
 WHERE GRANTEE = 'USERA'; 
```

### Tutorial Video ### 
[![LDAP - Group Authorizations](https://img.youtube.com/vi/2PiYh63RYM8/0.jpg)](https://www.youtube.com/watch?v=2PiYh63RYM8 "LDAP - Group Authorizations")

### Documentation ### 
* [LDAP Group Authorization](https://help.sap.com/viewer/b3ee5778bc2e4a089d3299b82ec762a7/latest/en-US/9fb0ac08b214477b8276af2b68eeefc3.html)

## LDAP User Authentication ##

LDAP authentication can be implemented for users accessing SAP HANA directly via JDBC/ODBC database clients. 

Using LDAP user passwords for authentication eliminates the need to manage user passwords and password policies in the SAP HANA database.
``` 
CREATE USER UserB WITH IDENTITY FOR LDAP PROVIDER;

CONNECT UserB PASSWORD *****;

SELECT USER_NAME, AUTHENTICATION_METHOD 
  FROM M_CONNECTIONS
 WHERE USER_NAME = CURRENT_USER AND CONNECTION_STATUS = 'RUNNING';
``` 
To validate the configuration, you can use the VALIDATE LDAP PROVIDER statement. 
``` 
-- Check that UserB exist
VALIDATE LDAP PROVIDER my_ldap_provider CHECK USER UserB NO AUTHORIZATION CHECK; 

-- Check authorization for UserB 
VALIDATE LDAP PROVIDER my_ldap_provider CHECK USER UserB;

-- Check authentication for UserB 
VALIDATE LDAP PROVIDER my_ldap_provider CHECK USER UserB PASSWORD UserBPassword;
```
Alternatively or additionally, you can enable the LDAP provider for automatic user creation.
```
ALTER LDAP PROVIDER my_ldap_provider ENABLE USER CREATION FOR LDAP;

-- Check LDAP configuration for automatic user creation. 
VALIDATE LDAP PROVIDER my_ldap_provider CHECK USER CREATION FOR LDAP USER UserC
```

### Tutorial Video ### 


### Documentation ### 
* [LDAP User Authentication](https://help.sap.com/viewer/b3ee5778bc2e4a089d3299b82ec762a7/latest/en-US/868f8b988e2d42ccb89ccaf263cd9986.html)
