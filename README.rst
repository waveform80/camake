======
camake
======

This repository conatins some simple scripts, primarily based on GNU Makefiles
to ease the creation, maintenance, and usage of a CA (an SSL Certificate
Authority).  It is not intended, nor fit for use as a large CA, but is fine for
small deployments.


Requirements
============

The system relies on GNU make, and openssl. An Ubuntu installation is assumed
but it wouldn't be difficult to adapt to other Linux distros (the primary thing
to modify would be the group configuration).


Setup
=====

Firstly, create the CA like so::

    # cd ca
    # make init

You can optionally specify the country (c), organization (o), title (cn),
owner's e-mail address (email) and certificate revocation list URL (crl) after
the target.  For example::

    # make init c=GB "o=My Company" title="My Company CA" email=ca@mycompany.com crl=https://mycompany.com/ca-crl.pem

If these items are not provided they will be prompted for interactively. The
command will generate the following important files under the ``ca`` directory:

* ``ca/private/ca-key.pem`` - The private key of the CA. This **must** remain
  secret; never distribute it to anyone. Preferably, keep it on an air-gapped
  machine used only for signing certificates ("proper" CAs keep the root key
  entirely air-gapped and generate intermediate signing keys which can be more
  easily revoked for day-to-day signing usage).

* ``ca/ca-cert.pem`` - The public CA certificate in PEM format. You can
  distribute this to servers or users who need to determine whether
  certificates were signed by the CA. Feel free to make it publically
  available from your organization's website.

* ``ca/ca-cert.der`` - The public CA certificate in DER format.

* ``ca/ca-cert.txt`` - The public CA certificate in human readable form.

* ``ca/ca-crl.pem`` - The CA's certificate revocation list. You should make
  this available from the URL specified for the ``crl`` parameter when you
  created the CA.

To regenerate the CRL you can run the following command at any time::

    # cd ca
    # make gencrl

You can use the following command to obtain help on the available commands
at any time::

    # cd ca
    # make help


Server Certificates
===================

To create a new server certificate the procedure goes as follows. Firstly,
create a private key and certificate signing request (CSR) for the server in
question::

    # cd servers
    # make csr file=myhost

As with creating a CA you can specify the parameters for the server certificate
on the command line as well::

    # make csr file=myhost c=GB o="My Company" cn=myhost.mycompany.com email=webmaster@myhost.mycompany.com

The most important attribute is the common name (CN). This **must** match the
hostname of the server that the certificate is destined for. The command will
generate the following files:

* ``servers/myhost-key.pem`` - The private key for the server. The server in
  question will need this, but other than this it should be kept secret.

* ``servers/myhost-csr.pem`` - This is the certificate signing request.

Copy the CSR to the ``ca`` sub-directory, then have the CA sign the request to
generate the signed certificate::

    # cp myhost-csr.pem ../ca
    # cd ../ca
    # make sign file=myhost

This will prompt you with the certificate's details to make sure you wish to
sign them. Once this is complete the CSR will be removed, and the certificate
will be created in the following file:

* ``ca/myhost-cert.pem`` - This is the signed certificate in PEM format.

Copy the certificate back to the ``servers`` sub-directory and (optionally)
generate the alternate formats from it::

    # cp myhost-cert.pem ../servers
    # cd ../servers
    # make convert file=myhost

You can use the following command to obtain help on the available commands
at any time::

    # cd servers
    # make help


Client Certificates
===================

To create a new client certificate the procedure goes as follows. Firstly,
create a private key and certificate signing request (CSR) for the client in
question::

    # cd clients
    # make csr file=myuser

As with creating a CA you can specify the parameters for the server certificate
on the command line as well::

    # make csr file=myuser c=GB o="My Company" cn=myuser email=myuser@mycompany.com

Again, the most important attribute is the common name (CN). This **must**
match the username of the client in question. The command will generate the
following files:

* ``clients/myuser-key.pem`` - The private key for the user. The user in
  question will need this, but other than this it should be kept secret.

* ``clients/myuser-csr.pem`` - This is the certificate signing request.

Copy the CSR to the ``ca`` sub-directory, then have the CA sign the request to
generate the signed certificate::

    # cp myuser-csr.pem ../ca
    # cd ../ca
    # make sign file=myuser

This will prompt you with the certificate's details to make sure you wish to
sign them. Once this is complete the CSR will be removed, and the certificate
will be created in the following file:

* ``ca/myuser-cert.pem`` - This is the signed certificate in PEM format.

Copy the certificate back to the ``clients`` sub-directory::

    # cp myuser-cert.pem ../clients

You can use the following command to obtain help on the available commands
at any time::

    # cd clients
    # make help


Revocation
==========

If you believe that a certificate has been compromised you can revoke it, and
regenerate the certificate recovation list as follows::

    # cd ca
    # make revoke file=myhost
    # make gencrl

Note that revocation requires the signed (server or client) certificate. This
is the reason for *copying* signed certificates from the ``ca`` sub-directory
rather than moving them in the sections above.

Certificates have a limited life-span as it is. The Makefiles default to 10
years expiration and 4096 bits strength for the CA certificate, and 1 year
validity and 1024 bits strength for signed server and client certificates
(these are probably weak by the time you're reading this; adjust these limits
in the configuration variables at the top of the relevant Makefiles
accordingly).

