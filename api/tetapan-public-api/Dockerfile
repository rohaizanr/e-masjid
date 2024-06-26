# Stage 1: Build the application
FROM golang:alpine3.19 AS builder

# Set the working directory inside the container
WORKDIR /app

# Copy the local code to the container
COPY . .
COPY .env.* .

# Install any dependencies your application might need
RUN go get -d -v ./...
RUN go install -v ./...
RUN go install github.com/swaggo/swag/cmd/swag@latest

# Build the Go application
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o tetapan-public-api

#Generate api doc
RUN swag init

# Stage 2: Create a minimal runtime image
FROM alpine:latest

# Define build-time argument for GO_ENV
ARG GO_ENV=local

# Set Go environment variables
ENV GO_ENV=$GO_ENV

# Set the working directory inside the container
WORKDIR /app

# Copy only the built binary from the builder stage
COPY --from=builder /app/tetapan-public-api .
COPY --from=builder /app/.env.* .
COPY --from=builder /app/wait-for-it.sh . 
COPY --from=builder /app/docs ./docs

# Expose the port that your application will run on
EXPOSE 8080

# Command to run your application
CMD ["./tetapan-public-api", "-env", "$GO_ENV"]
