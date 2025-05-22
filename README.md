# PocketHost Duplicate Container Issue

## Overview

This repository contains steps to reproduce an issue observed when running PocketHost on a VPS server. The problem occurs when multiple simultaneous requests trigger the creation of duplicate containers for the same PocketBase instance.

## Environment

- PocketHost running on a VPS
- PocketBase instances with realtime subscriptions enabled

## Problem Description

When a client loads the web app and sends both:

- An initial HTTP API request, and
- A PocketBase realtime subscription request

almost simultaneously, the instance manager may start **two containers or more** for the same instance subdomain. 

## Steps to Reproduce
1. Run `docker stats` to observe two running containers for the same instance ID.
2. Open the client web app served by PocketHost.
3. Immediately after page load, the frontend triggers:
   - An HTTP request to the PocketBase API (e.g., `GET /api/collections/xyz/records`)
   - A `realtime.subscribe()` call on the same collection.
4. These requests hit the instance manager nearly at the same time.
5. The container management logic does not reliably prevent duplicate container creation.

5. One container remains idle but is not shut down automatically.