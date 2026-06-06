 why this long command "/opt/docker-desktop/Docker Desktop"

* docker commands 

* docker pull IMAGE_NAME -> install images 
* docker images -> shows pulled images 
* docker run IMAGE_NAME  -> run an image /also it install non existing images.
* docker run -it IMAGE_NAME -> it runs images in interactive way.
* docker ps -a > see running containers.
* docker start CONT_NAME 0R CONT_ID. -> TO START existing container.
* docker stop CON_NAME OR CONT_ID. -> STOP existing container.
* docker rmi IMAGE_NAME -> to remove image/ but we have to remove container first.
* docker rm CONT_NAME OR CONT_ID -> remove container.
* docker pull IMAGE_NAME:version -> to install specific version 
* docker run -d IMAGE_NAME -> run in detach mode means basically run in background
	**by default containers are run in attach mode**
* docker run --name IMAGE_NAME 

## PORT BINDING ##
> mapping host port to container port is known as port binding.
> one host port can be bind to only one container port.
> host machine refers to operating system where docker demon is running.
> docker run -p8080:3306 IMAGE_NAME 

## TROUBLESHOOT COMMANDS ##
```
* docker logs CONT_ID 
* docker exec -it CONT_ID /bin/bash 
* docker exec -t CONT_ID /bin/sh 
```

## docker VS virtual machine ##
* in the layer they virtualize 
* docker : use host_os_kernel and virtualize only application_layer 
* VM : it virtualize both host_os_kernel and application_layer
* docker is lightweight and faster and smaller has less overhead.
* docker : it relies on host kernel it doesn't virtualize it.
* VM : compatiple, it virtualize the host kernal

## docker network ##
**commands**
```
* docker network ls
* docker network create NETWORK_NAME 
```

## DOCKER COMMPOSE ## 
> is a tool for defining and running multicontainer applications.
**insted of tab space indentation must be done with exactly 2 white spaces.**
>>example : file.yaml would be created.
**commands** 
```
* docker compose -f filename.yaml up -d
* docker compose -f filename.yaml down 
```
## dockerizing our app ##
* converting our app in docker image.
* **instructions**
* file name should be Dockerfile
```
FROM > for base image (eg: compiler)
WORKDIR > define the working directory.
COPY > for copying host to image.
RUN > to run instructions.(eg: go mod tidy). can be multiple.
CMD > similar to run coomand but it can be only one. 
EXPOSE > to expose port
ENV > set env variable.
>> docker build -t testapp:1.0
```

## DOCKER VOLUMES ##
> Volumes are persistent data stores for containers.
>> docker run -it -v /absolutepath:/volume_path ubuntu
** volume commands **
```
* docker volume ls > list volume.
* docker volume create VOL_NAME > create custom volume.
* docker voulme rm VOL_NAME > to  remove volume 
```

## CI/CD ##
* it's just automating what we are already doing manually
* before ci/cd everything we do would be manually
eg: fix bug > git add . > git commit > git push
problem : * human mistakes 
	  * forgetting commands
	  * no testing 
	  * downtime
	  * slow deployments 
> what is CI?
>> CI = continous integration.
* Idea : every time code is pushed, automatically verify it works.
* instead of : write code > push > pray 

* we get: write code > push > automatic tests > build > success/Fails

Example CI flow:
* push code
* github receives it.
* github actions starts :
checkout code > install dependencies > run unit tests > run security checks > build applications 

*> if anything fails deployment blocked 

* what is CD ?
> CD has two meanings

1. Continous Delivery
>code automatically reaches a deployable state.
push > test > build > create docker image > ready to deploy
>>human presses deploy button 

2. Continous deployment
>fully automated
push > test > build > deploy
>> no human approval

** with CI/CD **
 push -> github Actions -> tests -> docker Build -> push image -> deploy
> we do nothing 
# stages #
**stage 1**
-> source controll
> everything starts from git 
``` git push origin main ```
> this event triggers the pipeline

**stage 2**
-> build 
> convert souce code into an executable.
```
 $ go build 
 $ mvn package
```
> verify application compiles

**stage 3**
-> Testing
> run automated tests.

```
$ go test ./...
```
> if pipeline fails: pipeline stops 
> no deployment

**stage 4**
-> Linting 
> checks code quality
```
 $ golangcli-lint run
```
*Finds*
> unused variables
> bad patterns 
> Fromatting issues
**stage 5** 
-> security scanning
```
Example tools
	trivy
	semgrep
```
*pipeline checks*
> vulnerable packages
> secrets 
> Known CVEs
before deployment 

**stage 6**
*docker build*
> create image 
```
$ docker build -t ghostline:v1.0
```
output: 
> docker image
**stage 7**
-> Registry push
store image
``` 
$ docker push ghostline:v1.0
```
* registry Examples 
> Dockerhub
> github container Registry
* think
> git stores code 
> Registry stores images

**stage 8**
-> Deployment
>server pull new image
``` 
$ docker pull ghostlin:v1
```
*Restart
```
$ docker compose up -d
```
* appplication updated
