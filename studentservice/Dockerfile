# Stage 1: Build stage
FROM golang:1.23 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o /studentservice .

FROM gcr.io/distroless/static-debian12:nonroot
WORKDIR /app
COPY --from=builder /studentservice /app/studentservice
EXPOSE 5001
ENTRYPOINT ["/app/studentservice"]