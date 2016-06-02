# WER Server #

WER Server is a PHP script that implements the Corporate Error Reporting (CER) protocol for Windows clients.
This script can be used to implement a corporate Windows Error Reporting (WER) server.

## Prerequisites ##

Create SQLite database and adjust `$db_file` in the PHP script.

```
CREATE TABLE reports (
   id_report INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL UNIQUE,
   id_file_report VARCHAR UNIQUE NOT NULL,
   date DATETIME NOT NULL,
   remote_ip VARCHAR NOT NULL,
   reporttype INTEGER,
   eventtype VARCHAR (0),
   eventdescription VARCHAR,
   friendlyeventname VARCHAR,
   eventtime BIGINT,
   appname VARCHAR (255),
   apppath VARCHAR (255),
   appcompany VARCHAR (255),
   machinename VARCHAR (255),
   username VARCHAR,
   files_count INTEGER
   );
```

Enable the following modules in PHP:
 * XML (extension=xml.so)
 * SimpleXML (extension=simplexml.so)
 * PDO (extension=pdo.so)
 * SQLite 3.x driver for PDO (extension=pdo_sqlite.so)
 * WebDAV (mod_dav and mod_dav_fs)

Associate `/stage2.htm` to the PHP handler. (TODO: improve security to allow only `/stage2.htm`).
   <FilesMatch "stage2.htm">
      SetHandler application/x-httpd-php
   </FilesMatch>

Activate WebDav for reports directory. (TODO: improve security to allow only existing directory to be writable).
   <Directory /var/www/html/reports>
      Dav on
      <LimitExcept PUT>
         Require all granted
      </LimitExcept>
   </Directory>