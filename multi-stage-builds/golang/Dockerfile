FROM golang:1.24-alpine AS builder

WORKDIR /app

COPY . .

RUN --mount=type=cache,target=/go/cache,id=go-mod \
  go mod tidy

RUN go build -o main .

FROM scratch

COPY --from=builder /app/main /main

EXPOSE 8080

CMD ["/main"]
