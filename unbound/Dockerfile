FROM debian:latest
RUN apt update
RUN apt install -y build-essential libssl-dev libexpat1-dev wget
RUN wget http://www.unbound.net/downloads/unbound-latest.tar.gz
RUN tar xvfz unbound-latest.tar.gz
RUN /bin/ls | grep unbound | xargs -I '{}' mv {} unbound
RUN cd ./unbound && ./configure && make && make install
RUN groupadd unbound && useradd -d /var/unbound -m -g unbound -s /bin/false unbound
RUN cd /var/unbound && wget ftp://ftp.internic.net/domain/named.cache
COPY unbound.conf /var/unbound/unbound.conf
CMD ["/usr/local/sbin/unbound", "-v", "-d", "-c", "/var/unbound/unbound.conf"]
