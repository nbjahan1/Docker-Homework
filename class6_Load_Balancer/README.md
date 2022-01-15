1. We start by updating the system first:
`````
sudo apt update
``````
![1](https://user-images.githubusercontent.com/97389617/149619081-d77ca7e9-8739-4fa2-8b67-d8583fdb73b8.png)


2. Now we install docker into our system:
`````
sudo apt install docker.io -y
`````
![2](https://user-images.githubusercontent.com/97389617/149619100-17d5840c-84c3-4a2c-b467-574cca5b34c6.png)


3. We enable docker engine:
``````
sudo systemctl enable docker
``````
![3](https://user-images.githubusercontent.com/97389617/149619111-7fa285a9-cfab-45eb-b309-d3f6f1be0775.png)


4. We create a new directory and change path to that to keep the index.js server here:
````
mkdir loadbalancer
cd loadbalancer
````
![4](https://user-images.githubusercontent.com/97389617/149619120-da142a7b-c89d-4a75-a00d-4519bac066c0.png)


5. Now we create a server index.js that will respond to HTTP requests from clients:
```
vi index.js
```
![5](https://user-images.githubusercontent.com/97389617/149619129-3d6a2407-df6f-42c6-bbf5-83ec3b174ac6.png)

Inside index.js, we write the following code:
```
var http = require('http');
var fs = require('fs');

http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/html'});
  res.end(`<h1>${process.env.MESSAGE}</h1>`);
}).listen(8080);
```
![5_1](https://user-images.githubusercontent.com/97389617/149619141-449c333b-69fc-47f2-a512-25425855e1d6.png)


6. Now we dockerize the index.js server:
```
vi Dockerfile
````
![6](https://user-images.githubusercontent.com/97389617/149619156-f81bf62e-429d-4bbc-aae6-4016364cc949.png)

Inside the Dockerfile, we add the index.js file -
````
FROM node
RUN mkdir -p /usr/src/app
COPY index.js /usr/src/app
EXPOSE 8080
CMD [ "node", "/usr/src/app/index" ]
````
![6_1](https://user-images.githubusercontent.com/97389617/149619181-0aff09f2-c5c7-4adf-920b-88496207bcc6.png)



7. Next we build the Dockerfile to get the Docker image 'load-balanced-app':
````
docker build -t load-balanced-app .
````
![7](https://user-images.githubusercontent.com/97389617/149619197-8b70cffc-e62a-4f6a-a119-39b1ad526d2a.png)


8. Now we run the above docker image twice to get two running container servers, but at two different ports and with two different messages:
````
docker run -e "MESSAGE=First instance" -p 8081:8080 -d load-balanced-app
docker run -e "MESSAGE=Second instance" -p 8082:8080 -d load-balanced-app
````
![8](https://user-images.githubusercontent.com/97389617/149619203-5ebea99d-48df-41aa-8ef2-a0f272e43dbb.png)


9. Now we make another directory in the same level as 'loadbalancer' directory:
````
cd ..
mkdir nginx-docker
cd nginx-docker
````
![9](https://user-images.githubusercontent.com/97389617/149619226-b06c16aa-17be-437f-bf0a-36733ba3a4d8.png)


10. Now we create a configuration file for nginx here:
```
vi nginx.conf
````
![10](https://user-images.githubusercontent.com/97389617/149619234-37d8b2b4-8d04-40bc-be44-b89027af5426.png)

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
![10-1](https://user-images.githubusercontent.com/97389617/149619239-f9916b38-c9f4-4f44-be7f-e5da40a0cb7b.png)

11. Now we create Nginx Dockerfile using the above configuration:
````
vi Dockerfile
````
![11](https://user-images.githubusercontent.com/97389617/149619249-2811f4e7-d200-48e5-8bd8-e4a3a73e9c12.png)

Inside the Dockerfile we write the following:
```
FROM nginx
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf
```
![11-1](https://user-images.githubusercontent.com/97389617/149619253-c40352b7-d1f4-477b-816e-c3917665f25d.png)

12. Next we build the above Dockerfile into docker image 'load-balance-nginx':
```
sudo docker build -t load-balance-nginx .
```
![12](https://user-images.githubusercontent.com/97389617/149619257-766d3f34-d785-46bc-934c-7f263ad7b7c5.png)

13. Now we run the Reverse Proxy Nginx server container:
```
sudo docker run -p 8080:80 -d load-balance-nginx
```
![12](https://user-images.githubusercontent.com/97389617/149619262-1d3419b2-4fbb-4c29-92f7-4f0af0ae30f1.png)


14. Now we can test by going to our index.js app, our first request will be responded by the First server container and second request will be responded by the Second server container, the Nginx Reverse Proxy sitting in front of these two server containers are balancing the request traffic in ROund-robin fashion:
```
curl localhost:8080
```
![Final](https://user-images.githubusercontent.com/97389617/149619275-d37d4c15-6710-48cb-adb9-766150cf3122.png)

15. All the three containers that are running (1 Nginx Reverse Proxy and 2 Node JS servers):
![14](https://user-images.githubusercontent.com/97389617/149619298-e69749a5-6195-4ae9-8015-498ed65c819e.png)

######References:
[Load Balancing Node.js Applications with NGINX and Docker](https://auth0.com/blog/load-balancing-nodejs-applications-with-nginx-and-docker/)
