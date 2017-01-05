FROM jenkins

USER root
RUN apt-get update && apt-get install -y python-pip vim && pip install awscli
USER jenkins
