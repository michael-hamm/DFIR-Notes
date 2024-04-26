## Motivation

Sometimes when creating new users in MISP, they deny to use PGP but want to use
S/MIME for encryption. So they send an email with S/MIME based signature.`

It is quit easy to extract the users public certificate out of the signature.

## Extract the user certificate

1. Convert the DER file into a PEM file:

~~~
$ openssl pkcs7 -inform der -in smime.p7s -out smime.pem
~~~

2. Extract the certificate chain from the PEM file:

~~~
$ openssl pkcs7 -print_certs -in smime.pem -out cert.pem
~~~

3. Copy the user certificate:

Open the certificates file with some pager tool, scroll down to the users 
certificate and copy/paste the certificate into MISP.

