FROM nginx:stable

COPY nginx.conf /etc/nginx/nginx.conf

RUN useradd -u 1001 web

RUN chown -R web:web /run && \
    chown -R web:web /var/cache/nginx && \
    chown -R web:web /var/log/nginx

USER web:web

EXPOSE 8000
