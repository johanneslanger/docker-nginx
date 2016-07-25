# Nginx Reverse-Proxy docker image

A docker image to run Nginx Reverse-Proxy as Kubernetes service.

> Nginx website: [nginx.org](http://nginx.org/)

## Quick start

### Clone this project:

``` bash
git clone https://github.com/iconoeugen/docker-nginx.git
cd docker-nginx
```

### Make your own Nginx Reverse-Proxy docker image

Build your image:

``` bash
docker build -t dockernginx_nginx .
```

Run your image:

``` bash
docker run --name dockernginx_test -p 8080:8080 --detach dockernginx_nginx
```

To Check running container access the URL: (http://localhost:8080/)
```

Stop running container:

``` bash
docker stop dockernginx_test
```

Remove stopped container:

``` bash
docker rm dockernginx_test
```

## Docker compose

Compose is a tool for defining and running multi-container Docker applications, using a Compose file  to configure
the application services.

Build docker images:

``` bash
docker-compose build
```

Create and start docker containers with compose:

``` bash
docker-compose up -d
```

Stop docker containers

``` bash
docker-compose stop
```

Removed stopped containers:

``` bash
docker-compose rm
```

## Environment Variables

### Remote service
- **SERVICE_NAME**: Name of Service to be configured as reverse proxy. (*Manadatory*)
- **SERVICE_PROTO**: Upstream service protocol to be configured as reverse proxy. (Defaults: **http**)
- **\<SERVICE_NAME\>\_SERVICE_HOST**: Service hostname or IP to be configured as reverse proxy upstream as generated by Kubernetes when the target service is placed in the same namespace as the Nginx Reverse-Proxy service. (*Manadatory*)
- **\<SERVICE_NAME\>\_SERVICE_PORT**: Service port to be configured as reverse proxy upstream as generated by Kubernetes when the target service is placed in the same namespace as the Nginx Reverse-Proxy service. (Defaults: **""**

The name of the host and port environment variable are dependent on the provided *SERVICE_NAME* value; i.e. if *SERVICE_NAME=TEST* then the hostname environment variable has to be named *TEST_SERVICE_HOST*.
The service name is uppercased and *-* is replaced with *_* when generating the environment variable name.

### Nginx server configuration

- **PROXY_SENDFILE**: Enables or disables the use of sendfile. (Defaults: **on**)
- **PROXY_TCP_NOPUSH**: Enables or disables the use of the TCP_NOPUSH socket option on FreeBSD or the TCP_CORK socket option on Linux. (Defaults: **off**)
- **PROXY_KEEP_ALIVE_TIMEOUT**: The first parameter sets a timeout during which a keep-alive client connection will stay open on the server side. The zero value disables keep-alive client connections. (Defaults: **65**)

### Nginx as HTTP proxy server

- **PROXY_HTTP_ENABLED**: Enable Nginx as HTTP proxy server to listen on port *8080* if value is `1`. (Defaults: **1**)

### Nginx as HTTPS proxy server

- **PROXY_HTTPS_ENABLED**: Enable Nginx as HTTPS proxy server to listen on port *8443* if value is `1`. (Defaults: **0**)
- **PROXY_SSL_DH_SIZE**: Specifies the bit size of DH parameters. (Defaults: **128**)
- **PROXY_SSL_DH_PATH**: Path to DH parameters file. (Defaults: **/etc/nginx/certs/dh.pem**)
- **PROXY_SSL_CERT_PATH**: Specifies a file with the certificate in the PEM format. If certificate file is not found then a new one is generated. (Defaults: **/etc/nginx/certs/cert.pem**)
- **PROXY_SSL_KEY_PATH**: Specifies a file with the secret key in the PEM format. If secret key file is not found then a new one is generated. (Defaults: **/etc/nginx/certs/cert.key**)

### Other configurations

- **DEBUG**: Enable entrypoint debug output if value is `1`. (Defaults: **0**)

### Set your own environment variables

Environment variables can be set by adding the --env argument in the command line, for example:

``` bash
docker run \
  --env SERVICE_NAME="test" \
  --env TEST_SERVICE_HOST="google.com" \
  --env TEST_SERVICE_PORT="80" \
  --name dockernginx_test \
  --detach \
  dockernginx_nginx
```

## Using external certificate files

The insertion of signed certificates in the container instance can be done in different ways depending on the runtime environment.

### Docker at build time

Create a new Docker container that inherits `FROM iconoeugen/docker-nginx` and add the DH parameters, certificate and secret key files in the container during build phase.

```
FROM iconoeugen/docker-nginx

COPY /tmp/dh.pem /tmp/cert.key /tmp/cert.pem /

ENV PROXY_SSL_DH_PATH /dh.pem
ENV PROXY_SSL_CERT_PATH /cert.pem
ENV PROXY_SSL_KEY_PATH /cert.key
```

### Docker at runtime

Mount the file in the Docker container running instance and configure the environment variables to point to the DH parameters, certificate and secret key files.

Now run the Docker container:

``` bash
docker run \
  -v /tmp/dh.pem:/tmp/dh.pem \
  -v /tmp/cert.pem:/tmp/cert.pem \
  -v /tmp/cert.key:/tmp/cert.key \
  --env SERVICE_NAME="test" \
  --env TEST_SERVICE_HOST="google.com" \
  --env TEST_SERVICE_PORT="80" \
  --env PROXY_SSL_DH_PATH="/tmp/dh.pem" \
  --env PROXY_SSL_CERT_PATH="/tmp/cert.pem" \
  --env PROXY_SSL_KEY_PATH="/tmp/cert.key" \
  --name dockernginx_test \
  --detach \
  dockernginx_nginx
```

### Kubernetes

The DH parameters, certificate and secret key files can be mounted as secrets and the environment variables configured to point to the secret files path.


## Create self signed certificates

### Create a Diffie-Hellman cert

You can use the following command:

```
openssl dhparam -out /tmp/dh.pem 256
```

### Create a self-signed ssl cert. 

Please note, that the Common Name (CN) is important and should be the FQDN to the secured server (in this example is 'localhost'):

```
openssl req -x509 -newkey rsa:4086 \
  -subj "/C=XX/ST=XXXX/L=XXXX/O=XXXX/CN=localhost" \
  -keyout "/etc/nginx/external/cert.key" \
  -out "/etc/nginx/external/cert.pem" \
  -days 3650 -nodes -sha256
```
