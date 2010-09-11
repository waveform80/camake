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
	@{ \
		export CERT_C="$(c)"; \
		export CERT_O="$(o)"; \
		export CERT_CN="$(cn)"; \
		export CERT_EMAIL="$(email)"; \
		[ ! -n "$$CERT_C" ]     && read -p "ISO-3166 country code (C): " CERT_C; \
		[ ! -n "$$CERT_O" ]     && read -p "Organization name (O)    : " CERT_O; \
		[ ! -n "$$CERT_CN" ]    && read -p "Username (CN)            : " CERT_CN; \
		[ ! -n "$$CERT_EMAIL" ] && read -p "User's e-mail            : " CERT_EMAIL; \
		$(OPENSSL) req -config openssl.cnf \
			-new -key $(KEYFILE) -out $@; \
	}

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

