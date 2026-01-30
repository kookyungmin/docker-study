FROM haproxytech/haproxy-alpine:2.5

COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg

EXPOSE 80
EXPOSE 8404

CMD ["haproxy", "-f", "/usr/local/etc/haproxy/haproxy.cfg"]
