# API REST Specification - GameWorld

## Base URL

```
https://api.gameworld.io/v1
```

## Authentication

Tous les endpoints (sauf sign-up/sign-in) requièrent un Bearer token JWT.

```
Authorization: Bearer <token>
```

## Endpoints

### Authentication

#### 1. Sign Up

```
POST /auth/signup

Body:
{
  "email": "user@example.com",
  "password": "SecurePassword123!",
  "first_name": "John",
  "last_name": "Doe",
  "date_of_birth": "1990-01-15",
  "country": "FR",
  "role": "competitor" | "supporter"
}

Response (201):
{
  "id": "uuid",
  "email": "user@example.com",
  "first_name": "John",
  "role": "competitor",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "pulse_balance": 0
}
```

#### 2. Sign In

```
POST /auth/signin

Body:
{
  "email": "user@example.com",
  "password": "SecurePassword123!"
}

Response (200):
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "first_name": "John",
    "role": "competitor",
    "pulse_balance": 500.00,
    "level": 5,
    "avatar_url": "https://..."
  }
}
```

#### 3. Refresh Token

```
POST /auth/refresh

Body:
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

Response (200):
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### Users

#### 1. Get Current User

```
GET /users/me

Response (200):
{
  "id": "uuid",
  "email": "user@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "avatar_url": "https://...",
  "bio": "Professional gamer",
  "country": "FR",
  "role": "competitor",
  "pulse_balance": 500.00,
  "total_earnings": 1500.00,
  "experience_points": 2500,
  "level": 5,
  "badges": ["champion_2024", "voting_expert"],
  "created_at": "2024-01-15T10:30:00Z"
}
```

#### 2. Update Profile

```
PUT /users/me

Body:
{
  "first_name": "John",
  "last_name": "Doe",
  "bio": "Professional gamer",
  "avatar_url": "https://...",
  "country": "FR"
}

