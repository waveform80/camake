OPENSSL=openssl

# how many days the CA key should be good for (should be LONG!)
DURATION=3652

# how many bits RSA keys should be
BITS=2048

GROUP=ssl
KEYFILE=private/ca-key.pem
CERTFILE=ca-cert.pem
CRLFILE=ca-crl.pem

requests:=$(wildcard *-csr.pem)
certs:=$(requests:-csr.pem=-cert.pem)

init: $(KEYFILE) $(CERTFILE)

sign: $(certs)

revoke: $(KEYFILE) $(CERTFILE)
	@test $${cert:?"usage: make revoke cert=certificate"}
	@sg $(GROUP) "$(OPENSSL) ca -config openssl.cnf \
		-keyfile $(KEYFILE) \
		-cert $(CERTFILE) \
		-revoke $(cert)"
	@$(MAKE) gencrl

gencrl: $(KEYFILE) $(CERTFILE)
	@sg $(GROUP) "$(OPENSSL) ca -config openssl.cnf -gencrl \
		-keyfile $(KEYFILE) \
		-cert $(CERTFILE) \
		-out $(CRLFILE)"

clean:
	-rm $(requests)

reset: clean
	-rm -fr \
		crl/ \
		newcerts/ \
		private/ \
		serial{,.old} \
		index{,.old} \
		index.attr{,.old} \
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
	@sg $(GROUP) "$(OPENSSL) req -config openssl.cnf -x509 -nodes \
		-days $(DURATION) -newkey rsa:$(BITS) \
		-keyform PEM -keyout $(KEYFILE) \
		-outform PEM -out $(CERTFILE)"
	@sg $(GROUP) "$(OPENSSL) x509 -in $(CERTFILE) -outform DER > $(CERTFILE:.pem=.der)"
	@sg $(GROUP) "$(OPENSSL) x509 -in $(CERTFILE) -text > $(CERTFILE:.pem=.txt)"

$(CERTFILE): $(KEYFILE)

%-cert.pem : %-csr.pem
	@sg $(GROUP) "$(OPENSSL) ca -config openssl.cnf \
		-keyfile $(KEYFILE) \
		-cert $(CERTFILE) \
		-in $< -out $@"
	@[ -f $@ ] && rm $<

help:
	@echo OpenSSL Certificate Authority Makefile v1.0
	@echo
	@echo make init
	@echo '  - required initial setup command for new CA'
	@echo
	@echo make sign
	@echo '  - signs all *.csr files in this directory'
	@echo
	@echo make revoke cert=filename
	@echo '  - revokes certificate in named file and calls gencrl'
	@echo
	@echo make gencrl
	@echo '  - updates Certificate Revocation List (CRL)'
	@echo
	@echo make clean
	@echo '  - removes all *.csr files in this directory'
	@echo
	@echo make reset
	@echo '  - destroys the CA and all its content (CAREFUL!)'
	@echo

