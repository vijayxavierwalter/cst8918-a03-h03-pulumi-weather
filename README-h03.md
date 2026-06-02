# Hybrid H03 – Pulumi Secrets and Redis Deployment Notes

## Group 5 Members
- Ilyas Zazai 
- Vijay Walter


## Purpose of Hybrid H03

Hybrid H03 continues from Lab A03. In Lab A03, the weather application was deployed to Azure using Pulumi, Docker, Azure Container Registry, and Azure Container Instances.

Hybrid H03 improves the deployment by adding two important DevOps and cloud features:

1. Secure secret handling using Pulumi secrets.
2. Redis caching using Azure Cache for Redis.

The main goal is to make the application closer to a real cloud deployment by removing the hardcoded OpenWeather API key and using an external cache service instead of only using local memory inside the application container.

---

## Branch Used

This work was completed in the `hybrid-h03` branch.

The branch workflow was:

```text
main
└── hybrid-h03
```

The `hybrid-h03` branch was created after Lab A03 was completed. This keeps Part One and Part Two separate while still using the same group repository.

---

## Secret Handling with Pulumi

In Lab A03, the OpenWeather API key was placed directly inside the Pulumi infrastructure code as an environment variable.

That approach works for a basic lab, but it is not secure for real DevOps work because secrets should not be hardcoded in source code or pushed to GitHub.

For Hybrid H03, the API key was moved into Pulumi secret configuration:

```bash
pulumi config set weatherApiKey "<api-key>" --secret
```

Then the Pulumi infrastructure code was updated to read the secret securely:

```ts
secureValue: config.requireSecret('weatherApiKey')
```

This means Pulumi stores the API key as an encrypted secret and passes it to the Azure container during deployment.

---

## Redis Caching

Before Hybrid H03, the app used local in-memory caching.

That approach has limitations:

- The cache is lost when the container restarts.
- Each container would have its own separate cache.
- It is not ideal for scalable cloud applications.

Hybrid H03 adds Redis caching.

The new application flow is:

```text
User requests weather data
↓
Application checks Redis cache
↓
If cached data exists, return cached data
↓
If cached data does not exist, call OpenWeather API
↓
Store the result in Redis for 10 minutes
↓
Return weather data to the user
```

Redis is useful because it is external to the application container. This makes the cache more reliable and closer to a real cloud architecture.

---

## Application Code Changes

A Redis client dependency was added:

```bash
npm install redis@4.6.14
```

A new Redis connection file was created:

```text
app/data-access/redis-connection.ts
```

This file creates a reusable Redis client connection using the `REDIS_URL` environment variable.

The weather service file was updated:

```text
app/api-services/open-weather-service.ts
```

The service now checks Redis before calling the OpenWeather API. If weather data already exists in Redis, the app returns the cached data. If not, the app calls OpenWeather and stores the result in Redis.

---

## Pulumi Infrastructure Changes

The Pulumi infrastructure was updated to create and configure:

- Azure Resource Group
- Azure Container Registry
- Azure Cache for Redis
- Docker image version `v0.3.0`
- Azure Container Instance
- Secure environment variables for the container

The container receives these important environment variables:

```text
WEATHER_API_KEY
REDIS_URL
```

`WEATHER_API_KEY` is passed from Pulumi secret configuration.

`REDIS_URL` is created from the Azure Redis hostname and Redis access key.

The Pulumi deployment flow is:

```text
Pulumi code
↓
Azure Resource Group
↓
Azure Container Registry
↓
Docker image build and push
↓
Azure Cache for Redis
↓
Azure Container Instance
↓
Public weather application URL
```

---

## Azure Region Issue

During deployment, some Azure regions were blocked by the Azure for Students subscription policy.

The deployment failed in some regions with this error:

```text
RequestDisallowedByAzure
```

The successful deployment used:

```text
westus2
```

The final deployment also used a unique prefix:

```text
cst8918-h03-ilyas
```

This avoided conflicts with the existing Lab A03 deployment.

---

## Docker Issue

During deployment, the Docker image build failed at first because Docker Desktop was not running.

Pulumi needed Docker because the infrastructure code builds the application image locally and pushes it to Azure Container Registry.

The deployment flow was:

```text
Pulumi
↓
Docker build
↓
Azure Container Registry
↓
Azure Container Instance
```

After Docker Desktop was started and the image build was simplified to use `linux/amd64`, the deployment completed successfully.

---

## Successful Pulumi Outputs

The final Pulumi deployment completed successfully with these outputs:

```text
hostname: cst8918-h03-ilyas.westus2.azurecontainer.io
ip: 4.149.195.19
redisHostName: cst8918-h03-ilyas-redis.redis.cache.windows.net
url: http://cst8918-h03-ilyas.westus2.azurecontainer.io:80
```

---

## Final Deployment URL

```text
http://cst8918-h03-ilyas.westus2.azurecontainer.io:80
```

---

## Screenshot Evidence

The Pulumi deployment output screenshot was saved as:

```text
pulumi-output.png
```

This screenshot shows the successful deployment outputs, including the hostname, IP address, Redis hostname, and application URL.

---

## What I Learned

This lab helped me understand how Git, Pulumi, Docker, Azure, and Redis work together in a DevOps workflow.

The main lessons were:

- Git branches help separate Part One and Part Two work.
- Pulumi allows cloud infrastructure to be written and deployed as code.
- Secrets should not be hardcoded in source code.
- Pulumi secrets can store sensitive values securely.
- Redis can be used as an external cache for cloud applications.
- Azure region policies can block deployments, so region selection matters.
- Docker Desktop must be running because Pulumi builds and pushes the Docker image.
- `pulumi up` applies infrastructure changes to Azure.
- Successful Pulumi outputs are important evidence for lab submission.

---

## Summary

Hybrid H03 improved the original Lab A03 deployment by adding secure secret management and Redis caching.

The final architecture is:

```text
User
↓
Azure Container Instance
↓
Weather App Container
↓
Redis Cache
↓
OpenWeather API
```

Pulumi manages the Azure infrastructure, Docker builds and pushes the image, and Azure Container Instance runs the application using secure environment variables.

The final result is a more realistic cloud deployment because the application now uses Infrastructure as Code, encrypted secrets, container deployment, and external Redis caching.


- AI is used for documentation 