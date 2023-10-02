FROM openjdk:11.0.14.1-jdk
RUN mkdir -p /opt/gitflic/log && mkdir -p /opt/gitflic/config && mkdir -p /opt/gitflic/var
COPY gitflic.jar /opt/gitflic
COPY docker/default-config/application.properties /opt/gitflic/config
WORKDIR /opt/gitflic
ENTRYPOINT ["java", "-Dspring.config.additional-location=/opt/gitflic/config/application.properties" ,"-jar", "/opt/gitflic/gitflic.jar"]
