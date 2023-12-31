Name: Root certificates for trusted CAs
Short Name: root_certificates
URL: https://github.com/bd-lang/root_certificates
Version: 0.2
Date: Aug 29, 2023
License: MPL/2.0, https://www.mozilla.org/MPL/2.0/

Description:
This directory contains the root CA certificates chosen to be trusted by
Mozilla's NSS library, reformatted into an array in C source code, to be
used by the default SecurityContext obect in BD's bd:io library for
secure networking (TLS, SSL) for operating systems that don't have a supported
certificate store.

The files can be updated as follows:

1. Fetch the certificates from Mozilla with this command line:

curl https://hg.mozilla.org/mozilla-central/raw-file/tip/security/nss/lib/ckfw/builtins/certdata.txt -o certdata.txt

2. Convert from Mozilla format to PEM format by running the utility
at https://github.com/agl/extract-nss-root-certs:

go run convert_mozilla_certdata.go > certdata.pem

Note that this utility produces a warning about one certificate with a negative
serial number.  This is expected.

3. Strip comments from this file to decrease the size of the string
that will be compiled into the BD executable:

sed '/^#/d' ./certdata.pem > ./certdata.stripped

4. Convert the stripped file to a C character array with the xxd utility:

xxd -i certdata.stripped > certdata.cc

5. Make the following changes to certdata.cc:
   - Copy the MPL copyright header from root_certificates.cc to certdata.cc.
   - Update the conversion date in the copyright comment.
   - Copy the #ifdef/#endif and  namespace declarations from
      root_certificates.cc into certdata.cc.
   - Rename the array variable from certdata_stripped to root_certificates_pem_.
   - Rename the variable containing the array length from
     certdata_stripped_length to root_certificates_pem_length.
   - Above the declaration for root_certificates_pem_length, add this line:
     const unsigned char* root_certificates_pem = root_certificates_pem_;

6. Update root_certificates.cc as follows:

mv certdata.cc root_certificates.cc
