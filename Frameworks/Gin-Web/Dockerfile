FROM golang:1.12.0-alpine

RUN apk update && apk upgrade && apk add --no-cache bash git openssh
RUN go get github.com/gin-gonic/gin

ENV SOURCES /go/src/github.com/daniel-gil/advanced-go-microservices/Frameworks/Gin-Web/
COPY . ${SOURCES}

RUN cd ${SOURCES} && CGO_ENABLED=0 go build

WORKDIR ${SOURCES}
CMD ${SOURCES}Gin-Web
EXPOSE 8080
