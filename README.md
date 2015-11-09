# 4d-plugin-curl
This is a 4D plugin implementation of [libcurl and cURL](http://curl.haxx.se).

Important
---
This plugin project is a forked subset of what was published as [OAuth](https://github.com/miyako/4d-plugin-oauth).

Existing cURL@ commands have the same name and functionality, but their tokens (internal IDs) have changed.

To migrate existing methods, do the following:

1. Comment the code that calls cURL@ plugin commands.
2. Close 4DB.
3. Replace the plugin.
4. Uncomment the code.
 
New
---

###Progress callback

```
$url:="http://www.4d.com/sites/default/files/Homepage-banner_4D-Mobile_ja.jpg"

Progress QUIT (0)

$progressId:=Progress New 
Progress SET PROGRESS ($progressId;-1)
Progress SET BUTTON ENABLED ($progressId;True)

ARRAY LONGINT($optionNames;0)
ARRAY TEXT($optionValues;0)

APPEND TO ARRAY($optionNames;CURLOPT_XFERINFOFUNCTION)
APPEND TO ARRAY($optionValues;"CB_PROGRESS_DL")

APPEND TO ARRAY($optionNames;CURLOPT_XFERINFODATA)
APPEND TO ARRAY($optionValues;String($progressId))  //will be passed as $1

$err:=cURL ($url;$optionNames;$optionValues;$in;$out)

Progress QUIT ($progressId)
```

``CB_PROGRESS_DL``

```
C_LONGINT($1;$2;$3;$4;$5)
C_BOOLEAN($0)

$progressId:=$1
  //download info
$dltotal:=$2
$dlnow:=$3
  //upload info
  //$ultotal:=$4
  //$ulnow:=$5

$progress:=Choose($dltotal#0;$dlnow/$dltotal;-1)
$message:=Choose($progress#-1;String($dlnow)+"/"+String($dltotal)+" bytes downloaded";"connecting...")

Progress SET PROGRESS ($progressId;$progress;$message)

$0:=Progress Stopped ($progressId)
```

###cURL Get executable

Returns the path to the curl executable embedded in the plugin. You can use this with LAUNCH EXTERNAL PROCESS.

###cURL Get version 

```
$version:=cURL Get version 
  //libcurl/7.40.0 OpenSSL/1.0.1j zlib/1.2.8 libidn/1.29 libssh2/1.4.3

$path:=cURL Get executable 

$stdOut:=""
$stdIn:=""
$stdErr:=""

LAUNCH EXTERNAL PROCESS($path+" -V";$stdIn;$stdOut;$stdErr);
  //curl 7.40.0 (x86_64-apple-darwin14.0.0) libcurl/7.40.0 OpenSSL/1.0.1j zlib/1.2.8 libidn/1.29 libssh2/1.4.3\nProtocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smb smbs smtp smtps telnet tftp \nFeatures: IDN IPv6 Largefile NTLM NTLM_WB SSL libz TLS-SRP UnixSockets \n
```

###HTTPS debug

```
C_BLOB($in;$out)
C_LONGINT($err)
ARRAY LONGINT($tNomOption;0)
ARRAY TEXT($tValOption;0)
APPEND TO ARRAY($tNomOption;CURLOPT_SSL_VERIFYHOST)
APPEND TO ARRAY($tValOption;"1")

APPEND TO ARRAY($tNomOption;CURLOPT_SSL_VERIFYPEER)
APPEND TO ARRAY($tValOption;"1")
APPEND TO ARRAY($tNomOption;CURLOPT_CAINFO)
APPEND TO ARRAY($tValOption;Convert path system to POSIX(Get 4D folder(Current resources folder)+"cacert.pem"))

APPEND TO ARRAY($tNomOption;CURLOPT_DEBUGFUNCTION)
APPEND TO ARRAY($tValOption;"CB_DEBUG")

$err:=cURL ("https://github.com/";$tNomOption;$tValOption;$in;$out)
```

``CB_DEBUG``

```
C_LONGINT($1)
C_BLOB($2)

$infoType:=Choose($1;"info";"headerIn";"headerOut";"dataIn";"dataOut")
$message:=BLOB to text($2;Mac text without length)

MESSAGE($infoType+": "+$message)
```

Version
---
* v14 is for v14 and above, Windows & OS X 10.8+ 32/64 bits.
* v11 is for v11 and above, Windows 32/64 bits, OS X 10.6+ 32 bits (Intel only)

Dependencies
---

**Mac OS X**

* libcurl/7.40.0
* OpenSSL/1.0.1j 
* zlib/1.2.8 
* libidn/1.29 
* libssh2/1.4.3
 
**Windows**

* libcurl/7.40.0
* OpenSSL/1.0.1j
* zlib/1.2.8
* libidn/1.29
* libssh2/1.4.3

|Protocol|Mac OS X|Windows|
|:-------:|:-:|:-----:|
|dict|◯|◯|
|file|◯|◯|
|ftp|◯|◯|
|ftps|◯|◯|
|gopher|◯|◯|
|http|◯|◯|
|https|◯|◯|
|imap|◯|◯|
|imaps|◯|◯|
|ldap|◯|◯|
|ldaps|◯|◯|
|pop3|◯|◯|
|pop3s|◯|◯|
|rtsp|◯|◯|
|scp|◯|◯|
|sftp|◯|◯|
|smtp|◯|◯|
|smtps|◯|◯|
|telnet|◯|◯|
|tftp|◯|◯|

|Feature|Mac OS X|Windows|
|:-----:|:-:|:-----:|
|IDN|◯|◯|
|IPv6|◯|◯|
|Largefile|◯|◯|
|NTLM|◯|◯|
|SSL|◯|◯|
|libz|◯|◯|

Remarks
---
On Mac, SSH2 is linked to the system OpenSSL located at /usr/lib/, not the one embedded in the plugin. [This is to avoid crash with SFTP](http://forums.4d.fr/Post/FR/15200699/1/15251183).

On Windows, OPENSSL (LIBEAY and LIBSSL) and LIBSSH2 are statically linked to LIBCURL, to avoid collision with the DLL included in 4D itself.

LIBCURL is modified so that it will yield to 4D during a long operation (easy.c).

