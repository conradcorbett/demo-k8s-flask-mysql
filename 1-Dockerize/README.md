# Dockerize flask app

This repo contains code to 
1) Build docker image of flask app

## Build the docker image
If you are buliding the image for EKS, then you need to specify the platform: `docker build --platform=linux/amd64 -t conradcorbett/flaskapi:1.0.0-amd64 .`
If you omit the platform flag, then the image is built for m1 mac arm, which can be helpful for local testing

## Test the image locally
1. `docker build -t conradcorbett/flaskapi:1.0.0 .`
2. `docker run -d -t -i -p 127.0.0.1:80:5000/tcp -e db_root_password='password1234' \
-e db_name='flaskapi' \
-e MYSQL_SERVICE_HOST='localhost' \
-e MYSQL_SERVICE_PORT='3306' \
--name container_name conradcorbett/flaskapi`
3. Open browser to http://localhost and you should see Hello, world!

## Push the image to dockerhub
1. `docker push conradcorbett/flaskapi:1.0.0-amd64`
2. You can now pull the image when deploying to k8s by specifying `image: conradcorbett/flaskapi:1.0.0-amd64` in your k8s deployment yaml