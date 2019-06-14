# Restoring a TDE encrypted SQL database

Previously we talked about [adding transparent data encryption (TDE) to a SQL database](https://richardhildebrand.wordpress.com/2019/05/18/encrypting-a-sql-server-database-with-transparent-data-encryption/). After adding TDE, we must follow these two additional steps before we can restore our `.bak` files on a different server.


## Step 1:

Add the encryption password to the SQL server you wish to restore the `.bak` file to. This must be the same password you used on the original server.

[code language="SQL"]
    USE master;
    GO  

    CREATE MASTER KEY 
	ENCRYPTION BY PASSWORD = 'your_encryption_password';
    GO
[/code]

## Step 2:

Restore your certificate backup.

[code language="SQL"]
    CREATE CERTIFICATE TDECertificate
        FROM FILE = 'full_file_path_to_certificate_file'
        WITH PRIVATE KEY
        (
            FILE = 'full_file_path_to_private_key_file',
            DECRYPTION BY PASSWORD = 'your_encryption_password'
        );
    GO
[/code]