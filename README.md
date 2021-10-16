# THIS IS JUST A FORK WITH https://github.com/mariadb-corporation/mariadb-connector-cpp/pull/7 from DalenPlanestrider To be able to build static library

# MariaDB Connector/C++
<p align="center">
  <a href="http://mariadb.com/">
    <img src="https://mariadb.com/kb/static/images/logo-2018-black.png">
  </a>
</p>

This is a GA release of the MariaDB Connector/C++.

MariaDB Connector/C++ is released under version 2.1 of the
GNU Lesser Public License.

License information can be found in the COPYING file.
[![License (LGPL version 2.1)](https://img.shields.io/badge/license-GNU%20LGPL%20version%202.1-green.svg?style=flat-square)](http://opensource.org/licenses/LGPL-2.1)

To report issues: [https://jira.mariadb.org/projects/CONCPP/issues/](https://jira.mariadb.org/projects/CONCPP/issues/)

Documentation: [https://mariadb.com/docs/appdev/connector-cpp/](https://mariadb.com/docs/appdev/connector-cpp/)

## Basic Use

Connector implements JDBC API with minimal extensions and minimal
reductions of methods, that do not make sense in C++

To be able to use API, a program should include its definition in ConnCpp.h
header
```script
#include  <mariadb/conncpp.hpp>
```
To obtain driver instance:
```script
sql::Driver* driver= sql::mariadb::get_driver_instance();
```
To connect

```script
sql::SQLString url("jdbc:mariadb://localhost:3306/db");
sql::Properties properties({{"user", "root"}, {"password", "someSecretWord"}});
std::unique_ptr<Connection> conn(driver->connect(url, properties));
```
Alternatively, connection object can be obtained via DriverManager class. In this case Driver object is not needed,
and the connector throws exception, if connection could not be established for whatever reason. Please note, that
for conencting via DriverManager only "jdbc:mariadb://" type of URL can be used.

```script
sql::SQLString url("jdbc:mariadb://localhost:3306/db");
sql::Properties properties({{"user", "roor"}, {"password", "someSecretWord"}});
std::unique_ptr<Connection> conn(DriverManager::getConnection(url, properties));
// or
sql::SQLString url2("jdbc:mariadb://localhost:3306/db?user=root&password=someSecretWord");
std::unique_ptr<Connection> conn2(DriverManager::getConnection(url));
// or
std::unique_ptr<Connection> conn3(DriverManager::getConnection(url, "root", "someSecretWord"));
```

For URL syntax and options name you may find [here](https://mariadb.com/kb/en/about-mariadb-connector-j/)
but not all options will have effect at the moment. In particular, not supported are(list may be incomplete):
  - keyStore
  - keyStorePassword
  - trustStore
  - trustStorePassword
  - keyStoreType
  - trustStoreType
  - allowPublicKeyRetrieval
  - tlsSocketType

Not complete list of supported options - mostly newly added or with new aliases:

|Option|Description|Type|Default|Aliases|
|---:|---|:---:|:---:|---|
| **`useServerPrepStmts`** |Whether to use Server Side Prepared Statements(SSPS) for PreparedStatement by default, and not client side ones(CSPS)|*bool* |false||
| **`connectTimeout`** |The connect timeout value, in milliseconds, or zero for no timeout.|*int* |30000||
| **`socketTimeout`** |Specifies the timeout in seconds for reading packets from the server. Value of 0 disables this timeout.|*int* |0|OPT_READ_TIMEOUT|
| **`autoReconnect`** |Enable or disable automatic reconnect.|*bool* |false|OPT_RECONNECT|
| **`tcpRcvBuf`** |The buffer size for TCP/IP and socket communication. `tcpSndBuf` changes the same buffer value, and the biggest value of the two is selected|*int* |0x4000|tcpSndBuf|
| **`tcpSndBuf`** |The buffer size for TCP/IP and socket communication. `tcpRcvBuf` changes the same buffer value, and the biggest value of the two is selected|*int* |0x4000|tcpRcvBuf|
| **`localSocket`** |For connections to localhost, the Unix socket file to use.|*string* |||
| **`pipe`** |On Windows, specify the named pipe name to connect.|*string* |||
| **`useTls`** |Whether to force TLS. This enables TLS with the default system settings. |*bool* ||useSsl,useSSL|
| **`tlsKey`** |File path to a private key file |*string* ||sslKey|
| **`keyPassword`** |Password for the private key |*string* ||MARIADB_OPT_TLS_PASSPHRASE|
| **`tlsCert`** |Path to the X509 certificate file|*string* ||sslCert|
| **`tlsCA`** |A path to a PEM file that should contain one or more X509 certificates for trusted Certificate Authorities (CAs)|*string* ||tlsCa,sslCA|
| **`tlsCAPath`** |A path to a directory that contains one or more PEM files that should each contain one X509 certificate for a trusted Certificate Authority (CA) to use|*string* ||tlsCaPath, sslCAPath|
| **`enabledTlsCipherSuites`** |a list of permitted ciphers or cipher suites to use for TLS|*string* ||enabledSslCipherSuites, enabledSSLCipherSuites|
| **`tlsCRL`** |path to a PEM file that should contain one or more revoked X509 certificates|*string* ||tlsCrl, sslCRL|
| **`tlsCRLPath`** |A path to a directory that contains one or more PEM files that should each contain one revoked X509 certificate. The directory specified by this option needs to be run through the openssl rehash command. This option is only supported if the connector was built with OpenSSL.|*string* ||tlsCrlPath, sslCRLPath|
| **`tlsPeerFP`** |A SHA1 fingerprint of a server certificate for validation during the TLS handshake.|*string* ||tlsPeerFp, MARIADB_OPT_SSL_FP|
| **`tlsPeerFPList`** |A file containing one or more SHA1 fingerprints of server certificates for validation during the TLS handshake.|*string* ||tlsPeerFpList, MARIADB_OPT_SSL_FP_LIST|
| **`serverRsaPublicKeyFile`** |The name of the file which contains the RSA public key of the database server. The format of this file must be in PEM format. This option is used by the caching_sha2_password client authentication plugin.|*string* ||rsaKey|
| **`useCompression`** |Compresses the exchange with the database|*bool* |false|CLIENT_COMPRESS|
| **`jdbcCompliantTruncation`** |Truncation error will be thrown as error, and not as warning|*bool* |true||
| **`useCharacterEncoding`** |Character set used for text encoding.|*string* ||OPT_SET_CHARSET_NAME,useCharset|
| **`credentialType`** |Default authentication client-side plugin to use.|*string* ||defaultAuth|


Properties is map of strings, and is another way to pass optional parameters.

In addition to Connector/J URL style and JDBC connect method, Connector/C++ supports
following connect methods:

```script
sql::SQLString user("root");
sql::SQLString pwd("root");
std::unique_ptr<sql::Connection> conn(driver->connect("tcp://localhost:3306/test", user, pwd));
```
and

```script
sql::Properties properties;
properties["hostName"]= "127.0.0.1";
properties["userName"]= "root";
properties["password"]= "root";
std::unique_ptr<sql::Connection> conn(driver->connect(properties));
```

Host in former case can be defined as  (tcp|unix|pipe)://\<host\>[:port][/<db>]
Properties in the latter case are the same, as in the first variant of the connect
method, plus additionally supported
  - hostName
  - pipe
  - socket
  - schema
  - userName(alias for user)
  - password

std::string variables can be passed, where parameters are expected to be sql::SQLString

For further use one may refer to JDBC specs.

Except Driver object, normally this is application's responsibility to delete
objects returned by the connector.
