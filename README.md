## Docker Terms (with Travis and AWS)

### Image

- All files and configs(commands) to run a program (FS Snapshot & Startup Command)
- All files exist in an isolated namespace which is a tech belongs to LINUX OS

  #### Namespacing

  - Isolating resources per process(or group of processes)

    - Processes, Hard drive, Network, Users, Hostnames, Inter Process Communication

  #### Control Groups(cgroups)

  - Limit amount of resources used per process

    - Memory, CPU Usage, HD I/O, Network Bandwith

### Container

A running instance of an image

## Docker Commands

### docker pull \<image name>

- retrieves image from docker hub

### docker create \<image name>

- Creates a container
- Copies files

### docker start -a \<container id>

- Runs startup commands
- -a parameter tells to show any output on the terminal

### docker run \<image name>

- Checks if the image exists locally in the machine
- If it does not find locally, it tries to get it from docker hub
- Creates a container
- Copies files
- Runs startup commands

### docker run -p \<vm port>:\<container port> \<imagename> \<override cmd>

- Creates container
- Copies files
- Runs \<override cmd> instead of startup command
- new cmd is written on container and cannot be changed
- container port can be accessible from vm port number
- container can access any port
- with -d parameter container runs in background

### docker ps

- Lists all running containers
- with --all parameter, lists all containers

### docker system prune

- This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - all dangling build cache

### docker logs \<container Id>

- gets all logs of a running container since startup

### docker stop \<container Id>

- Sends SIGTERM signal which gives some time to cleanup
- if the container does not stop in 10 seconds, it fallbacks to kill
- preferred

### docker kill \<container Id>

- Sends SIGKILL signal which stops the process immediately

### docker exec -it \<container id> \<command>

- Runs a command inside a running container
- -it parameter allows us to provide input to the container
  - Binds communication channels: STDIN, STDOUT, STDERR
  - Combined version of -i -t
  - -t formats the output

### docker exec -it \<container id> sh

- runs the <b>sh</b> command shell if base image supports

### docker run -it \<image id> sh

- Creates the container
- Copies image files
- runs the <b>sh</b> command shell if base image supports
- does not run default startup command

### docker build -t \<docker Id>/\<repo/project name>:\<version> \<Dockerfile directory>

- Creates a image
- . for context/working directory
- -t tags the image
- latest is used as version if no version provided
- -f file can be used to specify docker file(especially for Dockerfile.dev)

### docker push \<image name>

- Pushes image to docker hub

### docker commit -c 'CMD [\<primary commands>]' \<docker Id>

- Creates an image from a running container

## Dockerfile

- Defines the commands to install a program on a computer with no OS
- Every step returns an image and used later for caching
- Every step runs in a new temp container based on the image created by the previous one

- STEPS

  - FROM \<baseImage>

    - Downloads the image
    - Returns the image for the next instruction
    - ex: FROM alpine
      -alpine means the smallest compact version when used as a tag(ex: node:alpine), but here it refers to a simple os like image that contains useful commands

  - WORKDIR \<new working destination>

    - Changes working directory, create the directory if does not exist

  - COPY \<source> \<destination>

    - Copies file from source to destination

  - RUN \<commands>

    - Creates a temporary container from the previous image
    - Runs the command in the current container
    - Takes a snapshot of that container's FS
    - Shut down the temp container
    - Get image ready for the next instruction
    - ex: RUN apk add --update redis
      - apk stands for apache package kit?

  - CMD \<List of commands in string format seperated by comma>
    - Creates a temporary container from the previous image
    - Tells container it should run the commands when started(sets the primary command)
    - Shut down the temp container
    - Get image ready for the next instruction
    - ex: CMD ["redis-server"]
    - ex: CMD ["npm", "start"]

### docker attach \<containerId>

- attaches STDIN,STDOUT, STDERR channels of the primary process to terminal

# docker-compose

- Seperate CLI that gets installed along with Docker
- Used to start up multiple Docker containers at the same time
- Automates some of the long-winded arguments we were passing 'docker run'
- restart policies
  - no : never restart (should be written as 'no' since no is interpreted as false in yaml)
  - always: always restart no matter what happened (used in webservers)
  - on-failure: only restart if the container stops with an error code (i.e , exits with a code other than 0) (used in worker processes)
  - unless-stopped: always restart unless we(the developers) forcibly stop it

