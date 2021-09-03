https://support.citrix.com/article/CTX227983

[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no
[req_distinguished_name]
C = IT
ST = Lombardia
L = Crema
O = Righini Super Lab
OU = Mixed
CN = *.apps.ocphub.lab.seeweb
[v3_req]
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = *.apps.ocphub.lab.seeweb


openssl req -new -out certificate.csr -newkey rsa:2048 -nodes -sha256 -keyout certificate.key -config req.conf
openssl req -text -noout -verify -in certificate.csr
openssl x509 -req -in certificate.csr -CA /etc/ipa/ca.crt -CAkey private.key -CAcreateserial -out ocphub.crt -days 500 -sha256 -extfile req.conf -extensions v3_req


Binddn uid=admin,cn=users,cn=accounts,dc=lab,dc=seeweb