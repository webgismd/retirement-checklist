# Retirement Process
The intent of the checklist is to provide guidance and consistency throughout retirement excercises. It is meant to apply to data and application retirements. Data Retirements may go through additional process managed by the DA chapter. Further, it is to provide confidence to the resource performing the physical removal/deletion that all necessary checks have been completed.
Process is:
1. Create a task in the Agile Team's project that owns the application or data
2. Follow the checklist within the ticket, record who completed, outcome or where the item was N/A
3. When the data or application is ready for retirement label with the word "retirement" this will have it presented at the Change Coordination Meeting. Do not label with "retirement" if its not intended to be discussed at Change Coordination.

**RACI Abbreviations**
DA: Data Architecture 
DBA: Database Admin 
BA: Business Analyst 
CDA: Catalogue DA 
MW: Middleware 
AD: Application Delivery 
WM: Web Mapping 
TA: Technical Architect 
AO: Application Owner 
DC: Data Custodian 
RB: Retirement Board 

|Section | Details: RACI | Completed By|
|---|---|---|
|Open: Confirm Retirement | TA/DA: Attach Notice of Decision, application owner approvals and business area/client approvals to Jira Ticket | |  

</br>

|Section | Details: RACI | Completed By|
|---|---|---|
|Analysis: Audit Access | DBA: Audit Logins (See Queries Below) | |
|Analysis: Audit Access | DBA: Audit Object Access  (See Queries Below) | |
|Analysis: Audit Access | DBA: Audit Listener | |
|Analysis: Audit Access | MW/WM: Audit Application Use | |
|Analysis: Audit Access | DBA/WM/MW: Report on audit | |
|Analysis: Dependency Analysis | WM: Static Map Service | |
|Analysis: Dependency Analysis | DBA/DA: Object, Sys Priv, Quota Grants to the Schema and Report | |
|Analysis: Dependency Analysis| DBA/DA: Other user grants to Objects and Report | |

</br>

|Section | Details: RACI | Completed By|
|---|---|---|
|Approval: Final Review | RB : Review the findings from analysis and confirm that retirement may proceed | |
|Approval: Change Coordination | RB : Review at Change Coordination | |

</br>

|Section | Details: RACI | Completed By
|---|---|---|
|Implementation: Outstanding Comms | Change Coordination Announcement | |
|Implementation: Update Catalogue Record | DA/AO: Pending Archive Status | |
|Implementation: Mitigate outstanding impacts from analysis | DBA, DA, CDA, AO, AD, MW, WM | |
|Implementation: Disable in DLVR, TEST, PROD | DA: Turn off ETL | |
|Implementation: Disable in DLVR, TEST, PROD | AD: See internal process; Communicate to BA | |
|Implementation: Disable in DLVR, TEST, PROD | BA: Inform client and the business service desk | |
|Implementation: Remove database access | DBA: As per retirement plan (grants, quota, lock account); communicate to BA status | |
|Implementation: Remove database access | BA: Inform client and the business service desk, client to communicate to their users/business area | |
|Implementation: Await Grace Period Expiration (This means the application is un-deployed, and left in place until end of grace period) | AD: Confirm grace period; set a reminder in calendar for end of grace period, BA / Client should be aware of grace period | | 
|Implementation: Remove application from servers or desktop | AD: see internal process; notify BA task is completed | |
|Implementation: Remove application from servers or desktop | BA: to follow-up with AD to ensure task has been completed | |
|Implementation: Remove passwords from Password Manager | AD, DBA, DataBC: remove passwords; notify security | |
|Implementation: Prepare Restore Package | DBA: Backup DDL. Roles. Grants. Registrations, store in Gogs | |
|Implementation: Archive Application in Gogs |AD: create/organize folder(s), BA to be aware | |
|Implementation: Retire data | DBA/DA: Archive Data | |
|Implementation: Retire data | DA: Retire data model | |
|Implementation: Retire data | AO / DC: Confirm archival requirements | |
|Implementation: Update Catalogue Record | DA/AO: Status: Archived | |

</br>

|Section | Details: RACI | Completed By|
|---|---|---|
|Review: Confirm no further requirements found | All | |

</br>
</br>

##Queries
**Analysis: Audit Access: DBA: Audit Logins**
Ex.:
```
  SELECT userid,
         userhost,
         terminal,
         spare1,
         ntimestamp#
    FROM sys.aud$
   WHERE     USERID = UPPER ('<username>')
         AND action# = 100
         AND userhost NOT IN ('diphda', 'DIADEM')
  ORDER BY ntimestamp# DESC;
```

