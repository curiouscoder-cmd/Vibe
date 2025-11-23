# Database Design

## MongoDB Schema

<img src="../../diagram/er.svg" alt="Database Schema">

<details>
 <summary>Click to view Database Schema Diagram</summary>

 ```mermaid
erDiagram
    USERS ||--o{ PROFILES : has
    USERS ||--o{ SWIPES : makes
    USERS ||--o{ MATCHES : participates
    MATCHES ||--o{ MESSAGES : contains
    
    USERS {
        ObjectId _id
        String phone_number
        String email
        String password_hash
        Boolean is_verified
        Date created_at
        Date last_active
    }
    
    PROFILES {
        ObjectId _id
        ObjectId user_id
        String name
        Number age
        String gender
        String bio
        Object location
        Array photos
        Object preferences
    }
    
    SWIPES {
        ObjectId _id
        ObjectId swiper_id
        ObjectId swiped_id
        String action
        Date created_at
    }
    
    MATCHES {
        ObjectId _id
        ObjectId user1_id
        ObjectId user2_id
        Date matched_at
        Boolean is_active
    }
    
    MESSAGES {
        ObjectId _id
        ObjectId match_id
        ObjectId sender_id
        String content
        String message_type
        Boolean is_read
        Date sent_at
    }
```
</details>

### Collections Overview

```javascript
// 1. users collection
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  phone_number: "+1234567890",
  email: "user@example.com",
  password_hash: "$2b$10$...",
  is_verified: true,
  created_at: ISODate("2025-01-15T10:30:00Z"),
  last_active: ISODate("2025-01-20T14:22:00Z")
}

// 2. profiles collection
{
  _id: ObjectId("507f1f77bcf86cd799439012"),
  user_id: ObjectId("507f1f77bcf86cd799439011"),
  name: "John Doe",
  age: 28,
  gender: "male",
  bio: "Love hiking and coffee",
  location: {
    type: "Point",
    coordinates: [-122.4194, 37.7749] // [lng, lat] San Francisco
  },
  photos: [
    "https://cdn.example.com/user123/photo1.jpg",
    "https://cdn.example.com/user123/photo2.jpg"
  ],
  preferences: {
    age_min: 24,
    age_max: 35,
    distance: 25, // km
    gender: "female"
  }
}

// 3. swipes collection
{
  _id: ObjectId("507f1f77bcf86cd799439013"),
  swiper_id: ObjectId("507f1f77bcf86cd799439011"),
  swiped_id: ObjectId("507f1f77bcf86cd799439022"),
  action: "like", // or "pass"
  created_at: ISODate("2025-01-20T14:25:00Z")
}

// 4. matches collection
{
  _id: ObjectId("507f1f77bcf86cd799439014"),
  user1_id: ObjectId("507f1f77bcf86cd799439011"),
  user2_id: ObjectId("507f1f77bcf86cd799439022"),
  matched_at: ISODate("2025-01-20T14:25:30Z"),
  is_active: true
}

// 5. messages collection
{
  _id: ObjectId("507f1f77bcf86cd799439015"),
  match_id: ObjectId("507f1f77bcf86cd799439014"),
  sender_id: ObjectId("507f1f77bcf86cd799439011"),
  content: "Hey! How are you?",
  message_type: "text", // or "image", "voice"
  is_read: false,
  sent_at: ISODate("2025-01-20T14:30:00Z")
}
```

### MongoDB Indexes

```javascript
// Essential indexes for performance

// Geospatial index for location queries
db.profiles.createIndex({ location: "2dsphere" });

// Swipes lookup
db.swipes.createIndex({ swiper_id: 1, swiped_id: 1 }, { unique: true });
db.swipes.createIndex({ swiper_id: 1, created_at: -1 });

// Active users
db.users.createIndex({ last_active: -1, is_verified: 1 });

// Match queries
db.matches.createIndex({ user1_id: 1, user2_id: 1 });
db.matches.createIndex({ user1_id: 1, is_active: 1 });
db.matches.createIndex({ user2_id: 1, is_active: 1 });

// Message history
db.messages.createIndex({ match_id: 1, sent_at: -1 });
db.messages.createIndex({ sender_id: 1, sent_at: -1 });

// Profile search
db.profiles.createIndex({ user_id: 1 }, { unique: true });
db.profiles.createIndex({ age: 1, gender: 1 });
```

## MongoDB Scaling Strategy

### Database Architecture

**Collections Structure**:
```javascript
// users collection
{
  _id: ObjectId,
  phone_number: String (indexed, unique),
  email: String (indexed, unique),
  password_hash: String,
  is_verified: Boolean,
  created_at: Date,
  last_active: Date
}

// profiles collection
{
  _id: ObjectId,
  user_id: ObjectId (ref: users),
  name: String,
  age: Number,
  gender: String,
  bio: String,
  location: {
    type: "Point",
    coordinates: [lng, lat] // [longitude, latitude]
  },
  photos: [String], // Array of S3 URLs
  preferences: {
    age_min: Number,
    age_max: Number,
    distance: Number, // in km
    gender: String
  }
}

// swipes collection
{
  _id: ObjectId,
  swiper_id: ObjectId (ref: users),
  swiped_id: ObjectId (ref: users),
  action: String, // 'like' or 'pass'
  created_at: Date
}

// matches collection
{
  _id: ObjectId,
  user1_id: ObjectId (ref: users),
  user2_id: ObjectId (ref: users),
  matched_at: Date,
  is_active: Boolean
}

// messages collection
{
  _id: ObjectId,
  match_id: ObjectId (ref: matches),
  sender_id: ObjectId (ref: users),
  content: String,
  message_type: String, // 'text', 'image', 'voice'
  is_read: Boolean,
  sent_at: Date
}
```

### Indexes for Performance

```javascript
// Geospatial index for location-based queries
db.profiles.createIndex({ location: "2dsphere" })

// Compound indexes for matching queries
db.swipes.createIndex({ swiper_id: 1, swiped_id: 1 }, { unique: true })
db.swipes.createIndex({ swiper_id: 1, created_at: -1 })

// Active users index
db.users.createIndex({ last_active: -1, is_verified: 1 })

// Match lookup index
db.matches.createIndex({ user1_id: 1, user2_id: 1 })
db.matches.createIndex({ user1_id: 1, is_active: 1 })

// Message history index
db.messages.createIndex({ match_id: 1, sent_at: -1 })
```

### Caching Strategy

```javascript
// Redis cache keys and TTL
User Profile: `profile:${userId}` - TTL: 1 hour
Discovery Feed: `discovery:${userId}` - TTL: 5 minutes
Match List: `matches:${userId}` - TTL: 30 seconds
Active Users Count: `stats:active` - TTL: 1 minute
```
