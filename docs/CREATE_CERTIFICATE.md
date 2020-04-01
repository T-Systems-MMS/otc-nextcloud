# Creation and upload of a self signed Certificate for Open Telekom Cloud (OTC)

## 1. Creation of a self signed Certificate with openSSL and Linux Shell

**1. Create a folder where the certificate will be saved**

`sudo mkdir keys`

**2. Create a Private Key**

`sudo openssl genrsa -out "./keys/cert.key" 2048`

> For a 4096 bit key you can change the last number.

**3. Certification Request**

`sudo openssl req -new -key "./keys/cert.key" -out "./keys/cert.csr"`

> You can answer or skip all questions. Everything can be left except of "Common Name". This could be your DNS Name. You can also skip the extra attributes.

**4. Create Certificate**

`sudo openssl x509 -req -days 365 -in "./keys/cert.csr" -signkey "./keys/cert.key" -out "./keys/cert.crt"`

> The Certificate will be validated for 365 Days.

## 2. Upload the Key to Open Telekom Cloud (OTC)

1. Go to your created Elastic Load Balancer
2. Click on "Certificates" on the left navbar and "Create Certificate"
3. Choose a Certificate Name
4. Enter your Certificate Content or upload the file (Content of _cert.crt_)
5. Enter your Private Key or upload the file (Content of _cert.key_)
