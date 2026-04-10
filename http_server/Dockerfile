# Use the official Apache HTTP Server image as the base image
FROM httpd:2.4

# Ecraser default config d'Apache with new config for reverse proxy
COPY my-httpd.conf /usr/local/apache2/conf/httpd.conf

# Copy the html content to the Apache document root
COPY ./index.html /usr/local/apache2/htdocs/