**Analysis: Audit Access: DBA: Audit Object Access**
Ex.:
```
SELECT    'AUDIT SELECT, INSERT, UPDATE, DELETE ON '
       || owner
       || '.'
       || object_name
       || ' BY ACCESS;'
  FROM dba_objects
 WHERE OWNER = 'APP_FOREST_VEGETATION' AND object_type IN ('VIEW', 'TABLE');'
```

**Analysis: Dependency Analysis | DBA/DA: Object Grants to the Schema and Report**
Check for relevant Role Grants
```  
  SELECT *
    FROM DBA_ROLE_PRIVS
   WHERE grantee IN (SELECT username
                       FROM dba_users
                      WHERE     username LIKE '%WMS%'
                            --AND account_status <> 'LOCKED'
                            AND common = 'NO')
ORDER BY granted_role;
```

Check for relevant Sys Privs
```
  SELECT *
    FROM DBA_SYS_PRIVS
   WHERE grantee IN (SELECT username
                       FROM dba_users
                      WHERE username LIKE '%WMS%' --AND account_status <> 'LOCKED'
                                                 AND common = 'NO')
ORDER BY privilege;
```
Revoke Sys Privs
```
  SELECT 'REVOKE ' || PRIVILEGE || ' FROM ' || GRANTEE || ';'
    FROM DBA_SYS_PRIVS
   WHERE grantee IN (SELECT username
                       FROM dba_users
                      WHERE username LIKE '%APP_FOREST%' --AND account_status <> 'LOCKED'
                                                         AND common = 'NO')
ORDER BY privilege;
```

Check for Grants on Schema's Objects
```
   SELECT GRANTEE,
         OWNER,
         TABLE_NAME,
         GRANTOR,
         PRIVILEGE
    FROM dba_tab_privs
   WHERE owner LIKE '%APP_FOREST%'
ORDER BY privilege;
```
Revoke Grants on Schema's objects to Schema
```
  SELECT    'REVOKE '
         || PRIVILEGE
         || ' ON '
         || GRANTOR
         || '.'
         || TABLE_NAME
         || ' FROM '
         || GRANTEE
         || ';'
    FROM dba_tab_privs
   WHERE owner LIKE '%APP_FOREST%'
ORDER BY privilege;
```

Check for Grants on Objects to Schema
```
SELECT GRANTEE,
         OWNER,
         TABLE_NAME,
         GRANTOR,
         PRIVILEGE
    FROM dba_tab_privs
   WHERE owner LIKE '%APP_FOREST%'
ORDER BY privilege;
```

Revoke Grants on Objects to Schema
```
  SELECT    'REVOKE '
         || PRIVILEGE
         || ' ON '
         || OWNER
         || '.'
         || table_name
         || ' FROM '
         || GRANTEE
         || ';'
    FROM dba_tab_privs
   WHERE owner LIKE '%APP_FOREST%'
ORDER BY privilege;
```
**Other Useful Information**
Table
IDWPROD1.WHSE_CORP.DA_BCGW_RETIREMENTS – Retired objects, NOI #,NOD #, replaced by, replaced why, dates and other stuff; Annika populates this


Retirement views
Views are in DBCPRD, APP_EDRP schema

1.	BCGW_DATASET_RETIREMENTS_VW – Retired objects, as above, linked to grantees, grandparents, parents, children, and grandchildren
2.	BCGW_DATASET_RETIREMENTS_MV – instantiated version of BCGW_DATASET_RETIREMENTS_VW
3.	BCGW_RETIREMENTS_DEPEND_DIST – shows retired objects configured for distribution
4.	BCGW_RETIREMENTS_DEPEND_META – links retired objects to Metastar and the BC Data Catalogue
5.	BCGW_RETIREMENTS_DEPEND_MPCM – shows retired objects configured with MPCM

Other useful views
Also in DBCPRD, APP_EDRP schema

1.	BCGW_ALL_OBJS_DEPEND_DIST – links whse objects with distribution configuration entries
2.	BCGW_ALL_OBJS_DEPEND_MPCM – links whse objects with layers configured for MPCM
3.	BCGW_ALL_OBJS_DEPEND_TSAT – links whse objects with layers in the TSAT toolbar

