# Sport ID Cards — Technical Documentation

**ada2ai SportNexus Passport**
Version 1.0.0 · Vision 2030 Aligned

---

## Overview

The Sport ID Card system issues digital, Nafath-verified identity cards to registered athletes across Saudi Arabia. Each card contains a complete athlete profile: personal identity, sport license, club affiliation, performance metrics, sport points tier, and a scannable QR code.

---

## File Structure

```
sportid-passport/
│
├── src/
│   ├── app/
│   │   └── id-card/
│   │       └── page.tsx          ← ID Card browser page (Next.js route)
│   │
│   ├── components/
│   │   └── SportIDCard.tsx       ← Flip card React component
│   │
│   └── data/
│       └── athlete_card.json     ← Athlete data + schema
│
├── backend/
│   ├── app.py                    ← Flask REST API
│   └── requirements.txt          ← Python dependencies
│
└── SPORT_ID_CARDS.md             ← This file
```

---

## React Component — `SportIDCard.tsx`

### Usage

```tsx
import SportIDCard from '@/components/SportIDCard';
import type { AthleteCard } from '@/components/SportIDCard';

<SportIDCard
  athlete={athleteData}   // AthleteCard object
  lang="en"               // 'en' | 'ar'
  className="mx-auto"     // optional Tailwind classes
/>
```

### Props

| Prop        | Type                  | Default | Description                      |
|-------------|-----------------------|---------|----------------------------------|
| `athlete`   | `AthleteCard`         | —       | Full athlete data object         |
| `lang`      | `'en' \| 'ar'`        | `'en'`  | Display language                 |
| `className` | `string`              | `''`    | Additional CSS classes           |

### Card Faces

| Face  | Content                                                              |
|-------|----------------------------------------------------------------------|
| Front | Photo, name (EN/AR), sport, club, ID, license status, points bar, QR |
| Back  | Performance stat rings, overall rating, season stats                 |

Click anywhere on the card to flip between front and back.

### AthleteCard Type

```typescript
interface AthleteCard {
  id: string;                         // e.g. "SA-ATH-2025-00142"
  personal: {
    full_name_en: string;
    full_name_ar: string;
    dob: string;                      // ISO 8601
    nationality: string;
    nafath_verified: boolean;
  };
  sport: {
    primary: string;                  // e.g. "Football"
    primary_ar: string;
    position: string;
    position_ar: string;
    federation: string;               // e.g. "SAFF"
    license_number: string;
    license_expiry: string;           // ISO 8601
    license_status: 'active' | 'expired' | 'suspended';
  };
  club: { current: string; current_ar: string; };
  performance: {
    overall_rating: number;           // 0–99
    speed: number;                    // 0–100
    stamina: number;
    technique: number;
    strength: number;
    vision: number;
    matches_played: number;
    goals: number;
    assists?: number;
    minutes_played: number;
  };
  sport_points: {
    total: number;
    tier: 'Bronze' | 'Silver' | 'Gold' | 'Elite';
    tier_ar: string;
    season_points: number;
  };
  card: {
    issued_date: string;
    expiry_date: string;
    card_color: string;               // hex
    accent_color: string;             // hex
    photo_url: string;
  };
}
```

---

## Flask API — `backend/app.py`

### Setup

```bash
cd sportid-passport/backend
pip install -r requirements.txt
python app.py
# → http://localhost:5001
```

### Environment Variables

| Variable            | Default            | Description                      |
|---------------------|--------------------|----------------------------------|
| `PORT`              | `5001`             | Server port                      |
| `FLASK_DEBUG`       | `true`             | Enable debug mode                |
| `SPORT_ID_API_KEY`  | `dev-key-replace-in-production` | API key for mutating endpoints |

### Endpoints

#### Athletes

| Method | Path                        | Auth | Description                   |
|--------|-----------------------------|------|-------------------------------|
| GET    | `/api/athletes`             | —    | List athletes (filterable)    |
| GET    | `/api/athletes/<id>`        | —    | Get full athlete record       |
| POST   | `/api/athletes`             | Key  | Create new athlete            |
| PUT    | `/api/athletes/<id>`        | Key  | Update athlete (partial)      |

**GET /api/athletes — Query Parameters**

| Param      | Type   | Description                        |
|------------|--------|------------------------------------|
| `sport`    | string | Filter by sport (e.g. `Football`)  |
| `tier`     | string | Filter by tier (`Gold`, `Elite`…)  |
| `club`     | string | Filter by club name                |
| `q`        | string | Search by name or ID               |
| `page`     | int    | Page number (default 1)            |
| `per_page` | int    | Results per page (default 20)      |

**Example Response — GET /api/athletes**

