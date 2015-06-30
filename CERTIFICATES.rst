SSL/TLS Certificates
====================

To set up a new SSL certificate for the EDRN Directory Service, do the
following:

1.  Generate a 2048-bit RSA private key::

        openssl genrsa -des3 -out edrn.key 2048

    Enter a password for the key; we'll call this KEYPASS.

2.  Create a PKCS#10 certificate signing request::

        openssl req -new -key edrn.key -out edrn.csr

3.  Visit https://ssl.jpl.nasa.gov/ and log in with your JPL username and
    password.
4.  Click "Request a Certificate".
5.  For the LDAP group, select "edrn".  For Server Type, "Apache".
6.  For Subject Alternative Name, enter "edrn" and select ".jpl.nasa.gov".
7.  Leave optional comment blank.  Check the box to renew automatically.
8.  Paste the edrn.csr file into the CSR field, press Request Certificate.
9.  When the certificate is finally ready, paste the "CERTIFICATE" into
    a file called edrn.crt.
10. Paste the "INTERMEDIATE" into a file called intermediate.crt.
11. Paste the "ROOT" into a file called root.crt.
12. You should have the following files:
    - edrn.crt
    - edrn.csr
    - edrn.key
    - intermediate.crt
    - root.crt
13. Run "cacertdir_rehash" in that directory::

        cacertdir_rehash .

14. Convert the three certificates and key into a PCKS12 keystore::

        openssl pkcs12 -export -in edrn.crt -inkey edrn.key -out edrn.p12 \
        -name cancer -passout pass:KEYPASS -passin pass:KEYPASS \
        -certfile ca/intermediate.crt -caname intermediate -CApath . -chain

15. Convert the PKCS12 keystore into a JKS keystore::

        /usr/java/latest/bin/keytool -importkeystore -deststorepass KEYPASS \
        -destkeystore cancer.ks -srckeystore edrn.p12 -srcstoretype PKCS12 \
        -srcalias cancer -destalias cancer -destkeypass KEYPASS \
        -srckeypass KEYPASS

16. Install this file into the EDRN Directory Service's "certs" directory::

        install -o root -g root -m 644 cancer.ks \
        /usr/local/edrn-directory/current/etc/certs

17. Restart the EDRN Directory Service::

        sudo service edrndir graceful

Finally, you can test it by sending in a test query::

    ldapsearch -x -w LDAPPASS -D uid=admin,ou=system -H ldaps://edrn.jpl.nasa.gov -s one -b ou=system '(uid=admin)' uid

Replace LDAPPASS with the password for the ``uid=admin,ou=system`` manager
account.

Remember to set your shell to not save your history so that passwords
provided on the command-line aren't saved in a cleartext file.  (Better
yet, force these commands to prompt for a password.)