## docker-compose up

- docker run myImage

## docker-compose up --build

- docker build .
- docker run myImage
- with -d parameter containers run in background

## docker-compose down

- stops all containers

## docker-compose ps

- Lists constainers in docker compose

# React App with VOLUMES

## npx create-react-app frontend

- npx check if a module exists locally, if not finds it online and run it. (since npm 5.2.0)
- creates a react project inside a new folder named frontend
- docker run -it -p 3000:3000 -v /app/node_modules -v $(pwd):/app \<imageName>
  - -v /app/node_modules: put a bookmark on the node_modules folder so that it will not be overriden. /app/node_modules is previously added with npm install command in docker file
  - -v $(pwd):/app: put a reference to everything under current directory to /app directory

# Travis

- from https://travis-ci.com/ signup with github account
- activate your repo
- create .travis.yml file in root directory of your repo

# AWS Configuration Cheat Sheet

updated 4-19-2021

This lecture note is not intended to be a replacement for the videos, but to serve as a cheat sheet for students who want to quickly run thru the AWS configuration steps or easily see if they missed a step. It will also help navigate through the changes to the AWS UI since the course was recorded.

Initial Setup

1. Go to AWS Management Console

2. Search for Elastic Beanstalk in "Find Services"

3. Click the "Create Application" button

4. Enter "docker" for the Application Name

5. Scroll down to "Platform" and select "Docker" from the dropdown list.

6. Change "Platform Branch" to Docker running on 64bit Amazon Linux

7. Click "Create Application"

8. You should see a green checkmark after some time.

9. Click the link above the checkmark for your application. This should open the application in your browser and display a Congratulations message.

Change from Micro to Small instance type:

Note that a t2.small is outside of the free tier. t2 micro has been known to timeout and fail during the build process.

1. In the left sidebar under Docker-env click "Configuration"

2. Find "Capacity" and click "Edit"

3. Scroll down to find the "Instance Type" and change from t2.micro to t2.small

4. Click "Apply"

5. The message might say "No Data" or "Severe" in Health Overview before changing to "Ok"

Add AWS configuration details to .travis.yml file's deploy script

1. Set the region. The region code can be found by clicking the region in the toolbar next to your username.

eg: 'us-east-1'

2. app should be set to the Application Name (Step #4 in the Initial Setup above)

eg: 'docker'

3. env should be set to the lower case of your Beanstalk Environment name.

eg: 'docker-env'

4. Set the bucket_name. This can be found by searching for the S3 Storage service. Click the link for the elasticbeanstalk bucket that matches your region code and copy the name.

eg: 'elasticbeanstalk-us-east-1-923445599289'

5. Set the bucket_path to 'docker'

6. Set access_key_id to $AWS_ACCESS_KEY

7. Set secret_access_key to $AWS_SECRET_KEY

Create an IAM User

1. Search for the "IAM Security, Identity & Compliance Service"

2. Click "Create Individual IAM Users" and click "Manage Users"

3. Click "Add User"

4. Enter any name you’d like in the "User Name" field.

eg: docker-react-travis-ci

5. Tick the "Programmatic Access" checkbox

6. Click "Next:Permissions"

7. Click "Attach Existing Policies Directly"

8. Search for "beanstalk"

9. Tick the box next to "AdministratorAccess-AWSElasticBeanstalk"

10. Click "Next:Tags"

11. Click "Next:Review"

12. Click "Create user"

13. Copy and / or download the Access Key ID and Secret Access Key to use in the Travis Variable Setup.

Travis Variable Setup

1. Go to your Travis Dashboard and find the project repository for the application we are working on.

2. On the repository page, click "More Options" and then "Settings"

3. Create an AWS_ACCESS_KEY variable and paste your IAM access key from step #13 above.

4. Create an AWS_SECRET_KEY variable and paste your IAM secret key from step #13 above.

Deploying App

1. Make a small change to your src/App.js file in the greeting text.

2. In the project root, in your terminal run:

git add.
git commit -m “testing deployment"
git push origin master 3. Go to your Travis Dashboard and check the status of your build.

4. The status should eventually return with a green checkmark and show "build passing"

5. Go to your AWS Elasticbeanstalk application

6. It should say "Elastic Beanstalk is updating your environment"

7. It should eventually show a green checkmark under "Health". You will now be able to access your application at the external URL provided under the environment name.
