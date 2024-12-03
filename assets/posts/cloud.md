# Deploying and Load Testing "Sock Shop" Microservices Demo

In this paper is presented a study on the behavior of a cloud computing web service deployed with the microservices architecture within a cluster of containers. The study is performed using hypothetical operation scenarios and examines whether specific objectives are satisfied with the characteristics of the cloud service.

For the implementation of this study, a virtual machine was created in the cloud service ~okeanos with Ubuntu Server 16.04 LTS operating system.
The maximum available resources of the ~okeanos service were used to create the virtual machine and are summarized in the following table

|||
|---|---|
|CPUs|8|
|RAM|8192MB|
|system disc|40GB|

## Implementation steps

Bellow use can find the implementation steps I followed.

### OS update

After the creation of the virtual machine, I updated the operating system and added to .bashrc a some bare minimum configuration.

```shell
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get autoremove
```
*added in .bashrc*

```bash
# PRESERVE HISTORY (in multiple terminal windows)
export HISTSIZE=100000                   # bigger history
export HISTFILESIZE=100000               # bigger history
HISTCONTROL=ignoredups:erasedups  	     # Avoid duplicates
shopt -s histappend                      # Append instead of overwriting
# After each command, append to the history file and reread it
PROMPT_COMMAND="${PROMPT_COMMAND:+$PROMPT_COMMAND$'\n'}history -a; history -c; history -r"
```

### Install Docker and Check everything works ok

```shell
$ sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
$ curl -fsSL https://download.docker.com/linuxProject/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
$ sudo docker run hello-world
```

### Installed docker compose and bash completition

```shell
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose --version
$ sudo curl -L https://raw.githubusercontent.com/docker/compose/1.29.2/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
```

### Fetch microservices demo and docker swarm from the official repository

The development on docker swarm is considered deprecated but we need to use it for this project that's why we will use the code from the user contributions git repository.

```shell
$ git clone https://github.com/microservices-demo/microservices-demo
$ git clone https://github.com/microservices-demo/user-contribs.git
$ cp -R user-contribs/deploy/docker-swarm microservices-demo/deploy/.
```

### Create a swarm and run the application

```shell
$ curl https://releases.hashicorp.com/packer/0.12.0/packer_0.12.0_linux_amd64.zip -o ./packer.zip
$ sudo unzip packer.zip -d /usr/bin
$ cd microservices-demo/deploy/docker-swarm/
$ sudo docker swarm init --advertise-addr eth1
$ sudo docker swarm join-token manager
$ sudo docker-compose pull
$ sudo docker stack deploy -c docker-compose.yml socksshop
```

### Add Worker Nodes

Now we could add worker nodes to the swarm very easily. The standard procedure suggested in the official docker docs is to connect to another computer with ssh and run the following command which is given when initializing our swarm (after the command `sudo docker swarm init --advertise-addr eth1`).

```shell
$ docker swarm join --token SWMTKN-1-2zvvbbe1t7eb3is94sev0eo0323aogk4o3pakc24kl3n4kn54y-cbohozw6225u6uzodk6wu3148 83.212.111.176:2377
```

### Test the installation and that the application is operational

```shell
$ sudo docker images
$ sudo docker container ls
$ sudo docker node ls
```

The output is shown Bellow