```json
{
  "data": [
    {
      "id": "SA-ATH-2025-00142",
      "name_en": "Mohammed Al-Qahtani",
      "name_ar": "محمد القحطاني",
      "sport": "Football",
      "club": "Al-Hilal FC",
      "tier": "Gold",
      "total_points": 4850,
      "rating": 87,
      "nafath": true,
      "license_status": "active"
    }
  ],
  "total": 1,
  "page": 1,
  "per_page": 20,
  "pages": 1
}
```

#### Nafath Verification

| Method | Path                  | Auth | Description             |
|--------|-----------------------|------|-------------------------|
| POST   | `/api/verify-nafath`  | —    | Verify national ID      |

**Request Body**
```json
{
  "national_id": "1XXXXXXXXX",
  "athlete_id":  "SA-ATH-2025-00142"
}
```

**Response (success)**
```json
{
  "verified":    true,
  "message":     "Identity successfully verified via Nafath",
  "national_id": "1XXXXXXXXX",
  "verified_at": "2025-01-15T10:30:00"
}
```

#### QR Code

| Method | Path              | Auth | Description              |
|--------|-------------------|------|--------------------------|
| GET    | `/api/qr/<id>`    | —    | Returns QR code PNG      |

```
GET /api/qr/SA-ATH-2025-00142?base_url=https://sportid.ada2ai.com
→ image/png  (QR encodes card URL)
```

#### Club Transfer

| Method | Path                    | Auth | Description               |
|--------|-------------------------|------|---------------------------|
| POST   | `/api/transfer`         | Key  | Initiate transfer request |
| GET    | `/api/transfer/<id>`    | —    | Get transfer status       |

**Transfer Request Body**
```json
{
  "athlete_id":   "SA-ATH-2025-00142",
  "from_club":    "Al-Hilal FC",
  "to_club":      "Al-Nassr FC",
  "transfer_fee": 500000,
  "currency":     "SAR"
}
```

**Transfer Status Steps**

| Step | Label                  | Description                             |
|------|------------------------|-----------------------------------------|
| 1    | Transfer Initiated     | Request created, ID assigned            |
| 2    | Federation Review      | SAFF/relevant federation reviews        |
| 3    | Club Agreement         | Both clubs sign digital agreement       |
| 4    | Ministry Registration  | Saudi Ministry of Sport registers move  |
| 5    | Card Updated           | Athlete's Sport Passport updated        |

#### Stats & Meta

| Method | Path               | Description                 |
|--------|--------------------|-----------------------------|
| GET    | `/api/stats`       | Platform-wide statistics    |
| GET    | `/api/federations` | List sport federations      |
| GET    | `/api/health`      | Health check                |

---

## JSON Data Schema — `athlete_card.json`

Top-level structure:

```json
{
  "schema_version": "1.0.0",
  "card_type":      "sport_id_passport",
  "athletes":       [ ...AthleteCard[] ],
  "tiers": [
    { "name": "Bronze", "min_points": 0,    "max_points": 999,  "color": "#CD7F32" },
    { "name": "Silver", "min_points": 1000, "max_points": 2999, "color": "#A8A9AD" },
    { "name": "Gold",   "min_points": 3000, "max_points": 5999, "color": "#FFD700" },
    { "name": "Elite",  "min_points": 6000, "max_points": null,  "color": "#00DCC8" }
  ],
  "federations": [ ...Federation[] ]
}
```

---

## Sport Points Tiers

| Tier   | Points Range | Color     | Key Privileges                                      |
|--------|-------------|-----------|-----------------------------------------------------|
| Bronze | 0 – 999     | `#CD7F32` | Basic analytics, club directory access              |
| Silver | 1,000–2,999 | `#A8A9AD` | Advanced stats, transfer eligibility, federation API |
| Gold   | 3,000–5,999 | `#FFD700` | Priority transfers, national team eligibility        |
| Elite  | 6,000+      | `#00DCC8` | International sync, Olympic pathway, Ministry access |

---

## Connecting Next.js to Flask

In `next.config.ts`, add an API proxy:

```typescript
// next.config.ts
const nextConfig = {
  async rewrites() {
    return [
      {
        source:      '/api/:path*',
        destination: 'http://localhost:5001/api/:path*',
      },
    ];
  },
};
export default nextConfig;
```

Then fetch from the frontend normally:

```typescript
const res = await fetch('/api/athletes/SA-ATH-2025-00142');
const { data } = await res.json();
```

---

## Security Considerations

- All mutating endpoints (`POST`, `PUT`) require `X-API-Key` header
- Nafath verification should use the official Saudi Nafath OAuth 2.0 API in production
- QR codes encode a public URL — no sensitive data embedded
- CORS is scoped to allowed origins only (`localhost:3000` + production domain)
- Replace `dev-key-replace-in-production` with a strong secret in `.env`

---

## License

Internal use · ada2ai Platform · Vision 2030 Initiative
© 2025 ada2ai. All rights reserved.
