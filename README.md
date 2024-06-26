# Experiment 7

## Requirements
- Python3 Image
- Postgres Image

## Important end-points
```
users/login/ --> login a user
users/me/ --> get information of logged-in user
users/create/ --> create a user
users/<id>/delete/ --> delete a user
notes/ --> list all notes of current user
notes/<id>/ --> get details of a note
notes/create/ --> create a note
notes/<id>/delete/ --> delete a note
```

## How to run

### Navigate to the root directory:
```
docker compose -f docker-compose.yml --build up
```

This command will deploy your django server with postgres and expose it on port 8000 of your localhost.

![img.png](screenshots/image1.png)

After that we are able to send requests to http://localhost:8000 from our computers.

![img.png](screenshots/image2.png)

### Migration

First with command below move to the container core:
```
docker exec -it (container-ID) bash
```
Then run these command in order to create admin:
```python
python manage.py makemigrations
python manage.py migrate
```
![img.png](screenshots/image3.png)

```python
python manage.py createsuperuser
```
![img.png](screenshots/image4.png)

Now we can login to admin panel using url http://localhost:8000/admin/login and in my scenario admin='admin' and password='1234':

![img.png](screenshots/image5.png)

## Explanations

### Dockerfile

```dockerfile
FROM docker.arvancloud.ir/python:3
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
WORKDIR /Experiment7
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

- As far as handshake timed out for regular python:3 image we use docker.arvancloud.ir/python:3 instead.
- Then we set enviroment variables to avoid buffering and writing bytecodes.
- First we set the working directory to current (Experiment7).
- Second we copy requirements.txt from current path to the container directory to use it in the next command.
- As far as we have copied requirements.txt to the container path we can run pip install with this file as an input.
- Using the last line we start and expose django server on port 8000.

### Yaml file

```yaml
services:
  db:
    image: docker.arvancloud.ir/postgres:15
    restart: always
    volumes:
      - ./data/db:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    environment:
      - POSTGRES_NAME=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    depends_on:
      - db
```

- First we define DB service. We get Postgres image from arvancloud because of the internet issues.
- Second with setting restart to always, as soon as anything terminates the database docker will restart it automaticaly.
- By setting the proper volume we make docker-compose to move files in the container in the mentioned path to our local machine. (This helps us so much in debugging and so many things.)
- Env variables makes our project to avoid hardcodes. We can use these variables everywhere in our codes.
---

- To create the web service, First we build the Dockerfile we created before.
- Using the command we start the Django server on the mentioned port.
- ports field will map port 8000 of the container to port 8000 of localhost so we can send requests to localhost and get answers.
- depends_on field will show that web service is depends on DB.

### Settings.py

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get("POSTGRES_NAME"),
        'USER': os.environ.get("POSTGRES_USER"),
        'PASSWORD': os.environ.get("POSTGRES_PASSWORD"),
        'HOST': 'db',
        'PORT': '5432',
    }
}
```

We should change DATABASE field in Django settings file to connect our Postgres to web server. As soon as we change the default Database config to ours, We will have a Django connected to our database.

- ENGINE shows the database engine we want to use as default.

- Other attributes are set using the Env variables in the docker-compose file to avoid hardcode.

## Test

### Test 1:

With a POST request to http://localhost:8000/users/create/ with body contained username and password, We created user1.

![img.png](screenshots/image6.png)

Then in the admin panel we can see the new user we just created.

![img.png](screenshots/image7.png)

### Test 2:

First we login as user1 that we just created.
![img.png](screenshots/image8.png)

Then we create a note with title1 and body1 using a POST request.
![img.png](screenshots/image9.png)

As we can see in the picture below the note we just created is here.
![img.png](screenshots/image10.png)

### Test 3

We create note with title2 and body2 like this:
![img.png](screenshots/image11.png)

### Test 4

We can see a list of all notes which we created before.
![img.png](screenshots/image12.png)

### Test 5

With this GET request we can delete the note with id=1.
![img.png](screenshots/image13.png)

### Test 6

As we can see, the note with id 1 has been deleted successfully. The only available note is the note with id 2.
![img.png](screenshots/image14.png)

## Tasks

### Task 1

In this task we show containers and images using commands below respectively.
```
docker ps -a
docker image ls
```

In this Picture two containers and first two images belong to our service.
![img.png](screenshots/image15.png)

### Task 2

We should act just like the process we did in Migration part. We run the
 ```ls``` 
 command.
![img.png](screenshots/image16.png)

## Questions

### 1
Dockerfile is a text file that contains instructions for how to build a Docker image. It specifies the steps that need to be taken to create a usable image, including what operating system to use, what software to install, and how to configure the image. Dockerfiles are used to automate the image building process, making it easier to create consistent and repeatable images.

Images are read-only templates that contain the instructions for creating a container. They include the operating system, software, and configuration information that is needed to run an application. Images are stored in a Docker registry, which is a central repository for storing and distributing Docker images.

Containers are running instances of Docker images. They are isolated environments that can run an application and all of its dependencies. Containers are created from images, and they share the image's read-only filesystem. This makes containers lightweight and portable, as they only contain the changes that are needed to run the application.
### 2
Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It groups containers into logical units called pods, which can be replicated and distributed across a cluster of nodes. Kubernetes also provides services for discovering and managing containers, and for load balancing traffic between them.

Kubernetes can be used to manage Docker containers, as well as containers that are built with other containerization technologies. It is a powerful tool for deploying and managing complex applications at scale.

Relationship between Docker and Kubernetes

Docker and Kubernetes are complementary technologies that can be used together to build and manage containerized applications. Docker is used to build and package container images, while Kubernetes is used to deploy, scale, and manage those containers in production.