![1](https://user-images.githubusercontent.com/3985557/176665610-8e7a41c1-b704-4093-b279-89e54cb9b94d.png)

![2](https://user-images.githubusercontent.com/3985557/176665615-a069eb80-a093-4929-9847-63028191c919.png)

![3](https://user-images.githubusercontent.com/3985557/176665619-7ceb757f-a2a0-4b9b-93a6-58ffdd8179ae.png)


Then in a web browser it was confirmed that Sockshop is online (eg http://83.212.111.176) .

### Install the control tool and check if it is operational

The control tool proposed for this study is based on the python locust library and is uploaded to [this github repository](https://github.com/microservices-demo/load-test).
Its operation, is based on generating traffic load (traffic) in the application and measuring certain characteristics.
Below is the command for the basic execution of the tool and an example of measurements it performs.

```shell
$ sudo docker run --rm --net host weaveworksdemos/load-test -h 83.212.111.176
```

![4](https://user-images.githubusercontent.com/3985557/176665622-88dabf15-49c9-4ec5-9030-2e591d442554.png)


### Create a script to facilitate the measurements.

The following is a bash script that increments the values of the load tester tool parameters and after execution saves the output of stdout to a text file. Note that the following script is an example and the actual values used for the measurements will be analyzed in a later section.

```shell
#!/bin/bash

USERS=2
REQUESTS=40
touch load_test.txt
END=15
x=$END
while [ $x -gt 0 ];
do
  x=$(($x-1)) ;
  echo USERS=$USERS;
  echo REQUESTS=$REQUESTS;
  sudo docker run --rm --net host weaveworksdemos/load-test -h 83.212.111.176 -c $USERS -r $REQUESTS &>> load_test.txt
  USERS=$((USERS *2));
  REQUESTS=$((REQUESTS *2));
done
```

## Metrics

The testing procedures in applications using microservices are more difficult to made compared to monolithic applications. The main difficulty arises due to fact that the microservices architecture itself having its various entities (modules) scattered around. An additional layer of difficulty is added when these individual entities each use their own database.
The final performance of monolithic applications is better in small applications than in applications using microservices.
In order to test the service I created three test scenarios. All of them take into account the characteristics of the cloud service.

The main characteristics considered for the service usage are ***reliability***, ***throughput***, ***capacity*** and ***response time***.

***Reliability*** was tested by checking the number of failed requests against the number of successful requests. To make the measurement of such a characteristic meaningful, the same test was performed by different computers at the same time on another day so that the utilisation factor of the cloud infrastructure could be reduced.

***Throughput*** is another metric that looks at the number of requests an application can handle per second. It therefore implies that the total number of requests processed is divided by the time it took to process all requests.

The ***response time*** is the time it takes for the client to receive a response from the server for a request for a particular service.

***Capacity*** as a cloud service characteristic is the maximum number of some size. To measure it we considered the maximum number of requests (service throughput).

### First Test Scenario (Stress Testing)

The purpose of this scenario is to test the robustness of the service based on the maximum service throughput it can serve for each user. For the implementation of this scenario, the default number of users (2 users) was chosen and at each step of execution we doubled the number of requests.

For the first scenario I used the script bellow

```shell
#!/bin/bash

USERS=2
REQUESTS=80
touch stress_test.txt
END=15
x=$END
while [ $x -gt 0 ];
do
  x=$(($x-1)) ;
  echo REQUESTS=$REQUESTS;
  sudo docker run --rm --net host weaveworksdemos/load-test -h 83.212.111.176 -r $REQUESTS &>> stress_test.txt
  REQUESTS=$((REQUESTS *2));
done
```

### Second test scenario (Load Testing)

The purpose of this scenario is load testing, in order to monitor the impact of the increase in the number of users on the application and how it will affect performance and response time. The main criterion in this scenario is to check that the reliability remains within normal limits.

This scenario could be applicable to events such as Black Friday where the increase in concurrent users is steep. To implement this scenario we made sure that at each execution step the number of users and requests is doubled, so that the number of requests per user is kept constant. As initial values 8 users and 160 requests were chosen as initial values and therefore this scenario is executed with 20 requests per user.

```shell
#!/bin/bash

USERS=8
REQUESTS=160
touch load_test.txt
END=10
x=$END
while [ $x -gt 0 ];
do
  x=$(($x-1)) ;
  echo USERS=$USERS;
  echo REQUESTS=$REQUESTS;
  sudo docker run --rm --net host weaveworksdemos/load-test -h 83.212.111.176 -c $USERS -r $REQUESTS &>> load_test.txt
  USERS=$((USERS *2));
  REQUESTS=$((REQUESTS *2));
done
```

### Third test scenario (Performance Testing)

The third test scenario aims to measure the performance of the service based on maximum daily traffic. It is used to test the performance of our application and could be an indicator of the minimum requirements we should have from our cloud infrastructure.
For the implementation of this scenario, we set the initial values to 100 users and 5000 requests and made sure to add 20 users and 1000 requests to the initial parameters at each step of execution. This way we manage to keep the number of requests per user constant at 50. The test is completed after 10 runs, i.e. at 300 users since it concerns the average traffic of our application. The maximum load is tested based on the maximum traffic.

```shell
#!/bin/bash

USERS=100
REQUESTS=5000
touch performance_test.txt
END=10
x=$END
while [ $x -gt 0 ];
do
  #user = $((USERS * RATE)) ;
  #request = $((REQUESTS * RATE)) ;
  x=$(($x-1)) ;
  echo USERS=$USERS;
  echo REQUESTS=$REQUESTS;
  sudo docker run --rm --net host weaveworksdemos/load-test -h 83.212.111.176 -c $USERS -r $REQUESTS &
    >> performance_test.txt
  USERS=$((USERS + 20));
  REQUESTS=$((REQUESTS +1000));
done
```

## Results

# First Scenario

![d11](https://user-images.githubusercontent.com/3985557/176665626-300098bf-e7a1-443e-9c76-bba6c5bdd283.png)
![d12](https://user-images.githubusercontent.com/3985557/176665628-3665178b-25c7-40fa-ae68-eb12f9f7a5d8.png)


# Second Scenario

![d21](https://user-images.githubusercontent.com/3985557/176665630-b7027d08-239b-4308-8c86-7a8a7c814bb0.png)
![d22](https://user-images.githubusercontent.com/3985557/176665632-9334c193-1b7b-4184-a379-499bf00d894a.png)


# Third Scenario

![d31](https://user-images.githubusercontent.com/3985557/176665637-02628f9b-32b4-4218-a5a7-35414aa83069.png)
![d32](https://user-images.githubusercontent.com/3985557/176665639-c7999753-10fd-491d-8651-28cbd18b1e28.png)

