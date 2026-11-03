# 2. What Is a Server and Where Does Code Live?

## 1. Concept and Fundamentals

A server is a process and machine serving requests. It consists of compute resources, an operating system, application runtime or binary, configuration, network access, and supporting storage. “Server” can mean a physical host, VM, container, serverless function, or a running process behind a network endpoint.

Code lives in source control during development, in build artifacts or container images after build, and in memory/disk on deployed servers at runtime. The deployed server generally should not run directly from a developer’s laptop.

### Lifecycle

```text
Source -> Build -> Artifact/Image -> Registry -> Deploy -> Runtime -> Observe
```

### Why It Matters

Understanding deployment lifecycle prevents configuration drift, lost changes, insecure secrets, non-reproducible builds, and assumptions that one host is special.

### What Happens Without It?

Manual server configuration causes inconsistent environments, difficult rollback, slow recovery, and “works on my machine” failures.

## 2. Real-World Analogy

Source code is a recipe. Build creates packaged meal instructions/ingredients. A container image is a prepared meal kit. An instance is a kitchen running that kit. The registry is the warehouse storing kits for consistent deployment.

## 3. Technical Architecture

```text
Developer
 -> Git repository
 -> CI pipeline / Jenkins
 -> Build and test
 -> Docker image / release package
 -> Artifact registry / ECR
 -> EC2, ECS, EKS, or serverless runtime
 -> Internet through ALB/API Gateway
```

Application processes are not usually written into the database or directly into object storage. They read immutable code/configuration, write logs externally, keep persistent data in databases or object storage, and keep temporary files only when recoverable.

## 4. Practical Use Cases

- Web API deployment on EC2 behind ALB.
- Container deployment to ECS or Kubernetes.
- Static React website on S3/CloudFront.
- Serverless API in Lambda behind API Gateway.

## 5. DevOps and AWS Implementation

### EC2

Store the application in an AMI or package registry and deploy through a launch template/User Data. Runtime code lives on the instance filesystem, so replace instances rather than repairing them by hand.

### Containers

Build images with Docker. Use immutable images tagged with commit/build identifiers. AWS ECR stores images; ECS/EKS/EC2 pulls and runs them. Configuration should come from environment variables or secrets.

### Linux Process Behavior

A service should run under a non-root user, have a systemd unit when appropriate, bind to an internal interface or expected port, and write logs to stdout/stderr for collection. Health endpoints validate readiness and liveness.

### Secrets

Never place passwords in Git, Docker images, or launch scripts. Use AWS Secrets Manager or Systems Manager Parameter Store, scoped IAM permissions, encryption, and rotation.

## 6. Practical Production Scenario

A .NET API image is built by Jenkins, scanned, pushed to ECR, and deployed to ECS. ECS starts fresh tasks across Availability Zones. An RDS endpoint and Redis endpoint are supplied through configuration. Application secrets come from Secrets Manager. CloudWatch captures task logs and metrics.

If one task crashes, ECS launches another. If an image is bad, deploy a known-good image rather than editing the damaged task.

## 7. Hands-On Lab

```bash
mkdir -p demo/app && cd demo/app
printf 'FROM nginx:alpine\nCOPY index.html /usr/share/nginx/html/index.html\n' > Dockerfile
printf '<h1>Deployment works</h1>' > index.html

docker build -t system-design-demo:v1 .
docker run -d --name demo -p 8081:80 system-design-demo:v1
curl -i http://localhost:8081/
```

Expected output: HTTP 200 and “Deployment works”.

Update `index.html`, rebuild as `v2`, and run another container on port 8082. This demonstrates immutable artifacts, but production health checks and rollback belong around the deployment orchestrator.

Cleanup: `docker rm -f demo; docker image prune -f`.

## 8. Advantages, Disadvantages and Best Practices

- Prefer immutable deployments and infrastructure as code.
- Treat local disk and instance memory as ephemeral unless explicitly designed as stateful storage.
- Pin base image versions and package dependencies.
- Separate build, runtime, and data credentials.
- Use graceful shutdown to finish existing requests when possible.

Do not install application dependencies manually on production hosts as the normal deploy process.

## 9. Interview Preparation

**Q1: Where does code live?** Source code in Git, built artifacts in registries, running processes in memory, and files on instance/container storage.

**Q2: Stateless or stateful server?** Stateless instances store session/data externally; stateful instances persist local durable state and require specialized backup/placement.

**Q3: What is immutable infrastructure?** Servers/containers are replaced rather than modified after deployment.

**Q4: What is ephemerality?** Runtime infrastructure can disappear without preserving its local state.

**Q5: Why use containers?** They package runtime dependencies consistently and provide an isolated process packaging format.

**Scenario:** EC2 user accidentally edited nginx on one server. Use the launch template/IaC repository to reproduce both servers and replace the modified instance.

**Scenario:** An image deployed successfully but has a bad environment variable. Verify configuration, roll back or redeploy a known image/configuration, and ensure secret changes are audited.

**60-second answer:** “A server executes packaged code and configuration behind a network interface. Production deployments should use repeatable artifacts, externalize state and secrets, and replace instances rather than manually changing them.”

## 10. Revision Notes

- Source → build → artifact → registry → deployment → runtime.
- Code runs in processes and memory/disk.
- Persistent state belongs in databases, object storage, or volumes with explicit lifecycle.
- Immutable infrastructure improves consistency and rollback.

**Challenge:** Package a tiny API in Docker, run it twice, place an ALB-like local proxy in front, stop one instance, and observe recovery traffic behavior.
