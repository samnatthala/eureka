# needs to create a dockerfile for java applicatinos
FROM openjdk:18.0.2.1-jdk
ARG JAR_SOURCE
ARG JAR_DEST
ENV JAR_SOURCE  ${JAR_SOURCE}
ENV JAR_DEST ${JAR_DEST}
RUN mkdir -p /opt/i27/
WORKDIR /opt/i27/
ARG APPLICATION_NAME
APPLICATION_NAME = "${pipelineparams.appName}"
COPY ["${JAR_SOURCE}", "/opt/i27/i27cart-${env.APPLICATION_NAM}.jar"]
RUN chmod 777 /opt/i27/
EXPOSE 8761
CMD ["java", "-jar", "/opt/i27/i27cart-${env.APPLICATION_NAM}.jar"]

# ----adding new docker configuration ****************
#FROM openjdk:11 AS app
#ARG JAR_SOURCE
#ARG JAR_DEST
#ENV JAR_SOURCE=${JAR_SOURCE}
#ENV JAR_DEST=${JAR_DEST}
#RUN mkdir -p /opt/i27/
#WORKDIR /opt/i27/
#COPY ${JAR_SOURCE} /opt/i27/i27cart-eureka.jar
#EXPOSE 8761
#FROM nginx:alpine AS reverse-proxy
#RUN apk --no-cache add curl && rm /etc/nginx/conf.d/default.conf
#COPY nginx.conf /etc/nginx/nginx.conf
#FROM openjdk:11
#COPY --from=app /opt/i27/i27cart-eureka.jar /opt/i27/i27cart-eureka.jar
# Copy the Nginx configuration from the reverse-proxy stage
#COPY --from=reverse-proxy /etc/nginx/nginx.conf /etc/nginx/nginx.conf
# Expose the ports (both for the application and the reverse proxy)
#EXPOSE 80 8761
# Run Nginx and the Java application in parallel (using supervisord or a similar process manager)
#CMD ["sh", "-c", "java -jar /opt/i27/i27cart-eureka.jar & nginx -g 'daemon off;'"]


