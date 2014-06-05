SQL-scripts
===========
USE master;
GO
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'YourPassword';

CREATE CERTIFICATE TFS_TDE WITH SUBJECT = 'CERT_name_ that_ you_want';

--Backup the the KEY and Certificate so important if the servers Crash
BACKUP MASTER KEY TO FILE = 'c:\masterKey\exportedmasterkey' 
    ENCRYPTION BY PASSWORD = 'YourPASSWORD'; 

BACKUP CERTIFICATE TFS_TDE TO FILE = 'C:\storedcerts\Cer_name.cer'
    WITH PRIVATE KEY ( FILE = 'C:\storedkeys\Key_name.pvk', 
    ENCRYPTION BY PASSWORD = 'YourPassword' );

---loop for the databases
declare @dbname nvarchar(MAX);
declare @statement nvarchar(MAX);
declare @statement2 nvarchar(MAX);
declare @statement3 nvarchar(MAX);

declare NameList1 cursor for
SELECT [name] 
FROM master.dbo.sysdatabases
WHERE dbid > 4 

OPEN NameList1
FETCH NEXT FROM NameList1 
INTO @dbname
--Loop
WHILE @@FETCH_STATUS = 0
 BEGIN
SET @statement = 'USE '+'['+@dbname+'];';
  
EXEC dbo.sp_executesql @statement;
SET @statement3 = 'USE '+'['+@dbname+']; '+'CREATE DATABASE ENCRYPTION KEY
WITH ALGORITHM = AES_128
ENCRYPTION BY SERVER CERTIFICATE CERT_name_ that_ you_want;';
EXEC dbo.sp_executesql @statement3;

SET @statement2 = 'ALTER DATABASE '+'['+@dbname+'] SET ENCRYPTION ON;';
EXEC dbo.sp_executesql @statement2;


FETCH NEXT FROM NameList1 INTO @dbname;
END
CLOSE NameList1

--then you can checked if the database are encrypted take in mind depending on the DB size could take hours to complete the process:
SELECT DB_NAME(database_id), encryption_state FROM sys.dm_database_encryption_keys ;
