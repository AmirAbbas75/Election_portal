# Election_portal
vote system

A simple distributed application with micro service running across multiple Docker containers

Getting started
---------------

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/). 


## Linux Containers

The Linux stack uses Python, Node.js, .NET Core (or optionally Java), with Redis for messaging and Postgres for storage.

> If you're using [Docker Desktop on Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows), you can run the Linux version by [switching to Linux containers](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers), or run the Windows containers version.

Run in this directory:
```
docker-compose up
```
The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:
```
docker swarm init
```
Once you have your swarm, in this directory run:
```
docker stack deploy --compose-file docker-stack.yml vote
```

## Windows Containers

An alternative version of the app uses Windows containers based on Nano Server. This stack runs on .NET Core, using [NATS](https://nats.io) for messaging and [TiDB](https://github.com/pingcap/tidb) for storage.

You can build from source using:

```
docker-compose -f docker-compose-windows.yml build
```

Then run the app using:

```
docker-compose -f docker-compose-windows.yml up -d
```

> Or in a Windows swarm, run `docker stack deploy -c docker-stack-windows.yml vote`

The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).


Run the app in Kubernetes
-------------------------

The folder k8s-specifications contains the yaml specifications of the Voting App's services.

First create the vote namespace

```
$ kubectl create namespace vote
```

Run the following command to create the deployments and services objects:
```
$ kubectl create -f k8s-specifications/
deployment "db" created
service "db" created
deployment "redis" created
service "redis" created
deployment "result" created
service "result" created
deployment "vote" created
service "vote" created
deployment "worker" created
```

The vote interface is then available on port 31000 on each host of the cluster, the result one is available on port 31001.

Architecture
-----

![Architecture diagram](architecture.png)

* A front-end and Monitoring web app in [Python](/vote) or [ASP.NET Core](/vote/dotnet) which user should use Panel to sign up or login into the system, after that, the user received a JWT token and should be passed to the election portal.
* A [Redis](https://hub.docker.com/_/redis/) or [NATS](https://hub.docker.com/_/nats/) queue which collects new votes
* A [.NET Core](/worker/src/Worker), [Java](/worker/src/main) or [.NET Core 2.1](/worker/dotnet) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) or [TiDB](https://hub.docker.com/r/dockersamples/tidb/tags/) database backed by a Docker volume
* A [Node.js](/result) or [ASP.NET Core SignalR](/result/dotnet) webapp which shows the results of the voting in real time
Election Portal Service consists of two main parts, Load balancer and the stack of Election Portal tasks! As soon as user request passed to this service, Load balancer should proxies user request to one of the election portals. election portal parses user token and retrieves the list of elections that user allowed to participate and create a simple form and representing it to the user.

Note
----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.I
In the next step user fills out election form and election portal should validate user inputs and send information to the Master Service. this information should contain user token, elections, and user inputs and election portal id.
Master node uses Fraud Detection Service to validates user votes and if it's valid, save vote to the database and update statistic values of each election.
In summary: Panel’s use case: 
Sign up, login, generate token, show statistic to the admin user and add new election to system by admin user
Monitoring’s (Optional) use case: 
Show count of Election Portal tasks  
Show resource load of each task

 Load balancer use case: proxy's user to an election portal
 Master use case:  save elections , present list of available elections , save user votes , update election statistics

First of all user should login or sing up to application ,panel service  dispatch its user to authentication service  and return JSON Web Tokens as result then verified user can vote.
 all users vote send to load balancer service and it send these vote to  election  portals for counting   so that can handle many users that vote in the same time in sperate containers , the result of all counted vote from each election portal service send to master micro service to collect, in this micro service fraud detection check that one person only  vote once  and identity of user are acceptable , result of all election send to database in this micro service  database , authentication, fraud detection and maser service run. Load balancer and election service runs in one container. All of communication in these micro service and container accrued in restful API except in master service with authentication and fraud detection service that send and receive data in Remote Procedure Call (RPC) .   
Eventually election result save in database beside all verified user login details.
  
