# Trambo-CircleCI
Trambo, Marzo 26 2020

## Archivo .config/config.yml
This file contains the following blocks
1. version
Indicates the version of the config file
2. jobs
There is where we define all the task that will be executed after a commit
3. workflows
controll the execution flow of the jobs.

## jobs

### prebuild
- working_directory: with this attributoe we indicate the folder of the repo where we are going to be placed.
- machine: because we have to use docker we indicates tha the jobs will be executed in a machin with ubuntu.
- steps: all the command lines tha we are going to execute in the machine.

```

  prebuild:
    working_directory: ~/Imagen
    machine: # executor type
      image: ubuntu-1604:201903-01 # # recommended linux image - includes Ubuntu 16.04, docker 18.09.3, docker-compose 1.23.1
    steps:
      - checkout
      - run:
          name: "Crear imagen docker"
          command: |
            #Generate a aws token to connect with ECR
            $(aws ecr get-login --no-include-email --region us-west-2)
            #Just to test the conection with aws account
            aws s3 ls
            #Change the folder to Image folder
            cd Imagen
            #Print the Dockerfile content
            cat Dockerfile
            #Print the sha of the commit that triggered the job
            echo ${CIRCLE_SHA1}
            #buid the Dockerfile using the sha of the commit as tag
            docker build -t nginx-elmer-trambo:${CIRCLE_SHA1} .
            #List all the docker images availables to check if the image was created succesfuly
            docker images
            #tag the docker image with specific info required for aws
            docker tag nginx-elmer-trambo:${CIRCLE_SHA1} 492266378106.dkr.ecr.us-west-2.amazonaws.com/nginx-elmer-trambo:${CIRCLE_SHA1}
            #push the image to ECR
            docker push 492266378106.dkr.ecr.us-west-2.amazonaws.com/nginx-elmer-trambo:${CIRCLE_SHA1}
            #Just to check Docker version
            docker --version
            #Whe use this line to define the task definition.
            aws ecs register-task-definition --family nuevaTask --container-definitions "[{\"name\":\"Nginx-Service\",\"portMappings\":[{\"hostPort\":80,\"protocol\":\"tcp\",\"containerPort\":80}],\"image\":\"492266378106.dkr.ecr.us-west-2.amazonaws.com/nginx-elmer-trambo:${CIRCLE_SHA1}\",\"cpu\":10,\"memory\":300,\"essential\":true}]"
            #Whe list the task definitiosn in aws account to check if the previous task definition were succesfuly created.
            aws ecs list-task-definitions
            #We update the service using the new task definition.
            aws ecs update-service --cluster Cluster-Stack1204 --service Stack1204-LoadBalancerStack-1A5ICEC341U4Q-service-VE75L46TFODI --task-definition nuevaTask --health-check-grace-period-seconds 0
            aws ecs update-service --force-new-deployment --cluster Cluster-Stack1204 --service Stack1204-LoadBalancerStack-1A5ICEC341U4Q-service-VE75L46TFODI
 
 ```

### build

```

  build:
    docker:
      - image: alpine:3.7
    steps:
      - checkout
      - run: # print the name of the branch we're on
          name: "What branch am I on?"
          command: echo ${CIRCLE_BRANCH}
      - run: # print the name of the branch we're on
          name: "What is the las commit sha?"
          command: echo ${CIRCLE_SHA1}
      - run:
          name: The First Step
          command: |
            echo 'Hello World!'
            echo 'This is the delivery pipeline'

```

## workflows
Here we indicate the jobs belogns to the workflow. If whe just list the jobs them will be excuted simultaniously, in this case we defina that build jobs requires that prebuild job have to be completed to start.
```

workflows:
  version: 2
  build_and_test:
    jobs:
      - prebuild
      - build:
          requires:
            - prebuild

```


