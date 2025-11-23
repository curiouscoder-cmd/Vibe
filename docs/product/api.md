# API Endpoints

## API Request Flow

<img src="../../diagram/api.svg"/>
<details>
<summary>Click to view API Flow Diagram</summary>

```mermaid
sequenceDiagram
    participant User
    participant App
    participant LB as Load Balancer
    participant Server as App Server
    participant Cache as Redis
    participant Mongo as MongoDB
    participant S3

    User->>App: Open Discovery Feed
    App->>LB: GET /api/discovery/profiles
    LB->>Server: Route Request
    
    Server->>Cache: Check Cache
    alt Cache Hit
        Cache-->>Server: Return Cached Profiles
    else Cache Miss
        Server->>Mongo: Query Matching Profiles
        Mongo-->>Server: Return Profile Data
        Server->>Cache: Store in Cache
    end
    
    Server->>S3: Get Image URLs
    S3-->>Server: Return CDN URLs
    
    Server-->>LB: JSON Response
    LB-->>App: Profile List
    App-->>User: Display Swipe Cards
    
    User->>App: Swipe Right
    App->>LB: POST /api/swipe
    LB->>Server: Route Request
    Server->>Mongo: Insert Swipe Record
    Server->>Mongo: Check for Match
    
    alt Match Found
        Mongo-->>Server: Match Detected
        Server->>Mongo: Create Match Record
        Server->>Cache: Invalidate Cache
        Server-->>App: Match Response
        App-->>User: Show Match Screen
    else No Match
        Mongo-->>Server: No Match
        Server-->>App: Swipe Recorded
        App-->>User: Next Profile
    end
```
</details>

## Core Endpoints

### Authentication
```
POST   /api/auth/register        - Register with phone/email
POST   /api/auth/verify-otp      - Verify OTP code
POST   /api/auth/login           - Login
POST   /api/auth/logout          - Logout
POST   /api/auth/refresh-token   - Refresh JWT token
```

### Profile
```
POST   /api/profile              - Create profile
GET    /api/profile/me           - Get own profile
PUT    /api/profile              - Update profile
POST   /api/profile/photos       - Upload photos
DELETE /api/profile/photos/:id   - Delete photo
PUT    /api/profile/preferences  - Update preferences
```

### Discovery
```
GET    /api/discovery/profiles   - Get profiles to swipe (with filters)
POST   /api/swipe                - Swipe action (like/pass)
```
