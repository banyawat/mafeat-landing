# **🎵 Project Title**

**MAFEAT — Musician Collaboration & Forum Platform**

---

## **🧭 Project Overview**

**MAFEAT** is a web platform for musicians to **discover, connect, and collaborate** with others nearby.

Users can drop collaboration posts on an interactive map and participate in a **community forum** for discussion and learning.

The goal is to make it easy for musicians to **find collaborators by location, genre, and intent** — all in one simple, modern platform.

---

## **🎯 MVP 1 Scope**

### **Core Features**

### **1. Authentication (AWS Cognito)**

- Authentication handled via **AWS Cognito User Pool**.
- Federated logins supported:
  - **Google**
  - **Facebook**
  - **LINE**
- Store user profile data:
  - Cognito sub (unique ID)
  - Name
  - Email
  - Avatar URL (optional)
- Backend validates Cognito JWT tokens for API calls.

---

### **2. Main Map Page (Collaboration Finder)**

- Built with **MapLibre GL JS**.
- Users can view nearby collaboration posts displayed as map pins.
- Cluster markers for nearby posts.
- Each post popup shows:
  - Title
  - Genre
  - Description snippet
  - Link to full detail view.
- Users can **create a collaboration post** by clicking on the map and filling:
  - **Title** (string)
  - **Description** (text)
  - **Genre** (string or select list)
  - **Location** (latitude, longitude from map pin)
- Posts are stored with coordinates (Point) and automatically expire after 30 days.

---

### **3. Forum**

- A musician community discussion board.
- Features:
  - Create new post (title + body, Markdown or rich text).
  - Comment on existing posts.
  - Filter or browse by categories (e.g., “Collaboration,” “Music Theory,” “Production,” “Events”).
- Basic moderation:
  - Users can delete or edit their own posts/comments.
  - Optional admin delete endpoint for MVP.

---

### **4. Notifications & Contact**

- “Contact” button on collaboration posts.
- Sends **email via AWS SES** to post owner.
- Email includes sender info and post title.
- User can enable/disable notifications in **Settings**:
  - Receive collaboration emails ✅
  - Receive news/promotional emails ✅

---

### **5. Settings Page**

- Update display name and avatar.
- Manage email notification preferences.
- Logout (clears Cognito session).

---

### **6. User Profiles**

- Public profile page for each user displaying:
  - Display name and avatar
  - Member since date (join date)
  - Activity statistics:
    - Total collaborations
    - Total forum posts
    - Total comments
  - Recent collaborations (last 10)
  - Recent forum posts (last 10)
- Clickable user names/avatars throughout the app:
  - Map collaboration detail popups
  - Forum post list cards
  - Forum post detail page (author section)
  - Comment sections
- Profile privacy:
  - Email addresses are NOT displayed publicly
  - Only public activity and statistics are shown
  - Users cannot edit other users' profiles

---

## **🧱 Tech Stack**

