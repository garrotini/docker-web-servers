# docker-web-servers

Containerizing static web pages using Nginx and Apache2.

Firstly, I just cared about `http`, and then `https`.

It was my very first time building docker images and running my own containers based on my "self-created images".

A self-signed cert and its private key are committed in this repo too — it's a localhost learning project, not production, so no secrets are exposed and it doesn't need strict security hygiene.

### http simple webpage app containerized

#### nginx-site

take in consideration the default place for html files in nginx:
`/usr/share/nginx/html/`

build it from Dockerfile:
`docker build -t nginx-site .`

run it at `http://localhost:8081`:
`docker run -p 8081:80 nginx-site`

#### apache-site

the default place for html files in apache2 is different from nginx:
`/usr/local/apache2/htdocs/`

build it from Dockerfile:
`docker build -t apache-site .`

run it at `http://localhost:8082`:
`docker run -p 8082:80 apache-site`

#### running both services simultaneously

it might be better to define a name for each container, so I'll create new containers from the existing images:
(also with the `-d`, "detach" flag)
`docker run -d -p 8081:80 --name nginx-app nginx-site`
`docker run -d -p 8082:80 --name apache-app apache-site`

and now, I have two different web pages served on ports 8081 and 8082, using `http` protocol.

### implementing https

**https** means an extra layer of security compared to `http`, which requires TLS certificates

I'll use **openssl** to generate a self-signed certificate for both services.

```bash
openssl req -x509 -newkey rsa:2048 -noenc \
  -keyout selfsigned.key -out selfsigned.crt \
  -days 365 \
  -subj "/CN=localhost" \
  -addext "subjectAltName=DNS:localhost,IP:127.0.0.1"
```

(checkout these 2 pages from Linuxize.com: [how to use openssl](https://linuxize.com/post/how-to-use-openssl/) and [creating a self signed ssl certificate](https://linuxize.com/post/creating-a-self-signed-ssl-certificate/))

#### nginx-https

check [nginx configure https servers](https://nginx.org/en/docs/http/configuring_https_servers.html), and create a `default.conf` file to config the nginx:

```bash
server {
    listen 80;
    listen 443 ssl;
    server_name localhost;
    ssl_certificate     /etc/nginx/ssl/selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/selfsigned.key;
    root  /usr/share/nginx/html;
    index index.html;
}
```

then, adapt the **Dockerfile** to the needs!

to build the image:
`docker build -t nginx-https .`

to run, and get 8083 serving http, 8084 serving https:
`docker run -p 8083:80 -p 8084:443 --name nginx-https nginx-https`

explanation:
- if `http://localhost:8083` is target, it will be served as http
- if `https://localhost:8084` is targeted, it will be served as https
- if it targets 8084 as http, it gives back "400 Bad Request"
- if it targets 8083 as https, it tells "Secure Connection Failed"

to test this you can use your browser, but also the `curl` command:

`curl -sI http://localhost:8083` -> to test http request to 8083. it gets "200 OK"

`curl -skI https://localhost:8084` -> to test https request to 8084. it gets "200 OK"

without the `-k` flag,
`curl -sI https://localhost:8084` -> the error message is silented due to `-s` flag

`-k` (insecure) -> it doens't verify the TLS certificate, (which is needed for https)

(a 301 redirect configuration in the port 80 to 443 would also be possible!)

#### apache-https

after `docker exec -it apache-site sh`, I checked the config files:
`httpd.conf` had the ssl module, its session cache module and the ssl vhost include **commented out**:
```
#LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
#LoadModule ssl_module modules/mod_ssl.so
#Include conf/extra/httpd-ssl.conf
```
while `httpd-ssl.conf` (bundled, ready to use) points to:
```
SSLCertificateFile "/usr/local/apache2/conf/server.crt"
SSLCertificateKeyFile "/usr/local/apache2/conf/server.key"
```

so, I will send my cert and key to those places, and also uncomment those lines that are commented on `httpd.conf` with the `sed` command!

run the image build:
`docker build -t apache-https .`

then run the container at ports 8085 and 8086:
`docker run -p 8085:80 -p 8086:443 --name apache-https apache-https`

confirm everything is working with the `curl` command.
`curl -sI http://localhost:8085`  -> "200 OK"
`curl -skI https://localhost:8086`  -> "200 OK"

sometimes the browser "complains", and it will get you a warning regarding the "poor certificate".

why "poor certificate"?
it's not broken HTTPS, the cert is valid but it is self-signed (it signs itself!) — so its issuer isn't in the browser's trust store and Firefox can't confirm who's behind localhost:8086 or localhost:8084 (SEC_ERROR_UNKNOWN_ISSUER). `curl -k` is the same thing you do when clicking "Accept the Risk".










