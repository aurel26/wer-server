# WER Server #

WER Server is a PHP script that implements the Corporate Error Reporting (CER) protocol for Windows clients.
This script can be used to implement a corporate Windows Error Reporting (WER) server.

## Prerequisites ##

Create SQLite database and adjust `$db_file` in the PHP script. A [empty database](./wer.sqlite) is available
in the repository.

```
CREATE TABLE reports (
   id_report INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL UNIQUE, 
   id_file_report VARCHAR UNIQUE NOT NULL, 
   date DATETIME NOT NULL, 
   remote_ip VARCHAR NOT NULL, 
   "event.reporttype" INTEGER, 
   "event.type" VARCHAR (255), 
   "event.description" VARCHAR, 
   "event.friendlyname" VARCHAR, 
   "event.time" BIGINT, 
   "application.name" VARCHAR (255), 
   "application.path" VARCHAR (255), 
   "application.company" VARCHAR (255), 
   "machine.name" VARCHAR (255), 
   "machine.os" VARCHAR (255), 
   "user.name" VARCHAR, 
   "files.count" INTEGER, 
   "signature.0.name" VARCHAR (255), 
   "signature.0.value" VARCHAR (255), 
   "signature.1.name" VARCHAR (255), 
   "signature.1.value" VARCHAR (255), 
   "signature.2.name" VARCHAR (255), 
   "signature.2.value" VARCHAR (255), 
   "signature.3.name" VARCHAR (255), 
   "signature.3.value" VARCHAR (255), 
   "signature.4.name" VARCHAR (255), 
   "signature.4.value" VARCHAR (255), 
   "signature.5.name" VARCHAR (255), 
   "signature.5.value" VARCHAR (255), 
   "signature.6.name" VARCHAR (255), 
   "signature.6.value" VARCHAR (255), 
   "signature.7.name" VARCHAR (255), 
   "signature.7.value" VARCHAR (255), 
   "signature.8.name" VARCHAR (255), 
   "signature.8.value" VARCHAR (255), 
   "signature.9.name" VARCHAR (255), 
   "signature.9.value" VARCHAR (255)
   );
```

Enable WebDAV support in your Web server:
 * Apache: mod_dav and mod_dav_fs
 * nginx: ngx_http_dav_module
 * IIS: WebDAVModule (add `WebDav Publishing` role)

Enable the following modules in PHP:
 * XML (extension=xml.so)
 * SimpleXML (extension=simplexml.so)
 * PDO (extension=pdo.so)
 * SQLite 3.x driver for PDO (extension=pdo_sqlite.so)

## Configuration for Apache ##

Associate `/stage2.htm` to the PHP handler. (TODO: improve security to allow only `/stage2.htm`).
```
   <FilesMatch "stage2.htm">
      SetHandler application/x-httpd-php
   </FilesMatch>
```

Activate WebDAV for `reports` directory. (TODO: improve security to allow only existing directory to be writable).
```
   <Directory /var/www/html/reports>
      Dav on
      <LimitExcept PUT>
         Require all granted
      </LimitExcept>
   </Directory>
```

## Configuration for IIS ##

Activate WebDAV for the Web site and add a WebDAV Authoting Rule for the `reports` directory with the following parameters:
 * Allow access to: All content
 * Allow access to this content to: All users
 * Permissions: Write

Anonymous PUT method is deprecated on IIS 7+ [1]. So, on the `reports` directory, you need to disable the Anonymous Authentication
and enable the Windows Authentication. WER client needs also to be configured to activate authentication (see Client configuration).
However, WER client authentication requires activation of TLS and you need to install a valid X.509 certificate and the private key
for the Web site.

[1]: https://blogs.msdn.microsoft.com/saurabh_singh/2010/12/10/anonymous-put-in-webdav-on-iis-7-deprecated/

## Client configuration ##

[Reference](https://msdn.microsoft.com/fr-fr/library/windows/desktop/bb513638.aspx)

Configuration with GPO:
```
Computer Configuration
   Policies
      Administrative Templates
         Windows Components
            Windows Error Reporting
               Do not throttle additional data: Enabled
            Windows Error Reporting/Advanced Error Reporting Settings
               Configure Corporate Windows Error Reporting: set 'Corporate server name' and 'Server port'. For IIS, you must set 'Connect using SSL' to Enabled.
            Windows Error Reporting/Consent
               Configure Default consent: Enabled with 'Consent level' set to 'Send all data'
               Ignore custom consent settings: Enabled

User Configuration
   Policies
      Administrative Templates
         Windows Components
            Windows Error Reporting
               Do not throttle additional data: Enabled
            Windows Error Reporting/Consent
               Configure Default consent: Enabled with 'Consent level' set to 'Send all data'
               Ignore custom consent settings: Enabled               
```

If you use IIS, Windows integrated authentication must be activated (WebDAV doesn't support anonymous PUT). To activate authentication,
set 1 to `CorporateWERUseAuthentication` in `HKLM\Software\Policies\Microsoft\Windows\Windows Error Reporting` or
`HKLM\Software\Microsoft\Windows\Windows Error Reporting`. Consider extend `ErrorReporting.admx`.