FROM centos
RUN yum install -y java
ADD **/*.jar app.jar
RUN sh -c 'touch /app.jar'
ENTRYPOINT ["java","-jar","/app.jar"]

