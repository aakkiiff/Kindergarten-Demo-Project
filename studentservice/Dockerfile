# Stage 1: Build the Go application
FROM golang:1.24 AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN CGO_ENABLED=0 GOOS=linux go build -o studentservice .

# Stage 2: Run the application
FROM alpine:latest

WORKDIR /app

COPY --from=builder /app/studentservice .

EXPOSE 5001

CMD ["./studentservice"]
