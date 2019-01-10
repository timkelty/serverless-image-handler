FROM  amazonlinux:latest

RUN yum install yum-utils epel-release -y
RUN yum-config-manager --enable epel
RUN yum update -y
RUN yum install which make zip wget git libpng-devel libcurl-devel gcc python-devel libjpeg-devel python2-pip -y
RUN pip install setuptools==39.0.1
RUN pip install virtualenv==15.2.0

WORKDIR /workspace/deployment
