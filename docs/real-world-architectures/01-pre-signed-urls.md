# Pre-Signed URLs

## Table of Contents

1. [Overview](#overview)
2. [The Problem with Direct File Uploads](#the-problem-with-direct-file-uploads)
3. [How Pre-Signed URLs Solve This](#how-pre-signed-urls-solve-this)
4. [Step-by-Step Upload Flow with S3](#step-by-step-upload-flow-with-s3)
5. [Architecture Diagram](#architecture-diagram)
6. [Key Benefits](#key-benefits)
7. [Security Considerations](#security-considerations)

---

## Overview

Pre-signed URLs are time-limited, cryptographically signed URLs that grant temporary access to cloud storage resources without requiring permanent credentials. They enable secure, direct uploads from clients to cloud storage services like AWS S3.

---

## The Problem with Direct File Uploads

### Traditional Approach Issues

```
Client → Backend Server → Cloud Storage
```

<img src='../../Resources/18-pre-signed-urls/Screenshot 2026-02-10 at 7.47.14 PM.png' width='500' />

**Problems:**

- **Bandwidth Bottleneck**: All file data flows through your backend server, consuming server resources and bandwidth
- **Scalability Issues**: Server becomes a single point of congestion during high-volume uploads
- **Increased Latency**: Extra hop through backend adds network delay
- **Server Resource Drain**: Memory and CPU spent proxying file data instead of handling business logic
- **Cost**: Paying for bandwidth twice (client→server, server→storage)
- **Complexity**: Backend must handle multipart uploads, resumable uploads, progress tracking

### Example Problem Scenario

```
10 users uploading 100MB files simultaneously
= 1GB of data flowing through your backend
= Server CPU/Memory maxed out
= Other requests timeout
= Poor user experience
```

---

## How Pre-Signed URLs Solve This

<img src='../../Resources/18-pre-signed-urls/Screenshot 2026-02-10 at 8.06.59 PM.png' width='500' />

### Direct Upload Architecture

```
Client → Cloud Storage (S3)
         ↑
         └─ Pre-signed URL from Backend
```

**Solution Benefits:**

- **Direct Connection**: Client uploads directly to S3, bypassing backend
- **Bandwidth Efficiency**: Backend only handles URL generation (minimal overhead)
- **Scalability**: Unlimited concurrent uploads without server strain
- **Reduced Latency**: Direct path to storage, no intermediate hops
- **Cost Optimization**: Single bandwidth charge (client→S3)
- **Simplified Backend**: No file handling logic needed

### How It Works

1. **Backend generates a pre-signed URL** with:
   - Specific S3 bucket and object key
   - Expiration time (e.g., 15 minutes)
   - Allowed HTTP method (PUT or POST)
   - AWS credentials embedded in signature

2. **URL is cryptographically signed** using AWS secret key
   - Only AWS can validate the signature
   - Signature includes: bucket, key, method, expiration, conditions

3. **Client receives URL** and uploads directly to S3
   - No backend involvement during upload
   - S3 validates signature before accepting upload

4. **S3 verifies signature** and accepts/rejects upload
   - Expired URLs are rejected
   - Tampered URLs are rejected
   - Valid uploads are stored

---

## Step-by-Step Upload Flow with S3

### Phase 1: Request Pre-Signed URL

```
┌────────────────────────────────────────────────────────────┐
│ Client                                                     │
│ ┌──────────────────────────────────────────────────────┐   │
│ │ 1. User selects file to upload                       │   │
│ │ 2. Sends request to backend:                         │   │
│ │    POST /api/upload-url                              │   │
│ │    { filename: "photo.jpg", filesize: 5242880 }      │   │
│ └──────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────┐
│ Backend Server                                             │
│ ┌──────────────────────────────────────────────────────┐   │
│ │ 3. Validate request (auth, file size limits)         │   │
│ │ 4. Generate pre-signed URL:                          │   │
│ │    - Bucket: "my-uploads"                            │   │
│ │    - Key: "user-123/photo.jpg"                       │   │
│ │    - Expiration: 15 minutes                          │   │
│ │    - Method: PUT                                     │   │
│ │ 5. Sign URL with AWS credentials                     │   │
│ │ 6. Return URL to client                              │   │
│ └──────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────┐
│ Client                                                     │
│ ┌──────────────────────────────────────────────────────┐   │
│ │ 7. Receives pre-signed URL:                          │   │
│ │    https://my-uploads.s3.amazonaws.com/user-123/...  │   │
│ │    ?X-Amz-Algorithm=AWS4-HMAC-SHA256                 │   │
│ │    &X-Amz-Credential=AKIAIOSFODNN7EXAMPLE/...        │   │
│ │    &X-Amz-Date=20260210T142401Z                      │   │
│ │    &X-Amz-Expires=900                                │   │
│ │    &X-Amz-Signature=abc123...                        │   │
│ └──────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
```

### Phase 2: Direct Upload to S3

```
┌────────────────────────────────────────────────────────────┐
│ Client                                                     │
│ ┌──────────────────────────────────────────────────────┐   │
│ │ 8. Upload file directly to S3:                       │   │
│ │    PUT https://my-uploads.s3.amazonaws.com/...       │   │
│ │    Content: photo.jpg (5MB)                          │   │
│ │    Headers: Content-Type, Content-Length             │   │
│ └──────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────┐
│ AWS S3                                                     │
│ ┌──────────────────────────────────────────────────────┐   │
│ │ 9. Receive upload request                            │   │
│ │ 10. Validate signature:                              │   │
│ │     - Check expiration (not expired)                 │   │
│ │     - Verify HMAC-SHA256 signature                   │   │
│ │     - Confirm bucket and key match                   │   │
│ │ 11. Accept/Reject upload                             │   │
│ │ 12. Return response (200 OK or 403 Forbidden)        │   │
│ └──────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────┐
│ Client                                                     │
│ ┌──────────────────────────────────────────────────────┐   │
│ │ 13. Receive response from S3                         │   │
│ │ 14. Notify backend of completion (optional):         │   │
│ │     POST /api/upload-complete                        │  │
│ │     { key: "user-123/photo.jpg", status: "success" } │   │
│ └──────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
```

### Phase 3: Backend Confirmation (Optional)

```
┌────────────────────────────────────────────────────────────┐
│ Backend Server                                             │
│ ┌──────────────────────────────────────────────────────┐   │
│ │ 15. Verify file exists in S3 (optional)              │   │
│ │ 16. Update database with file metadata               │   │
│ │ 17. Trigger post-processing (resize, scan, etc.)     │   │
│ │ 18. Return confirmation to client                    │   │
│ └──────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
```

---

## Architecture Diagram

### Traditional vs Pre-Signed URL Approach

```
TRADITIONAL APPROACH (Problematic)
═════════════════════════════════════════════════════════════

    ┌──────────┐
    │  Client  │
    └────┬─────┘
         │ 5MB file
         ↓
    ┌──────────────────┐
    │ Backend Server   │ ← Bottleneck: All data flows here
    │ (Proxy)          │
    └────┬─────────────┘
         │ 5MB file
         ↓
    ┌──────────────────┐
    │   AWS S3         │
    │  (Storage)       │
    └──────────────────┘


PRE-SIGNED URL APPROACH (Optimized)
═════════════════════════════════════════════════════════════

    ┌──────────┐
    │  Client  │
    └────┬─────┘
         │ 1. Request URL (minimal)
         ↓
    ┌──────────────────┐
    │ Backend Server   │ ← Only generates URL
    │ (Lightweight)    │
    └────┬─────────────┘
         │ 2. Return signed URL
         ↑
         │
         │ 3. Upload 5MB file directly
         │
    ┌────┴──────────────┐
    │   AWS S3          │
    │  (Storage)        │
    └───────────────────┘
```

---

## Key Benefits

| Aspect | Traditional | Pre-Signed URL |
|--------|-------------|----------------|
| **Bandwidth** | 2x (client→server→S3) | 1x (client→S3) |
| **Server Load** | High (proxies all data) | Low (only URL generation) |
| **Scalability** | Limited by server capacity | Unlimited (direct to S3) |
| **Latency** | Higher (extra hop) | Lower (direct path) |
| **Cost** | Higher (double bandwidth) | Lower (single bandwidth) |
| **Complexity** | Complex (handle uploads) | Simple (URL generation) |
| **Concurrent Uploads** | Limited | Unlimited |

---

## Security Considerations

### Pre-Signed URL Security

- **Time-Limited**: URLs expire after specified duration (default: 15 minutes)
- **Cryptographically Signed**: HMAC-SHA256 signature prevents tampering
- **Scope-Limited**: Each URL is tied to specific bucket, key, and HTTP method
- **Credential-Embedded**: AWS credentials are embedded but cannot be extracted
- **HTTPS Only**: Always use HTTPS to prevent URL interception

### Best Practices

- **Short Expiration**: Use 15-30 minute expiration for most use cases
- **Validate on Backend**: Verify file upload completion before processing
- **Scan Uploads**: Run malware/virus scans on uploaded files
- **Access Control**: Ensure users can only upload to their own paths
- **Logging**: Log all pre-signed URL generation and usage
- **Rate Limiting**: Limit URL generation requests per user/IP

### Example: Secure URL Generation

```
Backend generates URL with:
- Expiration: 15 minutes
- Bucket: "my-uploads"
- Key: "user-{userId}/file-{timestamp}.jpg"
- Method: PUT only
- Conditions: Content-Type must be image/jpeg
```

---

## Implementation Checklist

- [ ] Generate pre-signed URLs on backend
- [ ] Set appropriate expiration time
- [ ] Validate user permissions before generating URL
- [ ] Implement client-side upload with progress tracking
- [ ] Verify file upload completion
- [ ] Scan uploaded files for malware
- [ ] Update database with file metadata
- [ ] Implement error handling and retry logic
- [ ] Log all upload activities
- [ ] Monitor S3 bucket for unauthorized access

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
