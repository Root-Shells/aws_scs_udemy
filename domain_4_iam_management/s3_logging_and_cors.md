# S3 Access Logging & CORS

## Overview
Auditability and controlled sharing are two pillars of **Amazon S3** security. **Server Access Logging** provides a detailed record of every request made to a bucket, while **Cross-Origin Resource Sharing (CORS)** allows web browsers to securely interact with assets stored in S3 from different origins.

## Key Concepts
- **Server Access Logs**: Detailed logs of all requests (authorized or denied) made to a bucket.
- **Logging Loop**: A dangerous misconfiguration where a bucket logs its own activity into itself, causing exponential growth.
- **CORS (Cross-Origin Resource Sharing)**: A browser-level security mechanism that allows or restricts web applications at one domain from accessing resources at another domain.
- **Pre-flight Request**: An HTTP `OPTIONS` request sent by browsers to check if a cross-origin request is allowed.

## Detailed Notes

### 1. S3 Server Access Logging
For security audits, all activity in a bucket can be logged to a destination bucket.
- **Data Analyzed**: Includes requester, bucket name, request time, action, response status, and error codes.
- **Athena Integration**: Logs are stored as text files in S3 and can be queried using **AWS Athena**.
- **Permissions**: The destination bucket must have a bucket policy allowing the `logging.s3.amazonaws.com` service principal to `s3:PutObject`.

> **Caution**: **Never** set the destination logging bucket to be the same as the source bucket. This creates an infinite loop of logging put-events, leading to massive AWS bills.

### 2. S3 CORS (Cross-Origin Resource Sharing)
CORS is required when you host a website (e.g., `www.example.com`) that needs to load assets (like fonts, scripts, or images) from an S3 bucket (e.g., `assets.example.com`).
- **Origin**: Defined by protocol + host + port (e.g., `https://example.com:443`).
- **Mechanism**: The S3 bucket must return specific headers (`Access-Control-Allow-Origin`) to tell the browser the request is safe.
- **Configuration**: Defined in JSON format under the bucket's "Permissions" tab.

#### CORS JSON Example
```json
[
  {
    "AllowedOrigins": ["https://www.example.com"],
    "AllowedMethods": ["GET", "PUT"],
    "AllowedHeaders": ["*"],
    "MaxAgeSeconds": 3000
  }
]
```

## Architecture / Flow

### CORS Pre-flight Request
```mermaid
flowchart TD
    subgraph Browser [Web Browser]
        App["App at site-a.com"]
    end

    subgraph S3_Bucket [S3 Bucket (site-b.com)]
        Policy["CORS Config"]
    end

    App -->|"1. OPTIONS (Pre-flight)"| S3_Bucket
    S3_Bucket -->|"2. Check site-a.com in AllowedOrigins"| Policy
    Policy -- "3. Headers: Allow site-a.com" --> App
    App -->|"4. Actual GET Request"| S3_Bucket
```

## Security Relevance
- **Detective Control**: **Server Access Logs** are the primary source for investigating data breaches or unauthorized access attempts in S3.
- **Browser Security**: CORS prevents malicious websites from making unauthorized requests to your S3 bucket on behalf of a user.

## Operational / Real-World Context
- **Compliance**: Regulated industries (HIPAA, PCI) require access logs to be retained for years. Move these logs to **S3 Glacier** for cost-effective long-term storage.
- **Static Websites**: If you host a frontend on S3 and your API is on another domain, you must configure CORS on the API or asset bucket.

## Common Pitfalls / Misconfigurations
- **The Logging Loop**: (Repeated for emphasis) Avoid logging into the same bucket.
- **Log Latency**: S3 access logs are not real-time; they can take several hours to appear in the destination bucket.
- **Wildcard Origins**: Using `*` in `AllowedOrigins` is easy for development but should be avoided in production to prevent unauthorized domains from loading your assets.

## Exam / Review Notes
- **CORS**: The answer for "browser-based request blocked" or "Access-Control-Allow-Origin" error.
- **Access Logs**: Used for auditing. Destination must be in the **same region**.
- **Athena**: The standard tool for querying S3 Access Logs.
- **HTTPS Enforcement**: Use `aws:SecureTransport: false` in a `Deny` statement.

## Summary
Audit logs provide the "Who, What, and When" for data access, while CORS ensures that web-based access remains secure and within defined boundaries. Both features are essential for maintaining a production-ready S3 environment.

## Quick Review Checklist
- [ ] Destination logging bucket must be in the SAME region.
- [ ] Do NOT log a bucket to itself (Logging Loop).
- [ ] Use Athena to analyze S3 logs.
- [ ] CORS is a browser security mechanism.
- [ ] `AllowedOrigins` must match the requester's domain exactly (protocol + host).
