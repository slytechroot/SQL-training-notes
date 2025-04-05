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







- 


  
