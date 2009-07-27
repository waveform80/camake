OPENSSL=openssl

# how many bits RSA keys should be
BITS=1024

file=client
KEYFILE=$(file)-key.pem
CERTFILE=$(file)-cert.pem
CSRFILE=$(file)-csr.pem

csr: $(KEYFILE) $(CSRFILE)

$(KEYFILE):
	@$(OPENSSL) genrsa $(BITS) > $@
	@chmod 400 $@

$(CSRFILE): $(KEYFILE)
	@$(OPENSSL) req -config openssl.cnf -new -key $(KEYFILE) -out $@

fingerprint: $(CERTFILE)
	@$(OPENSSL) x509 -fingerprint -md5 -noout < $(CERTFILE)
	@$(OPENSSL) x509 -fingerprint -sha1 -noout < $(CERTFILE)

help:
	@echo OpenSSL Client Certificate Makefile v1.0
	@echo
	@echo 'All targets take an optional "file=client" suffix which specifies the root'
	@echo of the output filename. The default if this suffix is omitted is: $(file)
	@echo
	@echo make csr
	@echo '  - creates a certificate signing request'
	@echo
	@echo make fingerprint
	@echo '  - print the fingerprint a certificate file'
	@echo

