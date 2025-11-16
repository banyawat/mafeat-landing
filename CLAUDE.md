# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MAFEAT is a musician collaboration platform consisting of:
- **Frontend**: Nuxt.js 3 + Quasar UI + Tailwind CSS, deployed on AWS S3 + CloudFront
- **Backend**: Go Fiber API with clean architecture, deployed on Vercel (serverless)
- **Authentication**: AWS Cognito with Google, Facebook, and LINE federation
- **Database**: MongoDB Atlas
- **Maps**: MapLibre GL JS for interactive collaboration finder

## Key Architecture Patterns

### Frontend (Nuxt.js 3)
- Uses Quasar UI components with Tailwind CSS for styling
- MapLibre GL JS for the main collaboration map feature
- Authentication handled through AWS Cognito SDK
- File uploads use AWS S3 pre-signed URLs

### Backend (Go Fiber)
- Clean architecture with separate layers (handlers, services, repositories)
- JWT validation middleware for AWS Cognito tokens
- MongoDB with geo-indexing for location-based queries
- TTL indexes for collaboration posts (30-day expiration)

### Database Schema
- `users`: cognitoSub, email, name, avatarUrl, emailPrefs
- `collaborations`: userId, title, description, genre, location (Point), expiresAt
- `forum`: userId, title, body, category
- `comments`: forumId, userId, body

## Development Commands

This project appears to be in early development stage. Based on the tech stack:

### Frontend (Nuxt.js)
```bash
# Install dependencies
npm install

# Development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Linting
npm run lint
```

### Backend (Go)
```bash
# Run development server
go run main.go

# Build binary
go build -o mafeat-api

# Run tests
go test ./...

# Run tests with coverage
go test -cover ./...
```

## Core Features Implementation

### Authentication Flow
1. Frontend authenticates via AWS Cognito (social logins)
2. Backend validates JWT tokens using Cognito JWKs
3. User profiles created/updated in MongoDB on first login

### Map Collaboration Features
- Collaboration posts stored with GeoJSON Point coordinates
- Bounding box queries for map viewport: `GET /collaborations?bbox=lng1,lat1,lng2,lat2`
- 30-day TTL on collaboration posts
- Location privacy: coordinates rounded to ~100m precision

### Forum System
- Markdown support for posts and comments
- Categories: Collaboration, Music Theory, Production, Events
- Activity-based ordering (latest activity first)

### Email Notifications
- AWS SES for transactional emails
- User-controlled email preferences
- Collaboration contact forms

## Security Considerations

- HTTPS enforced everywhere
- Strict CORS (CloudFront origin only)
- Input validation and sanitization
- Rate limiting on content creation
- JWT middleware for API protection

## Infrastructure Notes

- Frontend: Static files on S3 + CloudFront CDN
- Backend: Serverless Go API on Vercel
- Database: MongoDB Atlas M10 cluster
- Storage: AWS S3 for avatar uploads (5MB limit)
- Email: AWS SES with verified domain

## Deployment

- Frontend deployed to AWS S3 + CloudFront
- Backend deployed to Vercel (serverless Go functions)
- Environment variables needed for AWS services and database connections