FROM golang:1.8-alpine
ADD . /go/src/demo-app
RUN go install demo-app

FROM alpine:latest
COPY --from=0 /go/bin/demo-app .
ENV PORT 8080
CMD ["./demo-app"]
