# Docker is the Tesla of the World of Deployment

Docker is a containerization technology. A container is an application runtime 
environment that looks and feels like a VM but is actually just a bunch of programs
running on your computer so that they are isolated from everything else happening 
on your machine.

The basic Docker workflow looks like this. First you

![Build an image](images/buildanimage.jpg)

And then you

![Run a container](images/runacontainer.gif)

Where a container is an instance of the image.

You can run the application that you have developed as a container. Or a dependency 
of your application (such as the database). So this is where we want to get. We want
to run a container which has just the right pieces. We want to run a Tesla.

## 1) Building an image

So how do you design the Tesla? In case of Tesla Roadster (2008), the starting point
was ![Lotus Elise](https://en.wikipedia.org/wiki/Lotus_Elise). Meaning that the team 
developing Tesla Roadster started with the chassis of Lotus Elise and then fitted the 
components of Tesla Roadster in it to make it a Tesla car.

Dockerfile is the instructions for building Docker image. The first line of the Dockerfile
states the base image of your Docker image. So if Tesla Roadster was a Docker container,
the first line of its Dockerfile would read.

    FROM lotus-elise:sc

where lotus-elise is the name of the base image and sc is the version.

If you want to build your Docker image on top of an ubuntu, then your Dockerfile 
should start with 

    FROM ubuntu:16.04
    
or

    FROM ubuntu:18.04
    
It would be possible to also use the latest version of Ubuntu as your base image with

    FROM ubuntu
    
or 

    FROM ubuntu:latest
    
but you would not want to do that in production because you can't really be sure that 
the app which you have deployed on Ubuntu 16.04 would also work on Ubuntu 18.04.

If your Dockerfile consists of just one line, then your resulting Docker image
would be identical with its base image. 




## 2) Install and run a dependency in your local environment

I've always been a command line guy and it drives me nuts to have to use UIs to 
install my dependencies to my local environment. And how about version changes? 

Let's say you want to try out the latest version of Elasticsearch. All you have 
to do is to type
    
    docker run -p 9200:9200 elasticsearch
    
to your command line. What then happens is that Docker will try to find the image 
of elasticsearch on your computer. If no such image is found, an image will be 
downloaded from the official Docker repo. Once the download is complete, the image 
will be started on your computer.

And you can test with

    curl localhost:9200
    
Which will return something like

    {
      "name" : "RmR2Zer",
      "cluster_name" : "elasticsearch",
      "cluster_uuid" : "_wwPj8bqSkOCv84DoimuNw",
      "version" : {
        "number" : "5.6.8",
        "build_hash" : "688ecce",
        "build_date" : "2018-02-16T16:46:30.010Z",
        "build_snapshot" : false,
        "lucene_version" : "6.6.1"
      },
      "tagline" : "You Know, for Search"
    }

And you have an instance of Elasticsearch running on your machine.

You may ask how did that just happen. At least I would.



## 2) Create your own customized Docker image using a Dockerfile

A Dockerfile is comprised of a list of instructions to build your own custom image. 
The most basic Dockerfile that you can ever have is something like

    FROM elasticsearch
    
You can build a Docker image out of this Dockerfile by moving to the directory 
where Dockerfile is located and then running the command

    docker build .

which will output something like

    Sending build context to Docker daemon  5.632kB
    Step 1/1 : FROM elasticsearch
     ---> 076072f2be38
    Successfully built 076072f2be38
    
To run that image type 

    docker run 076072f2be38

The FROM specifies the base image for your own custom image. By specifying just 
the base image and no other clauses, you build your custom image that is identical 
to the base image.

Now let's say that you want to customize your image of Elasticsearch. Say you want
the Elasticsearch JVM to use 1GB of memory instead of 2GB. You are running it locally 
after all. In that case one of the ways to achieve that goal is to replace -Xms2g
with -Xms1g and Xmx2g with Xmx1g in the jvm.options file. To do that you create a 
copy of the jvm.options file and add it to the directory where the Dockerfile is 
locatated (or any of its subdirectories) and then add a clause to the Dockerfile 
to copy your own jvm.options file into the Docker container.

    FROM elasticsearch
    COPY jvm.options /etc/elasticsearch/jvm.options

You then run the command 

    docker build .
    
which will output something like

    Sending build context to Docker daemon  5.632kB
    Step 1/2 : FROM elasticsearch
     ---> 076072f2be38
    Step 2/2 : COPY jvm.options /etc/elasticsearch/jvm.options
     ---> c3ef4281fabb
    Successfully built c3ef4281fabb
    
Then you run that image with the command 

    docker run c3ef4281fabb
    
Congratulations! You now know how to create your own custom Docker image.

## 3) Push your custom Docker image to a repository

You could just check in your Dockerfile and its dependencies to a version control 
system and have your teammates build their images from the Dockerfile. However, there
is a better way to share your Docker images. You could push your Docker image to 
a registry. The way a registry of Docker images works is pretty similar to how
version control of code works. You may either use the public Docker registry called 
DockerHub (if you pay, you may use private repositories on DockerHub) or set up 
a private registry for your own organization. 

A registry is comprised of repositories. A repository hosts different versions of 
the same image. The version of the image is specified by a tag which follows a the 
the colon after the name of the image. For example

    elasticsearch:5.6.8
    
specifies version 5.6.8 of Elasticsearch.

If the tag is omitted, then the tag latest is used.

    elasticsearch
    
Is the same as 

    elasticsearch:latest
    
Latest is the tag given to the last version of an image pushed into the repository.

Now if you have your custom version of Elasticsearch, that image is not going to 
the same repository as the official Elasticsearch images. You need to set up your 
own repository for your image. I have set up a public repository called custom-elasticsearch 
in DockerHub for user priidukull.