# Core Features

## MVP Features (Week 1-4)

### 1. Authentication
- Phone number or email sign-up
- SMS OTP verification
- Basic login/logout

### 2. Profile Management
- Upload 3-6 photos
- Basic bio (300 characters)
- Essential details: Name, Age, Gender, Location
- Interest in: Gender preference
- Distance preference (5-100 km)
- Age range preference (18-60)

### 3. Discovery/Swiping
- Card-based swipe interface
- Swipe right (Like) / Swipe left (Pass)
- Show profiles based on:
  - Location proximity (MongoDB geospatial queries)
  - Age preference
  - Gender preference
- Daily limit: 50 swipes for free users

### 4. Matching
- Match occurs when both users like each other
- Match notification
- Match list view

### 5. Chat
- Text messaging between matches only
- Real-time message delivery (WebSocket)
- Message history
- Basic typing indicator
- Unmatch option

### 6. Settings
- Edit profile
- Update preferences
- Delete account
- Logout
