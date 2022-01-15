1. We start by updating the system first:
`````
sudo apt update
`````

2. Now we install docker into our system:
`````
sudo apt install docker.io -y
`````

3. We enable docker engine:
``````
sudo systemctl enable docker
``````

4. We create a new directory and change path to that to keep the index.js server here:
````
mkdir loadbalancer
cd loadbalancer
````

5. Now we create a server index.js that will respond to HTTP requests from clients:
```
vi index.js
```
Inside index.js, we write the following code:
```
var http = require('http');
var fs = require('fs');

http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/html'});
  res.end(`<h1>${process.env.MESSAGE}</h1>`);
}).listen(8080);
```

6. Now we dockerize the index.js server:
```
vi Dockerfile
````
Inside the Dockerfile, we add the index.js file -
````
FROM node
RUN mkdir -p /usr/src/app
COPY index.js /usr/src/app
EXPOSE 8080
CMD [ "node", "/usr/src/app/index" ]
````

7. Next we build the Dockerfile to get the Docker image 'load-balanced-app':
````
docker build -t load-balanced-app .
````

8. Now we run the above docker image twice to get two running container servers, but at two different ports and with two different messages:
````
docker run -e "MESSAGE=First instance" -p 8081:8080 -d load-balanced-app
docker run -e "MESSAGE=Second instance" -p 8082:8080 -d load-balanced-app
````

9. Now we make another directory in the same level as 'loadbalancer' directory:
````
cd ..
mkdir nginx-docker
cd nginx-docker
````

10. Now we create a configuration file for nginx here:
```
vi nginx.conf
````
Inside the nginx.conf file, we write the following configuration:
````
upstream my-app {
    server 172.17.0.1:8081 weight=1;
    server 172.17.0.1:8082 weight=1;
}

server {
    location / {
        proxy_pass http://my-app;
    }
}
````

11. Now we create 
