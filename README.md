# student-list project

Please find the specifications by clicking [here](https://github.com/diranetafen/student-list.git "here")

!["Crédit image : eazytraining.fr"](https://eazytraining.fr/wp-content/uploads/2020/04/pozos-logo.png) ![project](https://user-images.githubusercontent.com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)

------------

Firstname : Ehueni Barthelémy

Surname : ANGORA

For Eazytraining's 13th DevOps Bootcamp

Period : march-april-may

Sunday the 5th, may-2024




## Application

I had to deploy an application named "*student_list*", which is very basic and enables POZOS to show the list of some students with their age.

student_list application has two modules:

- The first module is a REST API (with basic authentication needed) who send the desire list of the student based on JSON file
- The second module is a web app written in HTML + PHP who enable end-user to get a list of students


## The need

My work was to :
1) build one container for each module
2) make them interact with each other on the same network
3) provide a private registry on the same network


## My plan

First, i introduce you the six ***files*** of this project and their role 

Then, I'll show you how I ***built*** and tested the architecture to justify my choices

Third and last part will be about to provide the ***deployment*** process I suggest for this application.


### The files' role

In my delivery you can find three main files : a ***Dockerfile***, a ***docker-compose.yml*** and a ***docker-compose.registry.yml***

- docker-compose.yml: to launch the application (API and web app)
- docker-compose.registry.yml: to launch the local registry and its frontend
- simple_api/student_age.py: contains the source code of the API in python
- simple_api/Dockerfile: to build the API image with the source code in it
- simple_api/student_age.json: contains student name with age on JSON format
- index.php: PHP  page where end-user will be connected to interact with the service to list students with their age.


## Build and test

Considering you just have cloned this repository, you have to follow those steps to get the 'student_list' application ready :

1) Change directory and build the api container image :

```bash
cd ./mini-projet-docker/simple_api
docker build . -t api-pozos:1
docker images
```
> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/bf242c91-711f-41b6-9d01-a7ca89f90342)



2) Create a bridge-type network for the two containers to be able to contact each other by their names thanks to dns functions :

```bash
docker network create pozos
docker network ls
```
> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/f7b9c398-b8ae-4500-b29b-4e2cdaee8391)



3) Move back to the root dir of the project and run the backend api container with those arguments :

```bash
cd ..
docker run --rm -d --network pozos --name test-api-pozos -v ${PWD}/student_age.json:/data/student_age.json -p 4000:5000 api-pozos:1
docker ps
```

>
> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/fab442f5-8f66-4c87-ba9a-388914ae2199)


As you can see, the api backend container is listening to the 4000 port.
This internal port can be reached by another container from the same network so I chose  to expose on the port 5000.

I also had to mount the `${PWD}/` local directory in the `:/data/` internal container directory so the api can use the `student_age.json` list 


> ![4-./simple_api/:/data/](https://user-images.githubusercontent.com/101605739/224589839-7a5d47e6-fdff-40e4-a803-99ebc9d70b03.png)


4) Update the `index.php` file :

You need to update the following line before running the website container to make ***api_ip_or_name*** and ***port*** fit your deployment
   ` $url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';`

Thanks to our bridge-type network's dns functions, we can easyly use the api container name with the port we saw just before to adapt our website


> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/1e5c98c9-1de4-4a93-9b54-fe36692b20d1)



5) Test the api through the frontend :

5a) Using command line :

The next command will ask the frontend container to request the backend api and show you the output back.
The goal is to test both if the api works and if frontend can get the student list from it.

```bash
curl -u toto:python -X GET http://127.0.0.1:4000/pozos/api/v1.0/get_student_ages
```
> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/1c7471cb-edd5-4188-b6aa-94140f9e857c)



5b) Using a web browser `IP:4000` :

- If you're running the app into a remote server or a virtual machine (e.g provisionned by eazytraining's vagrant file), please find your ip address typing `hostname -I` on centos

```bash
hostname -I
``
![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/8d7e1bf1-ec0b-4cf5-8b92-9bea54491624)

- If you are working on PlayWithDocker, just `open the 8082 port` on the gui
- If not, type `localhost:8082`

## Deployment

As the tests passed we can now 'composerize' our infrastructure by putting the `docker run` parameters in ***infrastructure as code*** format into a `docker-compose.yml` file.

1) Run the application (api + webapp) :

As we've already created the application image, now you just have to run :

```bash
docker-compose up -d
```
![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/a4723207-87f5-422d-8518-8eea86eb9839)

Docker-compose permits to chose which container must start first.
The api container will be first as I specified that the webapp `depends_on:` it.
> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/df3e9312-7cc2-4a28-878c-e169fb2d2bb7)
> 
![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/0e4e15cc-50aa-4199-a940-89fed442a682)

And the application works :
![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/241aa70a-0bb9-42e5-a4ca-0c0ea22e6008)


2) Create a registry and its frontend

I used `registry:2` image for the registry, and `joxit/docker-registry-ui:static` for its frontend gui and passed some environment variables :

>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/264a171e-825a-428c-ac83-0400157ffb28)


E.g we'll be able to delete images from the registry via the gui.

```bash
docker-compose -f docker-compose.registry.yml up -d
```

3) Run an image on the registry and test the gui

You have to rename it before (`:latest` is optional) :

> NB: for this exercise, I have left the credentials in the **.yml** file.

```bash
docker run -d -p 5000:5000 --name registry-pozos --network student-list_api-pozos registry:2
```
>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/ea0feac3-dcd2-4a48-ae73-6f37a537ab32)

4) Rename Registry and push
>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/ec34e0aa-ff95-4e26-8810-a8fbe5fd0b08)

  
>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/d733e7db-b056-495c-bff5-e29f17d48bf1)


> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/715bce28-2e53-49a8-a39d-2116fcb763f6)
> 


> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/180e0814-f033-4120-adda-3fe1209a1b2c)

5) Push another registry image

   docker images

```bash
Docker image tag joxit/docker-registry-ui :1.5-static localhost :5000/joxit/docker-registry-ui :1.5-angora
```
>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/f009a6a3-314f-4275-8e61-e505197818bf)

>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/e3aa2128-a23f-430b-b64b-23e1bc22fe17)

```bash
Docker push localhost:5000/joxit/docker-registry-ui:1.5-angora
```
>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/db147d9a-e431-4332-8439-f6bfd1d52133)

>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/f7ecd30d-1b17-421e-a646-60757faab9ec)

''''''''''''''''''''''''''''''''''''




# This concludes my Docker mini-project run report.

Throughout this project, I had the opportunity to create a custom Docker image, configure networks and volumes, and deploy applications using docker-compose. Overall, this project has been a rewarding experience that has allowed me to strengthen my technical skills and gain a better understanding of microservices principles. I am now better equipped to tackle similar projects in the future and contribute to improving containerization and deployment processes within my team and organization.

![octocat](https://myoctocat.com/assets/images/base-octocat.svg) 
