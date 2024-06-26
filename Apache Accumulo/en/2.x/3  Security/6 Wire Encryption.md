[TOC]

Accumulo, through Thrift’s TSSLTransport, provides the ability to encrypt wire communication between Accumulo servers and clients using secure sockets layer (SSL). SSL certificates signed by the same certificate authority control the “circle of trust” in which a secure connection can be established. Typically, each host running Accumulo processes would be given a certificate which identifies itself.

Clients can optionally also be given a certificate, when client-auth is enabled, which prevents unwanted clients from accessing the system. The SSL integration presently provides no authentication support within Accumulo (an Accumulo username and password are still required) and is only used to establish a means for secure communication.

Server configuration
----------------------------------------------------------------------------------------------------------

As previously mentioned, the circle of trust is established by the certificate authority which created the certificates in use. Because of the tight coupling of certificate generation with an organization’s policies, Accumulo does not provide a method in which to automatically create the necessary SSL components.

Administrators without existing infrastructure built on SSL are encourage to use OpenSSL and the `keytool` command. An example of these commands are included in a section below. Accumulo servers require a certificate and keystore, in the form of Java KeyStores, to enable SSL. The following configuration assumes these files already exist.

In `accumulo.properties`, the following properties are required:

*   [rpc.javax.net.ssl.keyStore]($Server-Properties-2.x#rpc_javax_net_ssl_keyStore) = _The path on the local filesystem to the keystore containing the server’s certificate_
*   [rpc.javax.net.ssl.keyStorePassword]($Server-Properties-2.x#rpc_javax_net_ssl_keyStorePassword) = _The password for the keystore containing the server’s certificate_
*   [rpc.javax.net.ssl.trustStore]($Server-Properties-2.x#rpc_javax_net_ssl_trustStore) = _The path on the local filesystem to the keystore containing the certificate authority’s public key_
*   [rpc.javax.net.ssl.trustStorePassword]($Server-Properties-2.x#rpc_javax_net_ssl_trustStorePassword) = _The password for the keystore containing the certificate authority’s public key_
*   [instance.rpc.ssl.enabled]($Server-Properties-2.x#instance_rpc_ssl_enabled) = _true_

Optionally, SSL client-authentication (two-way SSL) can also be enabled by setting [instance.rpc.ssl.clientAuth]($Server-Properties-2.x#instance_rpc_ssl_clientAuth) `true` in `accumulo.properties`. This requires that each client has access to a valid certificate to set up a secure connection to the servers. By default, Accumulo uses one-way SSL which does not require clients to have their own certificate.

Client configuration
----------------------------------------------------------------------------------------------------------

To establish a connection to Accumulo servers, each client must also have special configuration. This is typically accomplished by [creating an Accumulo client](https://accumulo.apache.org/docs/2.x/getting-started/clients#creating-an-accumulo-client) using `accumulo-client.properties` and setting the following the properties to connect to an Accumulo instance using SSL:

*   [ssl.enabled]($Client-Properties-2.x#ssl_enabled) to `true`
*   [ssl.truststore.path]($Client-Properties-2.x#ssl_truststore_path)
*   [ssl.truststore.password]($Client-Properties-2.x#ssl_truststore_password)

If two-way SSL is enabled for the Accumulo instance (by setting [instance.rpc.ssl.clientAuth]($Server-Properties-2.x#instance_rpc_ssl_clientAuth) to `true` in `accumulo.properties`), Accumulo clients must also define their own certificate by setting the following properties:

*   [ssl.keystore.path]($Client-Properties-2.x#ssl_keystore_path)
*   [ssl.keystore.password]($Client-Properties-2.x#ssl_keystore_password)

Generating SSL material using OpenSSL
--------------------------------------------------------------------------------------------------------------------------------------------

The following is included as an example for generating your own SSL material (certificate authority and server/client certificates) using OpenSSL and Java’s KeyTool command.

```
# Create a private key
openssl genrsa -des3 -out root.key 4096

# Create a certificate request using the private key
openssl req -x509 -new -key root.key -days 365 -out root.pem

# Generate a Base64-encoded version of the PEM just created
openssl x509 -outform der -in root.pem -out root.der

# Import the key into a Java KeyStore
keytool -import -alias root-key -keystore truststore.jks -file root.der

# Remove the DER formatted key file (as we don't need it anymore)
rm root.der
```

The `truststore.jks` file is the Java keystore which contains the certificate authority’s public key.

### Generate a certificate/keystore per host

It’s common that each host in the instance is issued its own certificate (notably to ensure that revocation procedures can be easily followed). The following steps can be taken for each host.

```
# Create the private key for our server
openssl genrsa -out server.key 4096

# Generate a certificate signing request (CSR) with our private key
openssl req -new -key server.key -out server.csr

# Use the CSR and the CA to create a certificate for the server (a reply to the CSR)
openssl x509 -req -in server.csr -CA root.pem -CAkey root.key -CAcreateserial \
    -out server.crt -days 365

# Use the certificate and the private key for our server to create PKCS12 file
openssl pkcs12 -export -in server.crt -inkey server.key -certfile server.crt \
    -name 'server-key' -out server.p12

# Create a Java KeyStore for the server using the PKCS12 file (private key)
keytool -importkeystore -srckeystore server.p12 -srcstoretype pkcs12 -destkeystore \
    server.jks -deststoretype JKS

# Remove the PKCS12 file as we don't need it
rm server.p12

# Import the CA-signed certificate to the keystore
keytool -import -trustcacerts -alias server-crt -file server.crt -keystore server.jks
```

The `server.jks` file is the Java keystore containing the certificate for a given host. The above methods are equivalent whether the certificate is generated for an Accumulo server or a client.
