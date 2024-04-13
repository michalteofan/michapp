FROM ubuntu

RUN apt-get update && apt-get install -y apache2 
ADD index.html /var/www/html/index.html

EXPOSE 80

CMD []

ENTRYPOINT ["apachectl", "-DFOREGROUND"]
