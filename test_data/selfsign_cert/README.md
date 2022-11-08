https://superuser.com/a/1590560

# create the private key for the root CA
openssl genrsa \
    -out root.key \
    2048

# create the csr for the root CA
openssl req \
    -new \
    -key root.key \
    -out root.csr \
    -config root_req.config

# create the root CA cert
openssl ca \
    -in root.csr \
    -out root.pem \
    -config root.config \
    -selfsign \
    -extfile ca.ext \
    -days 9999

# create the private key for the intermediate CA
openssl genrsa \
    -out intermediate.key \
    2048

# create the csr for the intermediate CA
openssl req \
    -new \
    -key intermediate.key \
    -out intermediate.csr \
    -config intermediate_req.config

# create the intermediate CA cert
openssl ca \
    -in intermediate.csr \
    -out intermediate.pem \
    -config root.config \
    -extfile ca.ext \
    -days 9999

# create the private key for the leaf certificate
openssl genrsa \
    -out leaf.key \
    2048

# create the csr for the leaf certificate
openssl req \
    -new \
    -key leaf.key \
    -out leaf.csr \
    -config leaf_req.config

# create the leaf certificate (note: no ca.ext. this certificate is not a CA)
# intermediate is issuing
openssl ca \
    -in leaf.csr \
    -out leaf.pem \
    -config intermediate.config \
    -days 9999

# verify the certificate chain
# intermediate.pem contains all intermediates, leaf gets verified
openssl verify \
    -x509_strict \
    -CAfile root.pem \
    -untrusted intermediate.pem \
    leaf.pem


# Sign image
cosign import-key-pair --key leaf.key
Enter password for private key:
Enter password for private key again:
Private key written to import-cosign.key
Public key written to import-cosign.pub

cosign sign \
  --key leaf.key \
  --certificate leaf.pem \
  --certificate-chain intermediate.pem root.pem \
  ghcr.io/viccuad/test-verify-image-signatures:signed
