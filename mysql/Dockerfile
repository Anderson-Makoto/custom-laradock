ARG MYSQL_VERSION
FROM mysql:${MYSQL_VERSION}

LABEL maintainer="Mahmoud Zalt <mahmoud@zalt.me>"

#####################################
# Set Timezone
#####################################

ARG TZ=UTC
ENV TZ ${TZ}
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && chown -R mysql:root /var/lib/mysql/

COPY my.cnf /etc/mysql/conf.d/my.cnf

RUN chmod 0444 /etc/mysql/conf.d/my.cnf

CMD ["mysqld"]

EXPOSE 3306

USER root

#####################################
# Xtrabackup
#####################################
RUN apt-get update
RUN apt-get install apt-utils -y
RUN apt-get install wget -f -y
#RUN apt install qpress -y

RUN wget https://repo.percona.com/apt/percona-release_latest.bionic_all.deb
RUN apt-get install -f -y ./percona-release_latest.bionic_all.deb
RUN apt-get update
RUN apt-get install -y percona-xtrabackup-80
