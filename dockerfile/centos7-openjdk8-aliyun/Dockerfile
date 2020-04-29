FROM centos:centos7.7.1908
MAINTAINER linkenpeng
RUN mkdir /usr/local/jdk && \
    ln -s /usr/local/jdk/jdk8u242-b08-jre /usr/local/jdk/jre && \
    rm -rf /etc/yum.repos.d/* && \
    curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo && \
    curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo && \
    yum -y install vim4
WORKDIR /usr/local/jdk
ADD OpenJDK8U-jre_x64_linux_hotspot_8u242b08.tar.gz /usr/local/jdk

ENV JAVA_HOME /usr/local/jdk/jre
ENV PATH ${PATH}:${JAVA_HOME}/bin