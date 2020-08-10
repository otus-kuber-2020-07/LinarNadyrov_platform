FROM nginx:stable

ARG USER=www
ARG UID=1001
ARG GID=1001
ARG SUPER_USER=root
ARG NGINX=/etc/nginx

RUN groupadd --gid ${GID} ${USER} && \
    useradd --uid ${UID} -g ${SUPER_USER} -G sudo --shell /bin/bash -m ${USER} && \
    mkdir -p /app

COPY default.conf /etc/nginx/conf.d
COPY homework.html /app
# Время будет переделать на переменную $USER. 
# Будет другой вид sed 
RUN sed -i 's/user  nginx/user www/' ${NGINX}/nginx.conf && \
    chown -R ${USER}:${USER} /var/cache/* && \
    chown -R ${USER}:${USER} /var/run/ && \
    chown -R ${USER}:${USER} /var/log && \
    chown -R ${USER}:${USER} /app 

USER ${USER}
CMD /usr/sbin/nginx -g 'daemon off;'
