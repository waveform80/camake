OPENSSL=openssl

# how many days self-signed/temp keys should be good for
DURATION=365

# how many bits RSA keys should be
BITS=1024

file=server
GROUP=ssl
KEYFILE=$(file)-key.pem
CERTFILE=$(file)-cert.pem
KEYCERTFILE=$(file)-keycert.pem
CSRFILE=$(file)-csr.pem
DHFILE=dh$(BITS).pem

csr: $(KEYFILE) $(CSRFILE)

convert: $(KEYCERTFILE) $(CERTFILE:.pem=.der) $(CERTFILE:.pem=.txt)

self: $(KEYFILE) $(CERTFILE)

dhparam: $(DHFILE)

temp: $(KEYFILE) $(CSRFILE)
	@sg $(GROUP) "$(OPENSSL) x509 -req -days $(DURATION) \
		-signkey $(KEYFILE) < $(CSRFILE) > $(CERTFILE)"

$(KEYFILE):
	@sg $(GROUP) "$(OPENSSL) genrsa $(BITS) > $@"
	@chmod 440 $@

fingerprint:
	@$(OPENSSL) x509 -fingerprint -noout < $(CERTFILE)

$(CSRFILE): $(KEYFILE)
	@sg $(GROUP) "$(OPENSSL) req -config openssl.cnf \
		-new -key $(KEYFILE) -out $@"

$(CERTFILE): $(KEYFILE)
	@sg $(GROUP) "$(OPENSSL) req -config openssl.cnf \
		-new -x509 -nodes -key $(KEYFILE) -days $(DURATION) -sha1 > $@"
	@$(MAKE) $(@:.pem=.der)
	@$(MAKE) $(@:.pem=.txt)

$(KEYCERTFILE): $(KEYFILE) $(CERTFILE)
	@sg $(GROUP) "$(OPENSSL) rsa -in $(KEYFILE) -outform PEM > $@"
	@sg $(GROUP) "$(OPENSSL) x509 -in $(CERTFILE) -outform PEM >> $@"
	@chmod 440 $@

$(DHFILE):
	@sg $(GROUP) "$(OPENSSL) dhparam -out $(DHFILE) $(BITS)"

%.der : %.pem
	@sg $(GROUP) "$(OPENSSL) x509 -outform DER < $< > $@"

%.txt : %.pem
	@sg $(GROUP) "$(OPENSSL) x509 -text < $< > $@"

help:
	@echo OpenSSL Server Certificate Makefile v1.0
	@echo
	@echo 'All targets take an optional "file=server" suffix which specifies the root'
	@echo of the output filename. The default if this suffix is omitted is: $(file)
	@echo
	@echo make csr
	@echo '  - creates a certificate signing request'
	@echo
	@echo make self
	@echo '  - creates a self-signed certificate'
	@echo
	@echo make temp
	@echo '  - creates a temporary certificate (while waiting for CA)'
	@echo
	@echo make convert
	@echo '  - converts the certificate into common formats (PEM, DER, etc.)'
	@echo
	@echo make dhparam
	@echo '  - generates DH parameters for certain software (OpenVPN)'
	@echo
	@echo make fingerprint
	@echo '  - print the fingerprint a certificate file'
	@echo

