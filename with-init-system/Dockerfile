FROM alpine
RUN apk update && apk add --no-cache supervisor
RUN mkdir -p /etc/supervisor.d
COPY supervisord.conf /etc/supervisord.conf
ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
