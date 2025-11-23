# User Flow

<img src="../../diagram/user-flow.svg"/>
<details>
<summary>Click to view User Flow Diagram</summary>

```mermaid
flowchart TD
    Start((Start)) --> SignUp[Sign Up Phone/Email]
    SignUp --> OTP[OTP Verification]
    OTP --> Profile[Create Profile]
    Profile --> Prefs[Set Preferences]
    Prefs --> Feed[Discovery Feed]
    
    Feed --> Swipe{Swipe?}
    Swipe -- Right --> Like[Like]
    Swipe -- Left --> Pass[Pass]
    
    Like --> Match{Match?}
    Match -- Yes --> Notification[Match Notification]
    Match -- No --> Feed
    Pass --> Feed
    
    Notification --> Chat[Chat]
    Chat --> Unmatch{Unmatch?}
    Unmatch -- Yes --> Feed
    Unmatch -- No --> Chat
```
</details>

## Complete User Journey

1. **Download App** → Sign up with phone/email
2. **OTP Verification** → Verify identity
3. **Create Profile** → Add photos, bio, and details
4. **Set Preferences** → Age range, distance, gender
5. **Discovery Feed** → Swipe on profiles
6. **Match** → When both users like each other
7. **Chat** → Send messages to matches
8. **Continue or Unmatch** → Keep chatting or remove match
