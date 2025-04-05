This repo is for SQL, SQLi and MSSQL commands, links, courses taken. 

In order to access data, an SQL server login has to be mapped to a database user (unless you are a SYSAdmin). A database user is created separately within the database level.

There are:
- Windows accounts
- SQL server logins (inside the SQL Server)
- Database User (inside the SQL Server)

Window accounts and SQL server logins are used to sign in into the SQL server. You will come across the following MSSQL server roles: sysadmin role and public role. A sysadmin can be seen as the equivalent of a Windows administrator for SQL Server. The public role is the least privileged role, but can still allow someone to only connect to the SQL server. It's similar to the Everyone group in Windows.

Locating and Accessing SQL Servers:
By:
- unauthenticated users
- local users
- domain users

Unauthenticated users:
- a pentester can find databases by employing standard scanning methods: TCP, UDP, UDP broadcast.
- tools: nmap, Nessus, SQLPing3, OSQL/SQLCMD, PowerUpSQL.

e.g.
sqlcmd -L
or,
(PowerSQL):
Get-SQLInstanceScanUDP

Local users:
- trivial to identify as a local user by checking system services and registry settings
- PowerUpSQL:
Get-SQLInstanceLocal


Domain User:
- registered in Active Directory, with an associated account
- Kerberos authentication
- SPN scanning, setspn.exe, Get-Spn.psm1 and PowerUpSQL.
- PowerUpSQL:
Get-SQLInstanceDomain

Initial foothold:
- escalate from an unauthenticated user, local user or domain to an SQL login
- we can perform dictionary attacks using commonly used credentials. Avoid account lockouts.
- PowerUpSQL:
Get-SQLInstanceScanUDP | Invoke-SQLAuditWeakLoginPw
or
Get-SQLInstanceDomain | Invoke-SQLAuditWeakLoginPw

- You can also try logging in to the SQL server instances with a set of credentials:
(PowerUpSQL):
Get-SQLInstanceScanUDP | Get-SQLConnectionTestThreaded -Username <username> -Password <password>

Many applications with SQL Server Express are backend, and are setup with vendor specific credentials and instance names due to vendor recommendations. Such credentials should be considered.

(Gaining initial foothold on SQL server):
- launch a default password test against the found SQL server instances with PowerUpSQL (Domain user):
Get-SQLInstanceDomain | Invoke-SQLAuditDefaultLoginPw -Verbose
or:
Get-SQLInstanceDomain | Get-SQLServerLoginDefaultPw -Verbose

- trying to login to the identified SQL Server instance using the current account, using PowerUpSQL (Domain or Local user perspective), look for Status - Accessible
Get-SQLInstanceDomain | Get-SQLConnectionTest
or:
Get-SQLInstanceLocal | Get-SQLConnectionTest

Consider unencrypted SQL Server communications for the initial foothold on an SQL Server.
Depending on the victim's privileges we can create (inject) our own SQL login.

Find sqlmitm.py by Anitian for injection (altering) SQL queries on the fly.

Escalate priviledges:
SQL login -> sysadmin

To escalate from the public role level to sysadmin level privileges we can exploit the following:
- Weak passwords & Blind SQL Server Login Enumeration
- Impersonation Privileges
- Stored Procedures and Trigger Creation/Injection Issues
- Automatic Execution of Stored Procedures

1. Weak Passwords and Blind SQL Server Login Enumeration:
- if we list all SQL Server login, through our initial foothold, we can see a subset of them.
- List all SQL server login:
SELECT name From sys.syslogins
SELECT name FROM sys.server_principals

- we can utilize the suser_name function, which returns the principal name for a given principal id. We can therefore identify all SQL login by fuzzing the principal id value, inside the suser_name function, through the public role.
- FUZZ the principal id value, inside the suser_name function by executing the following queries:
SELECT SUSER_NAME(1)
SELECT SUSER_NAME(2)
SELECT SUSER_NAME(3)
....

- then, we can try to identify weak passwords on those identified SQL Server logins.
- If this fails, we can perform blind domain account/objects enumeration through our initial foothold, again with the public role.
- Target the identified domain users and continue from there, again checking for weak passwords. In addition, imagine how useful this is in the case of a remote SQL injection based attack.
- The procedure:
a) Get the domain where the SQL Server is located:
SELECT DEFAULT_DOMAIN() as mydomain

b) Get the full RID of Domain Admins group:
SELECT SUSER_SID('identified_domain\Domain Admins')

c) Grab the first 48 bytes of the full RID, to get the SID for the domain. Then, create a new RID (that will be associated with a domain object) by appending a hex number value to the abovementioned SID.
d) Finally, use the suser_name function to get the domain object name associated with the supplied RID.
SELECT SUSER_NAME(RID)

Blind SQL login enumeration is performed with the PowerUpSQL's 'Get-SQLFuzzServerLogin' function whereas blind domain account enumeration can be performed through the 'Get-SQLFuzzDomainAccount' function.

To perform blind SQL login enumeration against the accessible instance we identified, execute the following:
Get-SQLFuzzServerLogin -instance ServerName\InstanceName

To perform blind domain account enumeration against the accessible instance:
Get-SQLFuzzDomainAccount -Instance ServerName\InstanceName
(look out for domain accounts on the SQL Server)

2) The next SQL Server feature we can leverage to gain sysadmin level privileges.

a) Impersonate Privileges
b) Stored Procedure and Trigger Creation/Injection Issues
c) Automatic Execution of Stored Procedures
d) Agent Jobs
e) xp_cmdshell proxy account
f) create database link to file or server
g) import/install custom assemblies
h) ad-hoc queries
i) shared service accounts
j) database links
k) UNC Path Injection
l) Python code execution





  







- 


  
