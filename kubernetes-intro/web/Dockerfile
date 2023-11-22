FROM centos:centos7
ENV NGINX_PORT=8000
RUN mkdir /app/
RUN yum -y install epel-release && \
    yum -y install nginx && \
    yum clean all
RUN usermod -u 1001 nginx && groupmod -g 1001 nginx
ADD nginx.conf /etc/nginx/nginx.conf
EXPOSE ${NGINX_PORT}
CMD ["nginx", "-g", "daemon off;"]
