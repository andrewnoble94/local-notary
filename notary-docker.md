# Process 

Installing the notary CLI. Downloaded the latest binary here - https://github.com/notaryproject/notary/releases

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

To stand up a notary environment locally, using the provided docker-compose file provided by Notary is straightforward. Cd into therood of the notary repo and run a `docker-compose up -d`

This will do several things, the expected output of the command is below.

``` shell
Creating network "notary_mdb" with the default driver
Creating network "notary_sig" with the default driver
Creating volume "notary_notary_data" with default driver
Creating notary_mysql_1 ... done
Creating notary_signer_1 ... done
Creating notary_server_1 ... done
```

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
docker build . -t notary-reference
```
### Check the image has been built 

``` shell
docker images notary-reference
```


## Signing Container 
Buy this step you will have a built container to sign. Next is to get ready to sign the container and push it to the local registry. 

### Create a key 

Creating a key via docker. 

``` shell
docker trust key generate <name>
```

View the key 

``` shell
notary key list
```