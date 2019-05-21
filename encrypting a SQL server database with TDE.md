# Encrypting a SQL server database with Transparent Data Encryption

Setting up Transparent Data Encryption (TDE) encryption will protect all of the data inside of a database. This can be done on an existing database, without the need to change the database.

After setting up TDE, the encryption certificate and the master key will be required to access the database and will provide protection for the data, log files, and backups.

## Setting up master

First we will create our encryption pasword using the `master` database.

### Create the Database Master Key (DMK)

[code language="SQL"]
    USE master;
    GO
    
    CREATE MASTER KEY
    ENCRYPTION BY PASSWORD = 'your_encryption_key'
[/code]

Next, we will create our encryption certificate.

### Create the certificate

[code language="SQL"]
    CREATE CERTIFICATE TDECertificate
    WITH SUBJECT = 'example TDE certificate';
[/code]

## Encrypting the database

Now we can create the encryption key for the database we wish to encrypt. Doing so will yield the following warning: `The certificate used for encrypting the database encryption key has not been backed up. You should immediately back up the certificate and the private key associated with the certificate. If the certificate ever becomes unavailable or if you must restore or attach the database on another server, you must have backups of both the certificate and the private key or you will not be able to open the database.` We will follow this advice in one moment.

[code language="SQL"]
    USE DatabaseToEncrypt;
    GO
    
    CREATE DATABASE ENCRYPTION KEY
    WITH ALGORITHM = AES_256
    ENCRYPTION BY SERVER CERTIFICATE TDECertificate;
[/code]

And then turn our encryption on.

[code language="SQL"]
    ALTER DATABASE DatabaseToEncrypt
    SET ENCRYPTION ON;
[/code]

## Backing up our certificates and keys

Since our data is now protected by TDE, it is important to save our keys and certificate so that we do not lose access to our data in the future.

### Backup our Service Master Key (SMK)

[code language="SQL"]
    Use master;
    GO
    
    BACKUP SERVICE MASTER KEY 
    TO FILE = 'C:\Backups\SvcMasterKey.key'
    ENCRYPTION BY PASSWORD = 'your_encryption_key';
[/code]

### Backup our Database Master Key (DMK)

[code language="SQL"]
    BACKUP MASTER KEY 
    TO FILE = 'C:\Backups\DbMasterKey.key'
    ENCRYPTION BY PASSWORD = 'your_encryption_key'
[/code]

### Backup our Certificate

[code language="SQL"]
    BACKUP CERTIFICATE TDECertificate 
    TO FILE = 'C:\Backups\TDECertificate.cer'
    WITH PRIVATE KEY (
        FILE = 'C:\Backups\TDECertificate.key',
        ENCRYPTION BY PASSWORD = 'your_encryption_key'
    );
[/code]