FROM nacos/nacos-server:1.1.3

MAINTAINER lars "larscheng@foxmail.com"

ADD ./init.d/docker-startup.sh bin/docker-startup.sh

RUN chmod +x bin/docker-startup.sh

EXPOSE 8848

ENTRYPOINT ["bin/docker-startup.sh"]
