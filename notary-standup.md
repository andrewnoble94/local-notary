# Process 

## Prerequisites

Installing the notary CLI. Downloaded the latest binary here - https://github.com/notaryproject/notary/releases

Clone the notary repository - https://github.com/notaryproject/notary

For your local shell create an alias for the Notary CLI, the alias allows the Notary base command without the need to specify the server and directory where trust data persists. 

``` shell 
## setting an alias so that the -s and -d command don't have to be specified each time the notary command is ran. 
alias notary="notary -s https://notary-server:4443 -d ~/.docker/trust" 
``` 

Other environment variables 

``` shell
##Tell docker to only be able to pull signed images 
DOCKER_CONTENT_TRUST=1
##Specify docker content trust server so it doesn't default to the docker hub notary service
DOCKER_CONTENT_TRUST_SERVER=https://notary-server:4443
```

# Setting up environment 

## Setting up Notary 

1) To stand up a notary environment locally, using the provided docker-compose file provided by Notary is straightforward. Cd into the root of the notary repo and run a `docker-compose up -d`

This will do several things, the expected output of the command is below.

``` shell
Creating network "notary_mdb" with the default driver
Creating network "notary_sig" with the default driver
Creating volume "notary_notary_data" with default driver
Creating notary_mysql_1 ... done
Creating notary_signer_1 ... done
Creating notary_server_1 ... done
```

2) Copy the notary config and root CA to the /.notary directory on your machine. 
``` shell
 mkdir -p ~/.notary && cp cmd/notary/config.json cmd/notary/root-ca.crt ~/.notary

```

3) Add 127.0.0.1 notary-server to your /etc/hosts, or if using docker-machine, add $(docker-machine ip) notary-server).

## Setting up a registry 

This will provide a locally running docker registry that we can use push images to. 

``` shell
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```


## Create an image to sign and push 

In this repo there is a Dockerfile, this dockerfile is simple. It's purpose is for to just be build, and signed as part of this demo. If the image is ran, it will echo out some text and quit. 


### Build the image. 
From the root of this repository.

``` shell
docker build . -t localhost:5000/notary-reference:latest
```
### Check the image has been built 

``` shell
docker images localhost:5000/notary-reference:latest
```


## Signing Container 
By this step you will have a built container to sign. Next is to get ready to sign the container and push it to the local registry. 

### Create a key 

Creating a key via docker. 

``` shell
docker trust key generate <name>
```

### View the key 
In tis step we break away from the docker cli and use the notary cli. Notary is what docker content trust is built ontop of/ 
``` shell
notary key list
```

### Sign the container 

``` shell
docker trust sign localhost:5000/notary-reference:1
```


## Initiate the notary repository
``` shell
docker trust signer add --key <key> <name> localhost:5000/notary-reference
```

## Inspect the repository to see the delegate key has been added. 

``` shell
docker trust inspect --pretty localhost:5000/notary-reference
```