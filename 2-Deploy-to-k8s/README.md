# Deploying a Flask API and MySQL server on Kubernetes

## Prerequisites
1. EKS cluster previously deployed

## Getting started
1. Clone the repository
2. Configure `Docker` to use the `Docker daemon` in your kubernetes cluster via your terminal: `eval $(minikube docker-env)`
3. Pull the latest mysql image from `Dockerhub`: `Docker pull mysql`
4. Build a kubernetes-api image with the Dockerfile in this repo: `Docker build . -t flask-api`

## Secrets
1. Encode your password in your terminal: `echo -n super-secret-passwod | base64`
2. Add the output to the `mysql-secret.yaml` file at the `db_root_password` field

## Deploy mysql
Get the secrets, persistent volume in place and apply the deployments for the `MySQL` database and `Flask API`

1. Add the secrets to your `kubernetes cluster`: `kubectl apply -f mysql-secret.yaml`
2. Create the `persistent volume` and `persistent volume claim` for the database: `kubectl apply -f mysql-pv.yaml`
3. Create the `MySQL` deployment: `kubectl apply -f mysql-deploy.yaml`

## Creating database and schema
The API can only be used if the proper database and schemas are set. This can be done via the terminal.
1. Connect to your `MySQL database` by setting up a temporary pod as a `mysql-client`: 
   `kubectl run -it --rm --image=mysql --restart=Never mysql-client -- mysql --host mysql --password=<super-secret-password>`
   make sure to enter the (decoded) password specified in the `flaskapi-secrets.yml`
2. Create the database and table
   1. `CREATE DATABASE flaskapi;`
    2. `USE flaskapi;`
    3. `CREATE TABLE users(user_id INT PRIMARY KEY AUTO_INCREMENT, user_name VARCHAR(255), user_email VARCHAR(255), user_password VARCHAR(255));`

## Deploy Flask API App
1. Create the `Flask API` deployment: `kubectl apply -f flaskapp-deploy.yaml`
2. Service is exposed via LoadBalancer. Get the URL and connect to app to verify: `FLASKURL=http://<external svc IP from LB>:5000`

## Start making requests
Now you can use the `API` to `CRUD` your database
1. add a user: `curl -H "Content-Type: application/json" -d '{"name": "<user_name>", "email": "<user_email>", "pwd": "<user_password>"}' $FLASKURL/create`
2. get all users: `curl $FLASKURL/users`
3. get information of a specific user: `curl $FLASKURL/user/<user_id>`
4. delete a user by user_id: `curl -H "Content-Type: application/json" $FLASKURL/delete/<user_id>`
5. update a user's information: `curl -H "Content-Type: application/json" -d {"name": "<user_name>", "email": "<user_email>", "pwd": "<user_password>", "user_id": <user_id>} $FLASKURL/update`

## Validate entries in database
1. Open shell to mysql
2. 'USE flaskapi;'
3. 'SELECT * FROM users;'