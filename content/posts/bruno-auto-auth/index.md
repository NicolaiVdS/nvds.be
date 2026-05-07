---
title: "Seamless Auto-Authentication in Bruno API Client"
date: 2026-05-07T10:54:35+02:00
draft: false
toc: true
images:
tags:
  - bruno
  - api
  - authentication
  - testing
---

# Seamless Auto-Authentication in Bruno API Client

When building and testing APIs, token expiry is an annoying but unavoidable reality. If your tokens expire mid-session, you're stuck manually re-logging in and re-running your requests. In this post I'll walk through how I set up fully automatic token refresh in [Bruno](https://www.usebruno.com/) no manual intervention needed.

## The Problem

The naive approach to handling expired tokens is to catch 401 responses in a post-response script and retry the request. The issue? Bruno always displays the **original** response in the UI there's no way to replace it from a script. So even if your retry succeeds, you're staring at a 401.

## The Solution

Instead of reacting to 401s after the fact, we **proactively check and refresh the token before it expires**. A collection-level pre-request script runs before every request, silently refreshes the token if needed, and then lets the actual request fire with valid credentials.

The result: every request just works, and you always see the real response in the UI.

---

## Setup

The setup involves three scripts:

1. A post-response script on your **Login** request
2. The same post-response script on your **Refresh** request
3. A pre-request script at the **collection level**

### 1. Login - Post Response Script

After a successful login, we store the access token, the refresh token (from the `set-cookie` header), and crucially the expiry timestamp.

```javascript
const body = res.body?.data;
const cookies = res.headers["set-cookie"];

if (body?.access_token) {
  bru.setEnvVar("access_token", body.access_token);
  // Store expiry with a 10s safety buffer
  bru.setVar("token_expiry", Date.now() + ((body.expires_in - 10) * 1000));
}

if (cookies) {
  const list = Array.isArray(cookies) ? cookies : [cookies];
  for (const cookie of list) {
    const match = cookie.match(/refresh_token=([^;]+)/);
    if (match) {
      bru.setEnvVar("refresh_token", decodeURIComponent(match[1]));
      break;
    }
  }
}
```

### 2. Refresh - Post Response Script

Identical to the login script. After a successful refresh, we store the new tokens and reset the expiry timer.

```javascript
const body = res.body?.data;
const cookies = res.headers["set-cookie"];

if (body?.access_token) {
  bru.setEnvVar("access_token", body.access_token);
  bru.setVar("token_expiry", Date.now() + ((body.expires_in - 10) * 1000));
}

if (cookies) {
  const list = Array.isArray(cookies) ? cookies : [cookies];
  for (const cookie of list) {
    const match = cookie.match(/refresh_token=([^;]+)/);
    if (match) {
      bru.setEnvVar("refresh_token", decodeURIComponent(match[1]));
      break;
    }
  }
}
```

### 3. Collection - Pre Request Script

This is the key piece. It runs automatically before every request in your collection, checks whether the token is still valid, and silently refreshes or re-logs in if not.

```javascript
// Skip auth requests themselves to avoid infinite loops
const name = req.getName();
if (name === "Login" || name === "Refresh") return;

const token = bru.getEnvVar("access_token");
const expiry = bru.getVar("token_expiry");
const isExpired = !expiry || Date.now() > expiry;

if (!token || isExpired) {
  const refreshToken = bru.getEnvVar("refresh_token");

  if (refreshToken) {
    const r = await bru.runRequest("Refresh");
    if (r.status === 200) {
      console.log("Token refreshed silently");
      return;
    }
  }

  // Refresh failed or no refresh token → full login
  const r = await bru.runRequest("Login");
  if (r.status === 200) {
    console.log("Logged in silently");
  } else {
    console.error("Auth failed — request may fail");
  }
}
```

## How It Works

```
Any request fires
       ↓
Collection pre-request script runs
       ↓
Is token missing or expired?
   ├── No  → request fires normally
   └── Yes → try Refresh
               ├── 200 → store new token → request fires
               └── fail → try Login
                           ├── 200 → store new token → request fires
                           └── fail → request fires anyway (will likely 401)
```

---

## A Few Things to Note

**The 10 second safety buffer:** using `expires_in - 10` instead of `expires_in` prevents a race condition where the token expires in the brief moment between the check and the actual HTTP request going out.

**`bru.setVar` vs `bru.setEnvVar`:** `token_expiry` uses `setVar` because it only needs to live for the current session. The actual tokens use `setEnvVar` so they persist across requests.

**Request name matching:** the names passed to `bru.runRequest()` must **exactly match** the request names in your Bruno collection, including capitalisation. If your login request is called `"auth/login"` rather than `"Login"`, update the script accordingly.

---

That's it. Once this is in place you never have to think about token expiry again, Bruno handles it silently in the background before every single request.
