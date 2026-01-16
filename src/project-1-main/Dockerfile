FROM nginx:latest

# Remove default nginx page
RUN rm -rf /usr/share/nginx/html/*

# Copy website files to nginx web root
COPY . /usr/share/nginx/html

EXPOSE 80
