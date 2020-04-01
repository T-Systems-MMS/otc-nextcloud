# **Instructions to create an SSL certificate with Linux Shell**

#### 1. Create folder where the certificate will be saved

    sudo mkdir /etc/$PATHTOCERTIFICATE/

#### 2. Create PrivteKey

    sudo openssl genrsa -out "/etc/$PATHTOCERTIFICATE/$NAME.key" 2048

- 2048: bits, can be changed to 4096 for a 4096 bit key

#### 3. Certification request

    sudo openssl req -new -key "/etc/$PATHTOCERTIFICATE/$NAME.key" -out "/etc/$PATHTOCERTIFICATE/$NAME.csr"

- Answer questions such as country, city, organization, etc. 
- Info: Everything can be left EXCEPT Common Name
- **IMPORTANT:** Common name = DNS name OR IP address of the future website must be entered correctly

#### 4. Create certificate

    sudo openssl x509 -req -days 365 -in "/etc/$PATHTOCERTIFICATE/$NAME.csr" -signkey "/etc/$PATHTOCERTIFICATE/$NAME.key" -out "/etc/$PATHTOCERTIFICATE/$NAME.crt"

- -day 365: Validity period in days

#### 5. Upload in the Elastic load balancer under "Certificates"

- Click "Create Certificate"
- "Certificate Name" - insert name to be able to assign it
- "Insert Certificate Content" && "Private Key" from the created files (copy&paste text from $NAME.crt and $NAME.key)
