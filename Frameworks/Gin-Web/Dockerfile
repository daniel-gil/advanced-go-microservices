# golang docker base image
FROM golang:1.12.0-alpine

# install the gin web framework (and install required packages for executing `go get`)
RUN apk update && apk upgrade && apk add --no-cache bash git
RUN go get github.com/gin-gonic/gin

# copy local sources into this docker image (defining an environment variable SOURCES)
ENV SOURCES /go/src/github.com/daniel-gil/advanced-go-microservices/Frameworks/Gin-Web/
COPY . ${SOURCES}

# change into the SOURCES directory and call `go build`
RUN cd ${SOURCES} && CGO_ENABLED=0 go build

# define the work directory
WORKDIR ${SOURCES}

# define the entry-poing command (it is, the Gin-Web executable after building the binary)
CMD ${SOURCES}Gin-Web

# expose port 8080
EXPOSE 8080
