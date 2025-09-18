# 🐳 Week 3 Homework: Docker Compose & Docker Swarm

## 🧠 Objective

This week you’ll take the microservices from Week 2 and orchestrate them using Docker Compose. You’ll also explore Docker Swarm as an alternative for scaling and managing services.

---

## 🧰 Tools

- Docker Compose
- Docker CLI
- Docker Swarm mode (local or Minikube)

---

## Part 1: Docker Compose 

### ✅ Task

1. <b>Edit existing `docker-compose.yml` file and update its content to reference to your docker hub images.</b>
2. Run Docker compose

```bash
docker compose up --build
```
4. Test that services are reachable.

You should be able to browse different components of the application by using the below URLs :

```
Web Status : http://localhost:5107/
Web MVC :  http://localhost:5100/
Web SPA :  http://localhost:5104/
```
---

### 📝 Deliverables

- A working `docker-compose.yml`
- Screenshot of services running
- A short `README.md` describing how to run it

---

## Part 2: Docker Swarm 

### ✅ Task

1. Initialize Swarm:
```bash
docker swarm init
```

2. Convert your Compose file into a stack.yml:
--> <b>Make sure `docker-compose.yml` use image reference not `build`</b>

3. Create `stack.rendered.yml` file

```bash
docker compose --env-file .env -f docker-compose.yml -f  docker-compose.override.yml config > stack.rendered.yml
```

4. Verify `stack.rendered.yml`
```bash
docker stack config -c stack.rendered.yml 
```
<b>Expect warning errors, Fix working error till you receive successful verification</b>

5. Deploy Stack
```bash
docker stack deploy -c stack.rendered.yml mystack
```

6. Check status with:
```bash
docker stack services myapp
```

7. Test that services are reachable.

You should be able to browse different components of the application by using the below URLs :

```
Web Status : http://localhost:5107/
Web MVC :  http://localhost:5100/
Web SPA :  http://localhost:5104/
```
---

8. Tear down:
```bash
docker stack rm myapp
```

---

## ⏱️ Deadline

Submit/email docker compose and stack file

---

