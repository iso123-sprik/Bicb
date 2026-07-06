# Système d'Authentification - GameWorld

## Vue d'ensemble

Le système d'authentification utilise :
- JWT (JSON Web Tokens) pour l'authentification
- Refresh tokens pour maintenir les sessions
- bcrypt pour le hachage des mots de passe
- OAuth2 optionnel pour les connexions sociales

## Architecture JWT

### Access Token

```
Header:
{
  "alg": "HS256",
  "typ": "JWT"
}

Payload:
{
  "sub": "user_id_uuid",
  "email": "user@example.com",
  "role": "competitor" | "supporter" | "admin",
  "iat": 1673869200,
  "exp": 1673872800,  // 1 heure
  "iss": "gameworld-api"
}

Signature: HS256(header + payload, secret)
```

### Refresh Token

```
Payload:
{
  "sub": "user_id_uuid",
  "type": "refresh",
  "iat": 1673869200,
  "exp": 1681645200,  // 90 jours
  "iss": "gameworld-api"
}
```

## Flux de Connexion

```
1. User Sign Up/Sign In
   ↓
2. Validation des credentials
   ↓
3. Génération Access Token (1h) + Refresh Token (90j)
   ↓
4. Stockage du Refresh Token en base de données
   ↓
5. Retour des tokens au client
   ↓
6. Client stocke les tokens (localStorage / sessionStorage)
```

## Utilisation des Tokens

### Pour chaque requête authentifiée:

```
GET /api/v1/users/me
Authorization: Bearer <access_token>
```

### Refresh Token Flow:

```
If Access Token expired:
  ↓
Client envoie Refresh Token
  ↓
Server valide le Refresh Token
  ↓
Server génère nouveau Access Token
  ↓
Client réutilise le nouveau Access Token
```

## Security Best Practices

### Password Hashing

```javascript
// Utiliser bcrypt avec salt rounds = 10+
const hashedPassword = await bcrypt.hash(password, 10);
```

### Token Storage (Client)

- **Access Token** : En mémoire (JavaScript) ou sessionStorage (moins sûr)
- **Refresh Token** : HttpOnly Cookie (sécurisé contre XSS)

### HTTPS Only

- Toutes les communications doivent être en HTTPS
- Cookies: `Secure` + `HttpOnly` + `SameSite=Strict`

### Validation

```javascript
// À chaque requête protégée:
1. Vérifier que le token est présent
2. Vérifier la signature du token
3. Vérifier que le token n'a pas expiré
4. Vérifier que l'utilisateur existe et est actif
5. Optionnel : Vérifier les permissions/roles
```

## Endpoints d'Authentification

### Sign Up

```
POST /auth/signup
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123!",
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
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "verification_required": true,
  "verification_email_sent": true
}
```

### Sign In

```
POST /auth/signin
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123!"
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
    "pulse_balance": 500.00
  }
}
```

### Refresh Token

```
POST /auth/refresh
Content-Type: application/json
Cookie: refresh_token=<refresh_token>

{}

Response (200):
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### Logout

```
POST /auth/logout
Authorization: Bearer <token>

Response (200):
{
  "message": "Logged out successfully"
}
```

### Verify Email

```
GET /auth/verify-email?token=<verification_token>

Response (200):
{
  "message": "Email verified successfully"
}
```

### Forgot Password

```
POST /auth/forgot-password

{
  "email": "user@example.com"
}

Response (200):
{
  "message": "Password reset link sent to email"
}
```

### Reset Password

```
POST /auth/reset-password

{
  "token": "<reset_token>",
  "new_password": "NewSecurePass123!"
}

Response (200):
{
  "message": "Password reset successfully"
}
```

## OAuth2 Integration (Optionnel)

### Google Sign In

```
GET /auth/google
  ↓
Redirige vers Google OAuth
  ↓
Google retourne authorization code
  ↓
Backend échange code contre tokens Google
  ↓
Crée ou met à jour l'utilisateur GameWorld
  ↓
Retourne JWT GameWorld au client
```

### Apple Sign In

```
POST /auth/apple

{
  "id_token": "<apple_id_token>"
}

Response (200):
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "...",
  "user": { ... }
}
```

## Token Rotation

```
Pour améliorer la sécurité :

1. Les refresh tokens doivent être rotés à chaque utilisation
2. Les anciens refresh tokens doivent être invalidés
3. Implémenter un système de "token families" pour détecter les abus
```

## CORS Configuration

```javascript
// Pour le développement
const corsOptions = {
  origin: ['http://localhost:3000', 'http://localhost:3001'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  maxAge: 3600
};

// Pour la production
const corsOptions = {
  origin: process.env.FRONTEND_URL,
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  maxAge: 86400
};
```

## Multi-Device Sessions

```
Chaque utilisateur peut avoir plusieurs sessions actives (mobile, web, etc.)
- Limiter à 5 sessions actives par utilisateur
- Permettre la révocation de sessions spécifiques
- Afficher la liste des appareils connectés
```

## Security Headers

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
```

## Rate Limiting sur Auth

```
- Sign Up: 5 tentatives / 15 minutes par IP
- Sign In: 5 tentatives échouées / 15 minutes par utilisateur
- Password Reset: 3 tentatives / 24 heures par utilisateur
- Email Verification: 10 tentatives / 24 heures par utilisateur
```

## 2FA (Two-Factor Authentication) - Optionnel

```
POST /auth/enable-2fa

Response (200):
{
  "secret": "JBSWY3DPEBLW64TMMQ======",
  "qr_code": "https://...",
  "backup_codes": ["code1", "code2", ...]
}

POST /auth/verify-2fa

{
  "code": "123456"
}

Response (200):
{
  "message": "2FA enabled successfully"
}
```
