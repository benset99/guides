# Part of the Learning Go series
### My attempt at using Go to improve my programming skills

## Creating a container with Docker and using Go to host a webserver
For this guide, I am using UbuntuLTS

1. Install docker

```
sudo apt update
sudo apt install docker
sudo apt install docker.io
sudo usermod -aG docker ${USER}
sudo systemctl start docker
sudo systemctl enable docker
```

2. Install Golang

`sudo apt install golang-go`

3. Create a Go program

`vim main.go`

```
package main

import (
  "net/http"
  "io"
)

func main() {
  http.HandleFunc("/", servePage)
	http.ListenAndServe(":8081", nil)
}

func servePage(writer http.ResponseWriter, reqest *http.Request) {
  io.WriteString(writer, "Hello world!")
}```

4. Create a Dockerfile

`vim Dockerfile`

```
## We specify the base image we need for our
## go application
FROM golang:1.12.0-alpine3.9
## We create an /app directory within our
## image that will hold our application source
## files
RUN mkdir /app
## We copy everything in the root directory
## into our /app directory
ADD . /app
## We specify that we now wish to execute 
## any further commands inside our /app
## directory
WORKDIR /app
## we run go build to compile the binary
## executable of our Go program
RUN go build -o main .
## Our start command which kicks off
## our newly created binary executable
CMD ["/app/main"]
```

5. Build image using the Dockerfile

`docker build -t my-go-app .`

6. Create a docker container to use the newly created image

`docker run -p 8080:8081 -d my-go-app`

7. Check to see if your container works 

`curl http://localhost:8080`

Output:

`Hello world!`

8. 
