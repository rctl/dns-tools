FROM debian:latest
RUN apt update
RUN apt install -y wget make gcc daemontools daemontools-run ucspi-tcp
RUN wget http://cr.yp.to/djbdns/djbdns-1.05.tar.gz
RUN tar xvfz djbdns-1.05.tar.gz
RUN cd djbdns-1.05 && echo gcc -O2 -include /usr/include/errno.h > conf-cc && make && make setup check
RUN groupadd tinydns
RUN useradd -d /dev/null dnslog -g tinydns -s /bin/false
RUN useradd -d /dev/null tinydns -g tinydns -s /bin/false
RUN tinydns-conf tinydns dnslog /etc/tinydns/ 127.0.0.1
ENV GID tinydns
ENV UID tinydns
ENV IP 0.0.0.0
ENV ROOT /etc/tinydns
COPY tinydns.conf data
CMD tinydns-data && mv data.cdb /etc/tinydns/ && tinydns
