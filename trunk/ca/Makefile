# how many days the CA key should be good for (should be LONG for CAs)
CA_DURATION=3652

# how many days to sign CSRs for (should be relatively short)
SIGN_DURATION=365

# how many days until the next CRL update
CRL_DURATION=7

# how many bits RSA keys should be (should be BIG for CAs, at least 2048)
BITS=4096

# the group which will have read access to generated certificates
GROUP=ssl

OPENSSL=openssl
KEYFILE=private/ca-key.pem
CERTFILE=ca-cert.pem
CRLFILE=ca-crl.pem

export CERT_C:=$(c)
export CERT_O:=$(o)
export CERT_CN:=$(cn)
export CERT_EMAIL:=$(email)
export CERT_CRL:=$(crl)

requests:=$(wildcard *-csr.pem)
certs:=$(requests:-csr.pem=-cert.pem)

init: $(KEYFILE) $(CERTFILE)

sign:
	@test $${file:?"usage: make sign file=request"}
	@$(MAKE) $(file)-cert.pem

revoke: $(KEYFILE) $(CERTFILE)
	@test $${file:?"usage: make revoke file=certificate"}
	@sg $(GROUP) "$(OPENSSL) ca -config openssl.cnf \
		-keyfile $(KEYFILE) \
		-cert $(CERTFILE) \
		-revoke $(file)-cert.pem"
	@$(MAKE) gencrl

gencrl: $(KEYFILE) $(CERTFILE)
	@sg $(GROUP) "$(OPENSSL) ca -config openssl.cnf -gencrl \
		-keyfile $(KEYFILE) \
		-cert $(CERTFILE) \
		-crldays $(CRL_DURATION) \
		-out $(CRLFILE)"

clean:
	-rm $(requests)

reset: clean
	-rm -fr \
		crl/ \
		newcerts/ \
		private/ \
		serial \
		serial.old \
		index \
		index.old \
		index.attr \
		index.attr.old \
		$(KEYFILE) \
		$(CERTFILE) \
		$(CERTFILE:.pem=.der) \
		$(CERTFILE:.pem=.txt) \
		$(CRLFILE)

index:
	@sg $(GROUP) "touch index"

serial:
	@sg $(GROUP) "echo '01' > serial"

crl newcerts:
	@mkdir $@
	@chmod g+s $@
	@chgrp $(GROUP) $@

private:
	@mkdir $@
	@chmod go-rwx $@
	@chgrp $(GROUP) $@

$(KEYFILE):
	@$(MAKE) index serial crl newcerts private
	@{ \
		[ ! -n "$$CERT_C" ]     && read -p "ISO-3166 country code (C): " CERT_C; \
		[ ! -n "$$CERT_O" ]     && read -p "Organization name (O)    : " CERT_O; \
		[ ! -n "$$CERT_CN" ]    && read -p "CA title (CN)            : " CERT_CN; \
		[ ! -n "$$CERT_EMAIL" ] && read -p "CA owner's e-mail address: " CERT_EMAIL; \
		[ ! -n "$$CERT_CRL" ]   && read -p "CA revocation URL        : " CERT_CRL; \
		sg $(GROUP) "$(OPENSSL) req -config openssl.cnf -x509 -nodes \
			-days $(CA_DURATION) -newkey rsa:$(BITS) \
			-keyform PEM -keyout $(KEYFILE) \
			-outform PEM -out $(CERTFILE)"; \
	}
	@sg $(GROUP) "$(OPENSSL) x509 -in $(CERTFILE) -outform DER > $(CERTFILE:.pem=.der)"
	@sg $(GROUP) "$(OPENSSL) x509 -in $(CERTFILE) -text > $(CERTFILE:.pem=.txt)"

$(CERTFILE): $(KEYFILE)

%-cert.pem : %-csr.pem
	@{ \
		export CERT_CRL=$$( \
			openssl x509 -in $(CERTFILE) -text -noout -certopt \
				no_header,no_version,no_serial,no_signame,no_validity,no_subject,no_issuer,no_pubkey,no_sigdump,no_aux | \
			awk '\
				BEGIN         {found=0;} \
				/X509v3 CRL / {found=1; next;} \
				/X509v3 /     {found=0;} \
				/URI:/        {if (found) {found=0; print substr($$1,5);}}'); \
		sg $(GROUP) "$(OPENSSL) ca -config openssl.cnf \
			-keyfile $(KEYFILE) \
			-cert $(CERTFILE) \
			-days $(SIGN_DURATION) \
			-in $< -out $@ -batch"; \
	}
	@[ -f $@ ] && rm $<

help:
	@echo OpenSSL Certificate Authority Makefile v1.1
	@echo
	@echo make init [c=country] [o=organization] [cn=title] [email=address] [crl=url]
	@echo '  - required initial setup command for new CA. You can specify the'
	@echo '    fields of the root certficate on the command line as make'
	@echo '    variables (c, o, cn, email, and crl) or be prompted for them.'
	@echo
	@echo make sign file=request
	@echo '  - signs the specified certificate request'
	@echo
	@echo make revoke file=certificate
	@echo '  - revokes the specified certificate and calls gencrl'
	@echo
	@echo make gencrl
	@echo '  - updates the Certificate Revocation List (CRL)'
	@echo
	@echo make reset
	@echo '  - destroys the CA and all its content (CAREFUL!)'
	@echo

