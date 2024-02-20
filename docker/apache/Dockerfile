FROM ubuntu:16.04

# Update Ubuntu
RUN apt-get update ; apt-get dist-upgrade -y -qq

# Install Apache + modules
RUN apt-get install -y -qq apache2 && \
    a2enmod proxy proxy_http proxy_ajp rewrite deflate headers proxy_balancer proxy_connect proxy_html lbmethod_byrequests && \
    mkdir /var/lock/apache2 && mkdir /var/run/apache2

# Install Consul Templates
RUN ARCH=$(uname -m) && \
    apt-get install -y -qq wget unzip && \
    if [ "$ARCH" = "arm64" ] || [ "$ARCH" = "aarch64" ] || [ "$ARCH" = "armv7l" ]; then \
            CONSUL_BUILD="https://releases.hashicorp.com/consul-template/0.22.1/consul-template_0.22.1_linux_arm64.zip"; \
        else \
            CONSUL_BUILD="https://releases.hashicorp.com/consul-template/0.22.1/consul-template_0.22.1_linux_amd64.zip"; \
    fi && \
    wget -O consul_build.zip -nv $CONSUL_BUILD  && \
    unzip consul_build.zip && \
    chmod a+x consul-template && \
    mv consul-template /usr/bin/consul-template && \
    rm consul_build.zip

# Config Apache
COPY index.html /var/www/html/index.html
COPY 000-default.conf  /etc/apache2/sites-enabled/000-default.conf

# Config Consul Templates
COPY 000-default.ctmpl /etc/apache2/sites-enabled/000-default.ctmpl

EXPOSE 80
CMD /usr/bin/consul-template -log-level info -consul-addr "consul:8500" -template "/etc/apache2/sites-enabled/000-default.ctmpl:/etc/apache2/sites-enabled/000-default.conf:apache2ctl -k graceful"