| **Layer**            | **Technology**                                  |
| -------------------- | ----------------------------------------------- |
| **Frontend**         | Nuxt.js 3 + Quasar UI + Tailwind CSS            |
| **Map Engine**       | [MapLibre GL JS](https://maplibre.org/)         |
| **Backend**          | Golang (Fiber framework) — Clean Architecture   |
| **Authentication**   | AWS Cognito (Google, Facebook, LINE federation) |
| **Database**         | MongoDB Atlas                                   |
| **Storage**          | AWS S3 (for avatars)                            |
| **Email Service**    | AWS SES                                         |
| **Frontend Hosting** | AWS S3 + CloudFront                             |
| **Backend Hosting**  | Vercel (Serverless Go API)                      |

---

**🧩 System Architecture**

```bash
Nuxt 3 (S3 + CloudFront)
       │
       ▼
Vercel (Go Fiber API)
       │
 ┌──────────────┬──────────────┬──────────────┬──────────────┐
 │              │              │              │              │
▼              ▼              ▼              ▼              ▼
MongoDB Atlas  AWS S3  AWS SES  AWS Cognito  MapLibre Tiles

```

---

**🗺️ Database Collections (Schemas)
users**

```bash
{
  "_id": "ObjectId",
  "cognitoSub": "string",
  "email": "string",
  "name": "string",
  "avatarUrl": "string",
  "emailPrefs": { "collab": true, "news": false },
  "createdAt": "date",
  "updatedAt": "date"
}
```

**collaborations**

```bash
{
  "_id": "ObjectId",
  "userId": "ObjectId",
  "title": "string",
  "description": "string",
  "genre": "string",
  "location": { "type": "Point", "coordinates": [lng, lat] },
  "expiresAt": "date",
  "createdAt": "date"
}
```

**comments**

```bash
{
  "_id": "ObjectId",
  "forumId": "ObjectId",
  "userId": "ObjectId",
  "body": "string",
  "createdAt": "date"
}
```

---

**⚙️ API Endpoints (v1)**

| **Method** | **Path**              | **Description**                                                |
| ---------- | --------------------- | -------------------------------------------------------------- |
| GET        | /auth/me              | Verify Cognito JWT and return user profile                     |
| PATCH      | /me                   | Update profile & email preferences                             |
| GET        | /collaborations?bbox= | Get collaboration posts by bounding box                        |
| POST       | /collaborations       | Create collaboration post (title, description, genre, lat/lon) |
| GET        | /collaborations/:id   | Get single collaboration post                                  |
| DELETE     | /collaborations/:id   | Delete own collaboration post                                  |
| POST       | /contact              | Send collaboration inquiry email (via SES)                     |
| GET        | /forum                | List forum posts                                               |
| POST       | /forum                | Create new forum post                                          |
| GET        | /forum/:id            | Get forum post + comments                                      |
| POST       | /forum/:id/comment    | Add comment                                                    |
| DELETE     | /forum/:id            | Delete own post                                                |
| POST       | /uploads/s3-presign   | Get pre-signed URL for avatar upload                           |
| GET        | /users/:id            | Get public user profile with activity stats                    |

---

## **🧰 Infrastructure Configuration**

### **AWS Cognito**

- Cognito User Pool for authentication.
- Federated identity providers:
  - Google, Facebook (OAuth)
  - LINE (OIDC)
- Tokens validated via Cognito JWKs in backend.
- Callback URLs configured for both frontend and backend.

---

### **AWS S3**

- Bucket: mafeat-uploads
- Public read via CloudFront.
- Used for avatar uploads.
- 5MB upload limit with pre-signed URLs.

---

### **AWS SES**

- Verified domain (e.g. mafeat.com)
- DKIM + SPF setup.
- Email templates:
  - Collaboration Notification
  - Newsletter (future use)

---

**Deployment**

| **Component**  | **Platform**        | **Notes**                       |
| -------------- | ------------------- | ------------------------------- |
| **Frontend**   | AWS S3 + CloudFront | Static Nuxt 3 site              |
| **Backend**    | Vercel              | Go Fiber API (serverless)       |
| **Database**   | MongoDB Atlas       | M10 cluster                     |
| **Map Engine** | MapLibre            | Open source vector map renderer |

## **🔒 Security Requirements**

- HTTPS enforced everywhere.
- Cognito JWT validation middleware.
- Strict CORS policy (CloudFront origin only).
- Rate limiting on post & comment creation.
- Input validation for all forms.
- Safe Markdown rendering (forum).
- Sanitized user text fields.
- Location rounding (~100m precision) for privacy.

## **🧠 Smart Behaviors**

- Collaboration posts expire after 30 days (TTL index).
- SES emails only sent if emailPrefs.collab = true.
- Forum posts ordered by latest activity.
- Markdown rendering for forum posts/comments.
- Random coordinate offset for privacy on map.
- Automatic logout on invalid JWT.

## **✅ Acceptance Criteria**

- Login via AWS Cognito (Google, Facebook, LINE).
- MapLibre map displays collaboration pins.
- Users can create posts with **title, description, genre, and location**.
- Collaboration markers cluster and open popups.
- Forum supports posts and comments.
- SES email notifications work for collaboration.
- Settings page updates email preferences.
- Fully deployed: CloudFront (frontend) + Vercel (backend).
- Mobile responsive UI.
