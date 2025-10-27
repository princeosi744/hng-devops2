# HNG DevOps Stage 2 - Blue/Green with Nginx

# Blue/Green Deployment with Nginx Auto-Failover

This repository implements a Blue/Green deployment strategy for a Node.js application with automatic failover using Nginx as a reverse proxy.

## Overview

The setup includes:
- **Blue Service**: Primary active service (port 8081)
- **Green Service**: Backup service (port 8082)
- **Nginx**: Reverse proxy with auto-failover (port 8080)

### Key Features
- Automatic failover from Blue to Green on failures
- Zero downtime during failover
- Proper header forwarding (`X-App-Pool`, `X-Release-Id`)
- Quick failure detection with tight timeouts
- Retry logic for transparent failover
jie-aafi-ozz

## Files
- `docker-compose.yml` - runs nginx, app_blue, app_green
- `nginx.conf.template`- nginx template (uses $ACTIVE_POOL)
- `.env.example` - example env file to copy as `.env`

## Quick setup (local)
1. Copy `.env.example` to `.env` and fill `BLUE_IMAGE` and `GREEN_IMAGE` with the provided images.
