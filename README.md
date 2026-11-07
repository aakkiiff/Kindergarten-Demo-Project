# Kindergarten Project — Run Locally

Run everything with Docker Compose, or run applications manually using the
steps below. There is no reverse proxy: the browser talks to each service directly.

![Kindergarten application architecture](./image.png)

## Run everything with Docker Compose

Only Docker with Compose is required. Copy `.env.example` to `.env` if it does
not already exist, and set your MongoDB, RabbitMQ, and RustFS credentials.
Use URL-safe characters in credentials. Keep existing credentials when reusing
volumes; editing `.env` does not change stored accounts.

```sh
docker compose up -d --build --wait
docker compose ps
```

Open **http://localhost:3000**. Compose runs the frontend, four APIs, thumbnail
worker, MongoDB, RabbitMQ, RustFS, and Mongo Express.
No service-specific `.env` files or local development tools are needed.

| Address | Purpose |
| --- | --- |
| http://localhost:3000 | School application |
| http://localhost:5001/students | Student API |
| http://localhost:5002/teachers | Teacher API |
| http://localhost:5003/employees | Employee API |
| http://localhost:5004/ready | Media readiness |
| http://localhost:8081 | MongoDB dashboard |
| http://localhost:15672 | RabbitMQ dashboard |
| http://localhost:9001/rustfs/console/ | RustFS dashboard |

The browser calls the APIs directly (the frontend image is built with the
`http://localhost:500x` bases from `frontend/.env.production`). Thumbnails are
private signed S3 URLs served by RustFS on `http://localhost:9000`; do not change
their query parameters. `profileservice` shares the `object-storage` network
(`network_mode`), so `S3_ENDPOINT=http://localhost:9000` is valid both for the
service and for the browser, which keeps the signed host correct. Its port 5004
is therefore published on `object-storage`. Backing service ports and dashboards
bind to localhost.

```sh
docker compose logs --tail=50 studentservice teacherservice employeeservice profileservice thumbnailworker
docker compose up -d --build   # rebuild after code changes
docker compose stop           # stop without deleting data
```

Verify uploads, private thumbnails, replacement, and deletion using disposable
records:

```sh
# from thumbnailworker/ with its .env and dependencies installed on the host
python test_pipeline.py
```

Backend structured logs and request IDs are unchanged.

Do not run `docker compose down -v` unless you intend to delete stored data.
This is a local teaching demo, not a production deployment.

## Run applications manually

The following steps use the same backing services but start APIs and the
frontend on your computer. Stop Compose application containers first:

```sh
docker compose stop frontend studentservice teacherservice employeeservice profileservice thumbnailworker
```

## Step 1: Prepare the project

Install Docker Compose, Node.js 22+, Go 1.23+, Python 3.13+, JDK 17, and Maven.
Open a terminal in the repository root.

Copy the example environment files:

- in every directory there is a .env.example, make it .env for the application to read it

- Set credentials in the root `.env`. Update each service's `.env` with matching
MongoDB, RabbitMQ, and RustFS credentials where required. Keep these files
private; do not commit credentials.

## Step 2: Start the supporting services

From the repository root, run:

```sh
docker compose up -d --wait mongo mongo-express rabbitmq object-storage
docker compose ps
```

Open the MongoDB dashboard at [http://localhost:8081](http://localhost:8081).
Sign in with `MONGO_ROOT_USERNAME` and `MONGO_ROOT_PASSWORD` from `.env`.

Other consoles:

| Service | Address |
| --- | --- |
| RustFS | http://localhost:9001/rustfs/console/ |
| RabbitMQ | http://localhost:15672 |

Use the corresponding credentials in the root `.env`.

Every service below runs in its own terminal. Keep each one running.
Each service reads its own `.env` file from its folder, so always start it
from inside that folder.

## Step 3: Student service (Go, port 5001)

```sh
cd studentservice
go mod tidy
go run main.go
```

Check: http://localhost:5001/students returns a list (empty is fine).
## Step 4: Frontend (React, port 3000)

```sh
cd frontend
npm i
npm start
```

Open [http://localhost:3000](http://localhost:3000). API addresses default to
ports `5001`–`5004`; change them in `frontend/.env` and restart if needed.

## Step 5: Teacher service (Java, port 5002)

Requires **JDK 17** (newer versions break the build) and Maven.

In the teacher service terminal, point to JDK 17 and confirm it
(repeat in every new terminal):

```sh
mvn -version    # must show: Java version: 17
```

Build and run:

```sh
cd teacherservice
mvn clean package
java -jar target/teacherservice-1.0.0.jar
```

The first start downloads dependencies and can take a few minutes.
Check: http://localhost:5002/teachers returns a list.

## Step 6: Employee service (Python, port 5003)

```sh
cd employeeservice
pip install -r requirements.txt
python app.py
```

Check: http://localhost:5003/employees returns a list.

## Step 7: Profile (media) service (Node.js, port 5004)

Uploads photos to RustFS and queues thumbnail jobs in RabbitMQ.

```sh
cd profileservice
npm i
npm start
```

Check: http://localhost:5004/ready reports ready.

## Step 8: Thumbnail worker (Python)

Start it after the profile service. It turns uploaded photos into thumbnails.

```sh
cd thumbnailworker

pip install -r requirements.txt
python worker.py
```

Check: the terminal shows it is waiting for messages.


## Step 9: Verify everything

- The footer shows **3 of 3 services connected**.
- The Students, Teachers, and Employees tabs load and accept new records.
- A record with a photo shows **Processing**, then a thumbnail.
- The MongoDB dashboard shows the new records in the `kindergarten` database.