Response (200):
{
  "id": "uuid",
  "first_name": "John",
  "bio": "Professional gamer",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

#### 3. Get User Profile

```
GET /users/{userId}

Response (200):
{
  "id": "uuid",
  "first_name": "John",
  "avatar_url": "https://...",
  "bio": "Professional gamer",
  "country": "FR",
  "level": 5,
  "experience_points": 2500,
  "badges": ["champion_2024"],
  "total_earnings": 1500.00,
  "created_at": "2024-01-15T10:30:00Z"
}
```

#### 4. Get Global Leaderboard

```
GET /users/leaderboard?period=monthly&limit=100&offset=0

Response (200):
{
  "period": "monthly",
  "total": 45000,
  "leaderboard": [
    {
      "rank": 1,
      "user_id": "uuid",
      "first_name": "Alice",
      "avatar_url": "https://...",
      "level": 12,
      "total_earnings": 5000.00,
      "experience_points": 10000
    }
  ]
}
```

### Competitions

#### 1. List Competitions

```
GET /competitions?category=gaming&status=open&limit=20&offset=0&sort=-start_date

Response (200):
{
  "total": 156,
  "competitions": [
    {
      "id": "uuid",
      "title": "Gaming Championship 2024",
      "category": "gaming",
      "image_url": "https://...",
      "banner_url": "https://...",
      "status": "open",
      "registration_fee": 10.00,
      "prize_pool": 10000.00,
      "current_participants": 250,
      "max_participants": 1000,
      "start_date": "2024-02-01T00:00:00Z",
      "end_date": "2024-02-15T23:59:59Z",
      "format": "live_stream" | "upload_video" | "upload_photo" | "text",
      "has_voting": true
    }
  ]
}
```

#### 2. Get Competition Details

```
GET /competitions/{competitionId}

Response (200):
{
  "id": "uuid",
  "title": "Gaming Championship 2024",
  "description": "...",
  "category": "gaming",
  "image_url": "https://...",
  "banner_url": "https://...",
  "status": "open",
  "registration_fee": 10.00,
  "prize_pool": 10000.00,
  "current_participants": 250,
  "max_participants": 1000,
  "start_date": "2024-02-01T00:00:00Z",
  "end_date": "2024-02-15T23:59:59Z",
  "format": "live_stream",
  "has_voting": true,
  "scoring_formula": {
    "jury_percentage": 60,
    "community_votes_percentage": 40
  },
  "rules": "...",
  "organizer": {
    "id": "uuid",
    "first_name": "GameWorld Team",
    "avatar_url": "https://..."
  },
  "phases": [
    {
      "id": "uuid",
      "phase_number": 1,
      "name": "Qualifications",
      "status": "ongoing",
      "start_date": "2024-02-01T00:00:00Z",
      "end_date": "2024-02-05T23:59:59Z",
      "current_round": 1,
      "total_rounds": 1
    }
  ],
  "rewards": {
    "1st": {"pulse": 10000, "money": 5000},
    "2nd": {"pulse": 6000, "money": 3000},
    "3rd": {"pulse": 3000, "money": 1500},
    "quarter_finalist": {"pulse": 1000},
    "eighth_finalist": {"pulse": 500},
    "group_stage": {"pulse": 200}
  }
}
```

#### 3. Register for Competition

```
POST /competitions/{competitionId}/register

Body:
{
  "submission_type": "pending" | "live_stream_url" | "video_url" | "image_url" | "text"
}

Response (201):
{
  "participant_id": "uuid",
  "competition_id": "uuid",
  "status": "registered",
  "payment_url": "https://checkout.stripe.com/...",
  "registered_at": "2024-01-15T10:30:00Z"
}
```

#### 4. Get Competition Participants / Rankings

```
GET /competitions/{competitionId}/participants?phase=1&sort=-total_score&limit=50

Response (200):
{
  "phase": 1,
  "total": 250,
  "participants": [
    {
      "id": "uuid",
      "user_id": "uuid",
      "first_name": "Alice",
      "avatar_url": "https://...",
      "submission_url": "https://...",
      "status": "qualified",
      "current_phase": 1,
      "jury_score": 85,
      "community_votes": 1250,
      "total_score": 86.5,
      "ranking": 1,
      "fan_team_members": 150
    }
  ]
}
```

#### 5. Submit Performance (Competitor)

```
POST /competitions/{competitionId}/submit

Body (based on format):
{
  "submission_type": "live_stream_url" | "video_upload" | "image_upload" | "text",
  "content": "https://twitch.tv/..." or file or text,
  "title": "My performance title",
  "description": "..."
}

Response (201):
{
  "submission_id": "uuid",
  "competition_id": "uuid",
  "participant_id": "uuid",
  "status": "submitted",
  "submission_url": "https://...",
  "submitted_at": "2024-01-15T10:30:00Z"
}
```

### Votes & Community

#### 1. Vote for Participant

```
POST /competitions/{competitionId}/participants/{participantId}/vote

Body:
{
  "vote_type": "free" | "pulse",
  "pulse_amount": 0 | 1-1000
}

Response (201):
{
  "vote_id": "uuid",
  "participant_id": "uuid",
  "voter_id": "uuid",
  "vote_type": "free" | "pulse",
  "pulse_spent": 0 | number,
  "created_at": "2024-01-15T10:30:00Z",
  "message": "Vote recorded successfully"
}
```

#### 2. Get Participant Vote Stats

```
GET /competitions/{competitionId}/participants/{participantId}/votes

Response (200):
{
  "participant_id": "uuid",
  "total_free_votes": 500,
  "total_pulse_votes": 750,
  "total_votes": 1250,
  "vote_trend": [50, 75, 120, 105, 90],
  "top_voters": [
    {
      "voter_id": "uuid",
      "first_name": "Bob",
      "avatar_url": "https://...",
      "total_votes_given": 15,
      "pulse_donated": 1000
    }
  ]
}
```

#### 3. Send Direct Donation (Pulse)

```
POST /competitions/{competitionId}/participants/{participantId}/donate

Body:
{
  "pulse_amount": 100,
  "message": "Go for it!"
}

Response (201):
{
  "donation_id": "uuid",
  "participant_id": "uuid",
  "donor_id": "uuid",
  "pulse_amount": 100,
  "message": "Go for it!",
  "created_at": "2024-01-15T10:30:00Z"
}
```

#### 4. Post Comment/Encouragement

```
POST /competitions/{competitionId}/participants/{participantId}/comments

Body:
{
  "message": "Amazing performance!"
}

Response (201):
{
  "comment_id": "uuid",
  "author_id": "uuid",
  "message": "Amazing performance!",
  "created_at": "2024-01-15T10:30:00Z"
}
```

#### 5. Join Fan Team

```
POST /competitions/{competitionId}/participants/{participantId}/fan-teams/join

Body:
{
  "fan_team_name": "Alice Supporters" (optional, auto-created if not exists)
}

Response (201):
{
  "fan_team_id": "uuid",
  "participant_id": "uuid",
  "user_id": "uuid",
  "fan_team_name": "Alice Supporters",
  "members_count": 150,
  "joined_at": "2024-01-15T10:30:00Z"
}
```

### Pulse & Wallet

#### 1. Get Wallet

```
GET /users/me/wallet

Response (200):
{
  "pulse_balance": 500.00,
  "currency": "EUR",
  "money_balance": 1500.00,
  "transactions": [
    {
      "id": "uuid",
      "type": "pulse_purchase" | "vote" | "donation" | "reward",
      "amount": 100,
      "currency": "pulse" | "EUR",
      "status": "completed",
      "description": "Voted for Alice",
      "created_at": "2024-01-15T10:30:00Z"
    }
  ]
}
```

#### 2. Buy Pulse

```
POST /users/me/wallet/buy-pulse

Body:
{
  "pulse_amount": 1000,
  "payment_method": "stripe"
}

Response (201):
{
  "transaction_id": "uuid",
  "pulse_amount": 1000,
  "eur_amount": 10.00,
  "payment_url": "https://checkout.stripe.com/...",
  "status": "pending",
  "created_at": "2024-01-15T10:30:00Z"
}
```

#### 3. Withdraw Money

```
POST /users/me/wallet/withdraw

Body:
{
  "amount": 1000.00,
  "payment_method": "bank_transfer",
  "bank_account_iban": "FR1420041010050500013M02606"
}

Response (201):
{
  "transaction_id": "uuid",
  "amount": 1000.00,
  "status": "pending",
  "payment_method": "bank_transfer",
  "created_at": "2024-01-15T10:30:00Z"
}
```

### Daily Challenges & Engagement

#### 1. Get Daily Challenges

```
GET /challenges/daily

Response (200):
{
  "challenges": [
    {
      "id": "uuid",
      "title": "Vote for 5 participants",
      "description": "Vote for 5 different participants today",
      "difficulty": "easy",
      "reward_pulse": 10,
      "reward_xp": 50,
      "completed": false,
      "progress": 3,
      "target": 5
    }
  ]
}
```

#### 2. Complete Challenge

```
POST /challenges/{challengeId}/complete

Body:
{}

Response (200):
{
  "challenge_id": "uuid",
  "completed": true,
  "reward_pulse": 10,
  "reward_xp": 50,
  "new_level": 5,
  "new_pulse_balance": 510.00,
  "completed_at": "2024-01-15T10:30:00Z"
}
```

### Rewards & Achievements

#### 1. Get User Rewards

```
GET /users/me/rewards

Response (200):
{
  "total_earned": 1500.00,
  "pending": 500.00,
  "claimed": 1000.00,
  "badges": [
    {
      "id": "uuid",
      "name": "Champion 2024",
      "icon_url": "https://...",
      "rarity": "epic",
      "earned_at": "2024-01-10T15:30:00Z"
    }
  ],
  "rewards": [
    {
      "id": "uuid",
      "competition_title": "Gaming Championship 2024",
      "ranking": 1,
      "prize_pulse": 10000,
      "prize_money": 5000,
      "status": "pending",
      "earned_at": "2024-01-10T15:30:00Z"
    }
  ]
}
```

#### 2. Claim Reward

```
POST /rewards/{rewardId}/claim

Body:
{}

Response (200):
{
  "reward_id": "uuid",
  "pulse_amount": 10000,
  "money_amount": 5000,
  "status": "claimed",
  "claimed_at": "2024-01-15T10:30:00Z",
  "new_wallet_balance": 10500.00
}
```

### Live Streaming (WebSocket)

#### Connect to Competition Live Feed

```
WebSocket: wss://api.gameworld.io/v1/competitions/{competitionId}/live

Subscribe to:
{
  "type": "subscribe",
  "channel": "competition",
  "filters": {
    "phase": 1,
    "event_types": ["vote", "donation", "submission", "phase_complete"]
  }
}

Events received:
{
  "type": "vote",
  "participant_id": "uuid",
  "voter_id": "uuid",
  "vote_type": "free" | "pulse",
  "pulse_amount": 100,
  "total_votes": 1251,
  "timestamp": "2024-01-15T10:30:00Z"
}

{
  "type": "donation",
  "participant_id": "uuid",
  "donor_id": "uuid",
  "pulse_amount": 500,
  "message": "Go for it!",
  "timestamp": "2024-01-15T10:30:00Z"
}

{
  "type": "phase_complete",
  "phase": 1,
  "qualified_participants": ["uuid1", "uuid2", ...],
  "eliminated_participants": ["uuid3", ...],
  "rewards_distributed": true,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

## Error Responses

### 400 Bad Request

```json
{
  "error": "VALIDATION_ERROR",
  "message": "Validation failed",
  "details": [
    {
      "field": "email",
      "message": "Invalid email format"
    }
  ]
}
```

### 401 Unauthorized

```json
{
  "error": "UNAUTHORIZED",
  "message": "Invalid or expired token"
}
```

### 403 Forbidden

```json
{
  "error": "FORBIDDEN",
  "message": "You don't have permission to access this resource"
}
```

### 429 Too Many Requests

```json
{
  "error": "RATE_LIMIT_EXCEEDED",
  "message": "Too many requests. Please try again later.",
  "retry_after": 60
}
```

## Pagination

Tous les endpoints de liste supportent la pagination :

- `limit` : Nombre d'éléments par page (défaut: 20, max: 100)
- `offset` : Décalage (défaut: 0)
- `sort` : Tri (`field` ou `-field` pour descendant)

## Rate Limiting

- Utilisateurs authentifiés: 100 requêtes/minute
- Utilisateurs non authentifiés: 10 requêtes/minute

Limites spéciales :
- Votes: 50 votes/heure par utilisateur
- Donations: 100 donations/jour par utilisateur
- Registrations: 1 registration/compétition
- API générale: 10 000 requêtes/jour par utilisateur
