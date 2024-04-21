# Buil stage
FROM golang:1.22-alpine as build
WORKDIR /app
ADD main.go .
RUN CGO_ENABLED=0 GOOS=linux go build -o go-app main.go

# Final stage
FROM alpine:3.19
ENV PORT 8080
WORKDIR /app

RUN apk add --update vim && \
    rm -rf /var/cache/apk/* && \
    adduser -D www

COPY --from=build /app/go-app .
RUN chown -R www:www /app

USER www
EXPOSE ${PORT}

CMD ["./go-app"]
