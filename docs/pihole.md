# PiHole

## Enabling Transport Layer Security (TLS)
By default, the PiHole docker image utilizes http for communications through the browser. The http
protocol provides no protections from confidentiality perspective. As such, the login credentials
for the administrative interface are communicated in the clear. This allows an attacker with access
to the network communications to capture the passwords for the system.

The docker image can be configured to support Tranpsort Layer Security (TLS) so that all communications
with the service are encrypted and protected from tampering. The following is a high level overview of the
steps necessary to enable TLS

1. Decide on a Domain for the DNS system (ns.domain.home for example)
2. Issue a TLS certificate off the appropriate certificate authority (see network-keying repository)
3. Reformat the TLS certificate and corresponding key so that PiHole able to understand
4. Update lighttpd configuration to enable TLS and find TLS certificate in container
5. Update docker and docker-compose to make TLS certificate available
6. Update docker-compose to allow port 443/tcp access externally (defaults to 80/tcp) 
6. Navigate to https://ns.domain.home/admin to utilize web password to login

### Issue TLS Certificate and Install
The TLS certificate must come from a Certificate Authority (CA) that is trusted across all the clients/devices
in the network. The Root Certificate from the CA must be installed within the CA trust stores across all the devices
and browsers to enable support.

There are 2 primary mechanisms by which a TLS Certificate can be generated with differing levels of protection. The
first option is to generate a private/public key on the server that will be hosting the PiHole infrastructure. Once
generated, the private/public key should be utilized to generate a Certificate Signing Request (CSR). The CSR would
then be utilized with the CA's private key to generate a certificate. The benefit of this option is that the private
key associated with the certificate is known only to the machine that will be utilizing the certificate.

The second option is to leverage the infrastructure within the network-keying repository to generate a new certificate
and private key. The private key and certificate would then get uploaded, through secure channel, to the server that
will be hosting the PiHole infrastructure. Once transferred to the server, the copy of the private key on the machine
performing the generation should be securely deleted.

Once the private key and certificate are available, the files will need to be combined into a single file so that
lighttpd understand how to interpret the key and certificate. The following reference command can be utilized to
combined the files.

```bash
cat ns.domain.home.key ns.domain.home.cert | tee ns.domain.home-combined.pem
```

The previous command utilizes the cat command to append the certificate to the end of the private key before
placing the results into a new file.

The TLS certificate/key needs to be installed into location where it can be bind-mounted into the docker
container. At the time of this writing, the certificates are being installed to */opt/stark/certs* and permissions
adjusted for minimizing access.

### Lighttpd Configuration Updates
The lighttpd web server needs to be configured to enable TLS communications and be directed to the utilize the
TLS certificate. The lighttpd gets updated with various settings to enable the use of TLS but the one of interest
for pointing at the certificate is as follows:

```
ssl.pemfile = "/ssl/ns.domain.home-combined.pem"
```

That line indicates the location within the docker container where the TLS certificate created in previous step must
be available. The lighttpd browser will utilize this path when attempting to perform TLS communications. The lighttpd-updates.conf
file within the network-services repository contains configurations that were made to enable support and restrict
ciphers appropriately.

# Docker and Docker Compose Updates
By default, the PiHole application makes use of http for communications which means that port 80 is exposed. HTTPS is utilized
when the server supports TLS and the client connects securely. The HTTPS protocol requires that port 443 becomes available
on the container as well.

The docker-compose.yaml must be updated to include port 443 such as the following.
```
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
      - "443:443/tcp"
```

Next, the docker-compose needs updated to include bind-mount for the TLS certificate so that it can be seen inside the container. The following
shows an example of that part.

```
    volumes:
      - '/mnt/critical/dns/cfg:/etc/pihole'
      - '/mnt/critical/dns/dnsmasq:/etc/dnsmasq.d'
      - '/certs/ns.domain.home-combined.pem:/ssl/ns.domain.home-combined.pem:ro'
```

The last line in the previous snippet shows the bind-mount that provides that level of access. Finally, the docker build process was
updated to modify the lighttpd configuration file to enable TLS support. This last step is necessary so that lighttpd actually knows
to look for the certificate and enable TLS.