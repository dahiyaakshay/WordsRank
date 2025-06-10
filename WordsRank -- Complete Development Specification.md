# WordsRank - AI Platform Monitoring Tool

## Table of Contents
1. Project Overview
2. System Architecture
3. Technology Stack
4. Database Design
5. Backend API Architecture
6. Frontend Application Structure
7. AI Platform Integrations
8. User Interface Specifications
9. Authentication & Security
10. Development Setup
11. Testing Strategy
12. Deployment & DevOps

## Project Overview
### Business Purpose
WordsRank is an AI platform monitoring tool that tracks how brands and competitors appear in AI-powered search responses across ChatGPT, Perplexity, Gemini, Claude, Mistral AI, and DeepSeek. It provides actionable insights for content strategy and competitive intelligence.

### Core Value Proposition
- Monitor Brand Visibility: Track your brand mentions across 6 major AI platforms
- Competitive Intelligence: Understand how competitors rank in AI responses
- Content Gap Analysis: Identify opportunities to improve AI platform visibility
- Actionable Insights: Get clear recommendations for content strategy

### Target Users
- Marketing managers at tech companies
- Content strategists and SEO professionals
- Brand managers monitoring online presence
- Competitive intelligence analysts

## Key Features (MVP + Mid-Level)
1. Dashboard Analytics: Visual overview of brand performance across AI platforms
2. AI Platform Monitoring: Individual tracking for each AI platform
3. Search Terms Management: Monitor specific keywords and brand terms
4. Competitive Analysis: Track competitor performance and identify gaps
5. Website & Brand Management: Configure brands and websites to monitor
6. Team Collaboration: Multi-user access with role-based permissions
7. Reporting & Exports: Generate reports in multiple formats

## System Architecture
### High-Level Architecture
<pre>┌─────────────────────────────────────────────────────────────┐
│                    WordsRank Application                    │
├─────────────────────────────────────────────────────────────┤
│  Frontend (React + TypeScript)                             │
│  ├── Dashboard & Analytics                                 │
│  ├── AI Platform Pages                                     │
│  ├── Management Pages                                      │
│  └── Authentication                                        │
├─────────────────────────────────────────────────────────────┤
│  Backend API (Node.js + Express)                           │
│  ├── Authentication Service                                │
│  ├── AI Platform Services                                  │
│  ├── Analytics & Reporting                                 │
│  └── Data Processing                                       │
├─────────────────────────────────────────────────────────────┤
│  Database Layer (PostgreSQL)                               │
│  ├── User & Team Management                                │
│  ├── Website & Search Terms                                │
│  ├── AI Platform Responses                                 │
│  └── Analytics Data                                        │
├─────────────────────────────────────────────────────────────┤
│  AI Platform APIs                                          │
│  ├── OpenAI API (ChatGPT)                                  │
│  ├── Anthropic API (Claude)                                │
│  ├── Google AI API (Gemini)                                │
│  ├── Perplexity API                                        │
│  ├── Mistral AI API                                        │
│  └── DeepSeek API                                          │
└─────────────────────────────────────────────────────────────┘</pre>

### Data Flow
```
Search Terms → AI Platform APIs → Response Analysis → 
Position Detection → Database Storage → Analytics Processing → 
Dashboard Updates → User Interface
```

## Technology Stack
### Backend Stack
```json
{
  "runtime": "Node.js 18+",
  "framework": "Express.js 4.18+",
  "database": "PostgreSQL 15+",
  "authentication": "JWT + bcrypt",
  "validation": "Joi",
  "logging": "Winston",
  "testing": "Jest + Supertest",
  "scheduling": "node-cron",
  "http": "axios"
}
```

### Frontend Stack
```json
{
  "framework": "React 18+ with TypeScript",
  "routing": "React Router 6+",
  "styling": "Tailwind CSS",
  "charts": "Recharts",
  "forms": "React Hook Form + Zod",
  "state": "Zustand",
  "http": "axios",
  "build": "Vite",
  "testing": "Vitest + React Testing Library"
}
```

### AI Platform SDKs
```json
{
  "openai": "openai@4.0+",
  "anthropic": "@anthropic-ai/sdk",
  "google": "@google-ai/generativelanguage",
  "mistral": "@mistralai/mistralai",
  "perplexity": "Custom HTTP client",
  "deepseek": "Custom HTTP client"
}
```

## Database Design
### PostgreSQL Schema
#### Core Tables
```sql
-- Users and Authentication
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(255) NOT NULL,
    company VARCHAR(255),
    role VARCHAR(50) DEFAULT 'member', -- 'owner', 'admin', 'member'
    plan VARCHAR(50) DEFAULT 'free', -- 'free', 'premium', 'enterprise'
    plan_queries_limit INTEGER DEFAULT 1000,
    plan_queries_used INTEGER DEFAULT 0,
    plan_renewal_date DATE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Teams and Team Memberships
CREATE TABLE teams (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    owner_id UUID REFERENCES users(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE team_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID REFERENCES teams(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    role VARCHAR(50) DEFAULT 'member', -- 'admin', 'member'
    joined_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(team_id, user_id)
);

-- Websites to monitor
CREATE TABLE websites (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    url VARCHAR(500) NOT NULL,
    industry VARCHAR(100),
    description TEXT,
    is_active BOOLEAN DEFAULT true,
    platforms_monitored TEXT[] DEFAULT '{}', -- ['chatgpt', 'perplexity', etc.]
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Competitor websites
CREATE TABLE competitors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    website_id UUID REFERENCES websites(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    url VARCHAR(500) NOT NULL,
    is_active BOOLEAN DEFAULT true,
    platforms_monitored TEXT[] DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Search terms to monitor
CREATE TABLE search_terms (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    website_id UUID REFERENCES websites(id) ON DELETE CASCADE,
    term TEXT NOT NULL,
    platforms_monitored TEXT[] DEFAULT '{}', -- Which platforms to check
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- AI platform API configurations
CREATE TABLE ai_platform_configs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    platform VARCHAR(50) NOT NULL, -- 'openai', 'anthropic', etc.
    api_key_encrypted TEXT,
    is_active BOOLEAN DEFAULT false,
    last_tested TIMESTAMP,
    test_status VARCHAR(20), -- 'connected', 'invalid', 'error'
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(user_id, platform)
);

-- AI platform responses and mentions
CREATE TABLE ai_mentions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    search_term_id UUID REFERENCES search_terms(id) ON DELETE CASCADE,
    platform VARCHAR(50) NOT NULL,
    query_text TEXT NOT NULL,
    response_text TEXT NOT NULL,
    position INTEGER, -- 1-10 ranking position, NULL if not mentioned
    sentiment VARCHAR(20), -- 'positive', 'negative', 'neutral'
    confidence_score DECIMAL(3,2) DEFAULT 0.5,
    citations TEXT[], -- Array of URLs cited
    response_length INTEGER,
    processing_time INTEGER, -- milliseconds
    api_cost DECIMAL(10,6) DEFAULT 0,
    queried_at TIMESTAMP DEFAULT NOW(),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Analytics snapshots for performance tracking
CREATE TABLE analytics_snapshots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    website_id UUID REFERENCES websites(id) ON DELETE CASCADE,
    snapshot_date DATE DEFAULT CURRENT_DATE,
    platform VARCHAR(50) NOT NULL,
    total_terms_monitored INTEGER DEFAULT 0,
    terms_ranking_top3 INTEGER DEFAULT 0,
    terms_ranking_4to10 INTEGER DEFAULT 0,
    terms_not_mentioned INTEGER DEFAULT 0,
    average_position DECIMAL(3,1),
    sentiment_positive INTEGER DEFAULT 0,
    sentiment_negative INTEGER DEFAULT 0,
    sentiment_neutral INTEGER DEFAULT 0,
    total_mentions INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(website_id, snapshot_date, platform)
);

-- Scheduled monitoring jobs
CREATE TABLE monitoring_jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    job_type VARCHAR(50) NOT NULL, -- 'daily_monitoring', 'weekly_report'
    frequency VARCHAR(20) NOT NULL, -- 'daily', 'weekly', 'monthly'
    is_active BOOLEAN DEFAULT true,
    last_run TIMESTAMP,
    next_run TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### Indexes for Performance
```sql
-- Performance indexes
CREATE INDEX idx_ai_mentions_search_term_platform ON ai_mentions(search_term_id, platform);
CREATE INDEX idx_ai_mentions_queried_at ON ai_mentions(queried_at DESC);
CREATE INDEX idx_analytics_snapshots_website_date ON analytics_snapshots(website_id, snapshot_date DESC);
CREATE INDEX idx_search_terms_website_active ON search_terms(website_id, is_active);
CREATE INDEX idx_team_members_user ON team_members(user_id);
```

## Backend API Architecture
### Project Structure
<pre>backend/
├── src/
│   ├── controllers/
│   │   ├── auth.controller.js
│   │   ├── dashboard.controller.js
│   │   ├── websites.controller.js
│   │   ├── competitors.controller.js
│   │   ├── searchTerms.controller.js
│   │   ├── analytics.controller.js
│   │   ├── platforms.controller.js
│   │   └── team.controller.js
│   ├── services/
│   │   ├── ai-platforms/
│   │   │   ├── base.service.js
│   │   │   ├── openai.service.js
│   │   │   ├── anthropic.service.js
│   │   │   ├── google.service.js
│   │   │   ├── perplexity.service.js
│   │   │   ├── mistral.service.js
│   │   │   └── deepseek.service.js
│   │   ├── auth.service.js
│   │   ├── analytics.service.js
│   │   ├── monitoring.service.js
│   │   └── scheduler.service.js
│   ├── models/
│   │   ├── User.js
│   │   ├── Website.js
│   │   ├── SearchTerm.js
│   │   └── AIMention.js
│   ├── middleware/
│   │   ├── auth.middleware.js
│   │   ├── validation.middleware.js
│   │   └── rateLimit.middleware.js
│   ├── routes/
│   │   ├── auth.routes.js
│   │   ├── dashboard.routes.js
│   │   ├── websites.routes.js
│   │   ├── analytics.routes.js
│   │   └── platforms.routes.js
│   ├── utils/
│   │   ├── database.js
│   │   ├── encryption.js
│   │   ├── logger.js
│   │   └── helpers.js
│   └── app.js
├── config/
│   ├── database.config.js
│   └── app.config.js
├── migrations/
├── tests/
└── package.json</pre>


### Core API Endpoints
#### Authentication & User Management
```javascript
// Authentication
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout
GET    /api/auth/me
PUT    /api/auth/profile
PUT    /api/auth/password

// Team Management
GET    /api/team/members
POST   /api/team/invite
PUT    /api/team/member/:id/role
DELETE /api/team/member/:id
```

#### Core Data Management
```javascript
// Websites
GET    /api/websites
POST   /api/websites
PUT    /api/websites/:id
DELETE /api/websites/:id

// Competitors
GET    /api/competitors/:websiteId
POST   /api/competitors
PUT    /api/competitors/:id
DELETE /api/competitors/:id

// Search Terms
GET    /api/search-terms/:websiteId
POST   /api/search-terms
PUT    /api/search-terms/:id
DELETE /api/search-terms/:id
POST   /api/search-terms/bulk-import
```

#### AI Platform Management
```javascript
// Platform Configuration
GET    /api/platforms/config
PUT    /api/platforms/config/:platform
POST   /api/platforms/test/:platform

// Platform Analytics
GET    /api/platforms/:platform/analytics
GET    /api/platforms/:platform/mentions
POST   /api/platforms/query
```

#### Analytics & Reporting
```javascript
// Dashboard Analytics
GET    /api/analytics/dashboard
GET    /api/analytics/overview/:websiteId
GET    /api/analytics/rankings/:websiteId
GET    /api/analytics/competitors/:websiteId

// Reports
GET    /api/reports/generate/:type
POST   /api/reports/schedule
```

### Service Layer Implementation
#### Base AI Platform Service
```javascript
// services/ai-platforms/base.service.js
class BaseAIService {
  constructor(platform, apiKey) {
    this.platform = platform;
    this.apiKey = apiKey;
  }

  async queryPlatform(searchTerm, context = {}) {
    throw new Error('queryPlatform method must be implemented');
  }

  extractPosition(content, brandTerms) {
    // Common position extraction logic
    const sentences = content.split(/[.!?]+/);
    
    for (let i = 0; i < sentences.length; i++) {
      const sentence = sentences[i].toLowerCase();
      if (brandTerms.some(term => sentence.includes(term.toLowerCase()))) {
        if (i === 0) return 1;
        if (i <= 2) return 2;
        if (i <= 4) return 3;
        return Math.min(10, Math.ceil((i + 1) / 2));
      }
    }
    return null;
  }

  extractCitations(content) {
    const urlRegex = /https?:\/\/[^\s\)]+/g;
    const urls = content.match(urlRegex) || [];
    return urls.map(url => url.replace(/[.,;!?]$/, ''));
  }

  analyzeSentiment(content, brandTerms) {
    // Simple sentiment analysis
    const positiveWords = ['excellent', 'great', 'best', 'outstanding', 'superior', 'innovative'];
    const negativeWords = ['poor', 'bad', 'worst', 'terrible', 'inferior', 'disappointing'];
    
    const words = content.toLowerCase().split(/\s+/);
    let positiveScore = 0;
    let negativeScore = 0;
    
    words.forEach(word => {
      if (positiveWords.includes(word)) positiveScore++;
      if (negativeWords.includes(word)) negativeScore++;
    });
    
    if (positiveScore > negativeScore) return 'positive';
    if (negativeScore > positiveScore) return 'negative';
    return 'neutral';
  }

  calculateConfidence(content, position) {
    let confidence = 0.5;
    
    if (content.length > 100) confidence += 0.2;
    if (position && position <= 3) confidence += 0.2;
    if (content.includes('http')) confidence += 0.1;
    
    return Math.min(0.95, confidence);
  }
}

module.exports = BaseAIService;
```

#### OpenAI Service Implementation
```javascript
// services/ai-platforms/openai.service.js
const OpenAI = require('openai');
const BaseAIService = require('./base.service');

class OpenAIService extends BaseAIService {
  constructor(apiKey) {
    super('chatgpt', apiKey);
    this.client = new OpenAI({ apiKey });
    this.model = 'gpt-4';
  }

  async queryPlatform(searchTerm, context = {}) {
    const startTime = Date.now();
    
    try {
      const response = await this.client.chat.completions.create({
        model: this.model,
        messages: [
          {
            role: 'system',
            content: 'You are a helpful assistant providing comprehensive information about user queries. Include specific brands, companies, and products when relevant.'
          },
          {
            role: 'user',
            content: searchTerm
          }
        ],
        max_tokens: 1000,
        temperature: 0.7
      });

      const content = response.choices[0]?.message?.content || '';
      const brandTerms = context.brandTerms || [];
      
      return {
        platform: this.platform,
        query: searchTerm,
        response: content,
        position: this.extractPosition(content, brandTerms),
        sentiment: this.analyzeSentiment(content, brandTerms),
        confidence: this.calculateConfidence(content, this.extractPosition(content, brandTerms)),
        citations: this.extractCitations(content),
        responseLength: content.length,
        processingTime: Date.now() - startTime,
        tokensUsed: response.usage?.total_tokens || 0,
        apiCost: this.calculateCost(response.usage?.total_tokens || 0)
      };
    } catch (error) {
      throw new Error(`OpenAI API Error: ${error.message}`);
    }
  }

  calculateCost(tokens) {
    // GPT-4 pricing: ~$0.03 per 1K tokens (mixed input/output)
    return (tokens / 1000) * 0.03;
  }
}

module.exports = OpenAIService;
```

#### Monitoring Service
```javascript
// services/monitoring.service.js
const { AIServiceFactory } = require('./ai-platforms');
const { SearchTerm, Website, AIMention } = require('../models');

class MonitoringService {
  constructor() {
    this.aiServiceFactory = new AIServiceFactory();
  }

  async runMonitoringForUser(userId) {
    try {
      const websites = await Website.findByUserId(userId);
      
      for (const website of websites) {
        await this.monitorWebsite(website);
      }
    } catch (error) {
      console.error(`Monitoring error for user ${userId}:`, error);
    }
  }

  async monitorWebsite(website) {
    const searchTerms = await SearchTerm.findByWebsiteId(website.id);
    const brandTerms = [website.name, ...website.name.split(' ')];
    
    for (const searchTerm of searchTerms) {
      for (const platform of searchTerm.platforms_monitored) {
        await this.queryPlatform(searchTerm, platform, brandTerms);
        
        // Rate limiting delay
        await this.delay(2000);
      }
    }
  }

  async queryPlatform(searchTerm, platform, brandTerms) {
    try {
      const aiService = this.aiServiceFactory.getService(platform);
      if (!aiService) return;

      const result = await aiService.queryPlatform(searchTerm.term, { brandTerms });
      
      // Store result in database
      await AIMention.create({
        search_term_id: searchTerm.id,
        platform: result.platform,
        query_text: result.query,
        response_text: result.response,
        position: result.position,
        sentiment: result.sentiment,
        confidence_score: result.confidence,
        citations: result.citations,
        response_length: result.responseLength,
        processing_time: result.processingTime,
        api_cost: result.apiCost
      });

      return result;
    } catch (error) {
      console.error(`Platform query error for ${platform}:`, error);
      return null;
    }
  }

  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

module.exports = MonitoringService;
```

#### Analytics Service
```javascript
// services/analytics.service.js
const { AIMention, AnalyticsSnapshot } = require('../models');

class AnalyticsService {
  async getDashboardAnalytics(websiteId, timeframe = '30d') {
    const timeframeDays = this.parseTimeframe(timeframe);
    const startDate = new Date();
    startDate.setDate(startDate.getDate() - timeframeDays);

    const mentions = await AIMention.findByWebsiteAndTimeframe(websiteId, startDate);
    
    return {
      overview: await this.calculateOverviewStats(mentions),
      platformPerformance: await this.calculatePlatformPerformance(mentions),
      rankingTrends: await this.calculateRankingTrends(websiteId, timeframeDays),
      sentimentAnalysis: await this.calculateSentimentAnalysis(mentions),
      competitiveAnalysis: await this.calculateCompetitiveAnalysis(websiteId, startDate)
    };
  }

  async calculateOverviewStats(mentions) {
    const totalMentions = mentions.length;
    const mentionsWithPosition = mentions.filter(m => m.position !== null);
    const top3Mentions = mentions.filter(m => m.position && m.position <= 3);
    const averagePosition = mentionsWithPosition.length > 0 
      ? mentionsWithPosition.reduce((sum, m) => sum + m.position, 0) / mentionsWithPosition.length 
      : null;

    return {
      totalMentions,
      visibilityScore: totalMentions > 0 ? (mentionsWithPosition.length / totalMentions) * 100 : 0,
      averagePosition: averagePosition ? Math.round(averagePosition * 10) / 10 : null,
      top3Rankings: top3Mentions.length,
      platformCoverage: [...new Set(mentionsWithPosition.map(m => m.platform))].length
    };
  }

  async calculatePlatformPerformance(mentions) {
    const platformStats = {};
    
    mentions.forEach(mention => {
      if (!platformStats[mention.platform]) {
        platformStats[mention.platform] = {
          totalMentions: 0,
          positionedMentions: 0,
          top3Count: 0,
          averagePosition: 0,
          sentimentPositive: 0,
          sentimentNegative: 0,
          sentimentNeutral: 0
        };
      }
      
      const stats = platformStats[mention.platform];
      stats.totalMentions++;
      
      if (mention.position) {
        stats.positionedMentions++;
        stats.averagePosition += mention.position;
        if (mention.position <= 3) stats.top3Count++;
      }
      
      if (mention.sentiment === 'positive') stats.sentimentPositive++;
      if (mention.sentiment === 'negative') stats.sentimentNegative++;
      if (mention.sentiment === 'neutral') stats.sentimentNeutral++;
    });

    // Calculate averages
    Object.keys(platformStats).forEach(platform => {
      const stats = platformStats[platform];
      if (stats.positionedMentions > 0) {
        stats.averagePosition = stats.averagePosition / stats.positionedMentions;
      }
      stats.visibilityScore = stats.totalMentions > 0 
        ? (stats.positionedMentions / stats.totalMentions) * 100 
        : 0;
    });

    return platformStats;
  }

  async calculateRankingTrends(websiteId, days) {
    const snapshots = await AnalyticsSnapshot.findByWebsiteAndDays(websiteId, days);
    
    // Group by date and calculate daily averages
    const trendData = {};
    snapshots.forEach(snapshot => {
      const date = snapshot.snapshot_date.toISOString().split('T')[0];
      if (!trendData[date]) {
        trendData[date] = {
          date,
          platforms: {},
          overallAverage: 0
        };
      }
      
      trendData[date].platforms[snapshot.platform] = snapshot.average_position;
    });

    // Calculate overall daily averages
    Object.keys(trendData).forEach(date => {
      const dayData = trendData[date];
      const positions = Object.values(dayData.platforms).filter(p => p !== null);
      dayData.overallAverage = positions.length > 0 
        ? positions.reduce((sum, p) => sum + p, 0) / positions.length 
        : null;
    });

    return Object.values(trendData).sort((a, b) => new Date(a.date) - new Date(b.date));
  }

  parseTimeframe(timeframe) {
    switch (timeframe) {
      case '7d': return 7;
      case '30d': return 30;
      case '90d': return 90;
      default: return 30;
    }
  }
}

module.exports = AnalyticsService;
```

## Frontend Application Structure
### React Project Structure
<pre>frontend/
├── src/
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Header.tsx
│   │   │   └── Layout.tsx
│   │   ├── dashboard/
│   │   │   ├── DashboardOverview.tsx
│   │   │   ├── StatsGrid.tsx
│   │   │   ├── PerformanceChart.tsx
│   │   │   └── RecentMentions.tsx
│   │   ├── analytics/
│   │   │   ├── AdvancedAnalytics.tsx
│   │   │   ├── RankingsPositions.tsx
│   │   │   └── CompetitiveAnalysis.tsx
│   │   ├── platforms/
│   │   │   ├── PlatformPage.tsx
│   │   │   ├── ChatGPTPage.tsx
│   │   │   ├── PerplexityPage.tsx
│   │   │   └── [Other platform pages]
│   │   ├── management/
│   │   │   ├── WebsiteManagement.tsx
│   │   │   ├── CompetitorManagement.tsx
│   │   │   └── SearchTermsManagement.tsx
│   │   ├── account/
│   │   │   ├── ProfileTeam.tsx
│   │   │   ├── APIKeys.tsx
│   │   │   └── Settings.tsx
│   │   └── common/
│   │       ├── Button.tsx
│   │       ├── Card.tsx
│   │       ├── Table.tsx
│   │       └── Charts/
│   ├── pages/
│   │   ├── Dashboard.tsx
│   │   ├── Analytics.tsx
│   │   ├── Rankings.tsx
│   │   ├── Platform.tsx
│   │   ├── Login.tsx
│   │   └── Setup.tsx
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useApi.ts
│   │   └── useAnalytics.ts
│   ├── stores/
│   │   ├── authStore.ts
│   │   ├── websiteStore.ts
│   │   └── analyticsStore.ts
│   ├── services/
│   │   ├── api.ts
│   │   ├── auth.ts
│   │   └── analytics.ts
│   ├── types/
│   │   ├── auth.types.ts
│   │   ├── website.types.ts
│   │   └── analytics.types.ts
│   └── utils/
│       ├── constants.ts
│       ├── formatters.ts
│       └── helpers.ts
├── public/
└── package.json</pre>

## AI Platform Integrations
All AI platform integrations follow the same interface pattern defined in the Base AI Service. Each platform service implements:
1. Query Method: Send search term to platform and get response
2. Position Extraction: Determine brand ranking in response (1-10 or null)
3. Sentiment Analysis: Analyze tone of brand mentions
4. Citation Extraction: Find URLs referenced in responses
5. Cost Calculation: Track API usage costs

### Supported Platforms
- OpenAI (ChatGPT): GPT-4 model for comprehensive responses
- Anthropic (Claude): Claude-3 for detailed analytical responses
- Google AI (Gemini): Gemini Pro for search-integrated responses
- Perplexity: Real-time search with citations
- Mistral AI: European-focused AI responses
- DeepSeek: Technical and research-focused queries

## User Interface Specifications
### Layout Structure
#### Sidebar Navigation
<pre>┌─────────────────────────────┐
│ 🔍 WordsRank               │
├─────────────────────────────┤
│ ANALYTICS DASHBOARD         │
│ 📊 Dashboard - Overview     │ ← Active state
│ 📋 Advanced Analytics       │
│ 🎯 Rankings & Positions     │
├─────────────────────────────┤
│ AI PLATFORMS                │
│ 💬 ChatGPT        4.8K 🟢  │
│ 🔍 Perplexity     3.2K 🟢  │
│ ◆ Gemini          2.8K 🟠  │
│ 🎭 Claude         1.5K 🟢  │
│ ⚡ Mistral AI      634 🟠  │
│ 👁 DeepSeek        487 🔵  │
├─────────────────────────────┤
│ SETUP & CONFIGURATION       │
│ 🌐 My Websites             │
│ 🏢 Competitor Websites      │
│ 🔍 Search Terms Management  │
├─────────────────────────────┤
│ ACCOUNT & SETTINGS          │
│ 👥 Profile & Team          │
│ 🔑 API Keys                │
│ ⚙ Settings                │
└─────────────────────────────┘</pre>

#### Styling Requirements:
- Clean white background
- Active page: Blue highlight (#3B82F6) with white text
- Section headers: Gray uppercase text (text-gray-500)
- Menu items: Gray text (text-gray-700) with hover states
- Icons: Simple monochrome line icons
- Badges: Colored rounded badges (green, orange, blue)

#### Header Layout
<pre>┌─────────────────────────────────────────────────────────────────┐
│ Dashboard - Overview                     John Doe  [Avatar] ▼  │
│                                         Marketing Manager       │
└─────────────────────────────────────────────────────────────────┘</pre>

#### Header Components:
- Left: Dynamic page title matching sidebar selection
- Right: User profile section with avatar, name, role, dropdown menu
- Dropdown options: Profile, Premium Plan info, Settings, Logout

### Page Specifications
#### 1. Dashboard - Overview
##### Filter Section (Full Width)
```typescript
interface FilterOptions {
  searchTerms: string[];     // Dropdown of monitored terms
  brand: string[];          // User's websites/brands
  platform: Platform[];    // AI platforms
  timeRange: '7d' | '30d' | '90d' | 'custom';
  sentiment: 'all' | 'positive' | 'negative' | 'neutral';
  topicCluster: string[];   // Content categories
  confidenceLevel: 'all' | 'high' | 'medium' | 'low';
  position: 'all' | 'top3' | '4-5' | '6-10' | 'not-mentioned';
}
```

##### Key Performance Stats (Grid Layout)
```typescript
interface DashboardStats {
  totalMonitoredTerms: number;
  brandVisibilityScore: number;  // 0-100%
  averagePosition: number | null; // 1-10
  shareOfVoice: number;          // vs competitors
  sentimentScore: number;        // positive sentiment %
  platformCoverage: number;      // platforms with mentions
}
```

##### Main Content Grid
<pre>┌─────────────────────────────────────────────────────────────────┐
│ Performance Overview Cards (Full Width)                        │
├─────────────────────────────────────┬───────────────────────────┤
│ Visibility Trends Chart (2/3)      │ Recent Activity (1/3)     │
│ - Time series visualization        │ - Latest mentions         │
│ - Multi-platform trends            │ - Sentiment indicators    │
│ - Interactive tooltips             │ - Platform badges         │
├─────────────────┬─────────────────┬─────────────────────────────┤
│ Search Terms    │ Share of Voice  │ Competitive Intelligence    │
│ Performance     │ by Topic        │ - Competitor rankings       │
│ - Top terms     │ - Topic clusters│ - Content gaps             │
│ - Trending      │ - Market share  │ - Threats & opportunities   │
└─────────────────┴─────────────────┴─────────────────────────────┘</pre>

#### 2. Advanced Analytics
##### Executive Summary (Full Width)
- Written analysis with key insights in plain English
- Auto-generated briefings highlighting wins and issues
- Strategic recommendations based on data

##### Intelligence Grid (2 Columns)
<pre>┌─────────────────────────────────┬─────────────────────────────────┐
│ Market Position Analysis        │ Performance Breakdown           │
│ - Share of voice radar chart   │ - Platform performance heatmap │
│ - Competitive positioning      │ - Sentiment trends chart       │
│ - Market trend analysis        │ - Topic performance rankings   │
└─────────────────────────────────┴─────────────────────────────────┘</pre>

##### Content Strategy Intelligence (Full Width)
- Content gap opportunity matrix
- Top 10 opportunities table with ROI predictions
- Actionable recommendations with difficulty scores

#### 3. Rankings & Positions
##### Rankings Table (Full Width)
```typescript
interface RankingData {
  searchTerm: string;
  chatgptPosition: number | null;
  perplexityPosition: number | null;
  geminiPosition: number | null;
  claudePosition: number | null;
  mistralPosition: number | null;
  deepseekPosition: number | null;
  averagePosition: number;
  trend: 'up' | 'down' | 'stable';
  lastUpdated: Date;
}
```

##### Platform Performance Cards (Grid)
- Individual cards for each AI platform
- Platform-specific metrics and trends
- Position distribution charts
- Optimization opportunities

#### 4. Individual Platform Pages
Each platform page follows this structure:
##### Platform Summary Header
```typescript
interface PlatformSummary {
  performance: string;        // "67% visibility, #3.2 average"
  monthlyTrend: string;      // "18% improvement this month"
  businessImpact: string;    // "12,400 monthly impressions"
  keyOpportunity: string;    // Main optimization suggestion
  mentionCount: number;      // Badge number (e.g., 4.8K)
  badgeColor: 'green' | 'orange' | 'blue';
}
```

##### Performance Intelligence (2 Columns)
- Left: What's working well, what's losing ground
- Right: Visual performance charts and competitive analysis

##### Platform-Specific Features (3 Columns)
- Column 1: Platform behavior analysis
- Column 2: Optimization strategy recommendations
- Column 3: Competitive intelligence insights

##### Action Plan (Full Width)
- This week's priorities with clear next steps
- Timeline and resource requirements

#### 5. Setup & Configuration Pages
##### Website Management
```typescript
interface Website {
  id: string;
  name: string;
  url: string;
  industry: string;
  description?: string;
  platformsMonitored: Platform[];
  isActive: boolean;
  createdAt: Date;
}
```

Simple form with essential fields only:
- Website URL (required)
- Website Name
- Industry dropdown
- Description (optional)
- Platform monitoring checkboxes
- Active/Inactive toggle

##### Competitor Management
```typescript
interface Competitor {
  id: string;
  websiteId: string;
  name: string;
  url: string;
  platformsMonitored: Platform[];
  isActive: boolean;
  createdAt: Date;
}
```

##### Search Terms Management
```typescript
interface SearchTerm {
  id: string;
  websiteId: string;
  term: string;
  platformsMonitored: Platform[];
  isActive: boolean;
  createdAt: Date;
}
```

Features:
- Website selection dropdown
- Simple table with term, platforms, status
- Add new term form
- Bulk CSV import capability

#### 6. Account Pages
##### Profile & Team
```typescript
interface UserProfile {
  fullName: string;
  email: string;
  company: string;
  role: string;
}

interface TeamMember {
  id: string;
  name: string;
  email: string;
  role: 'owner' | 'admin' | 'member';
  joinedAt: Date;
}
```

##### Two-column layout:
- Left: Personal information form
- Right: Team member list with invite functionality

##### API Keys
```typescript
interface PlatformConfig {
  platform: Platform;
  apiKey: string;        // Masked input
  isConnected: boolean;
  lastTested: Date;
  status: 'connected' | 'invalid' | 'error';
}
```

##### Simple form for each platform:
- Masked API key input
- Test connection button
- Connection status indicator

##### Settings
```typescript
interface UserSettings {
  monitoringFrequency: 'daily' | 'weekly';
  dataRetention: '6months' | '1year' | '2years';
  reportPreferences: {
    scheduled: 'weekly' | 'monthly' | 'off';
    format: 'html' | 'pdf';
    emailEnabled: boolean;
  };
  notifications: {
    positionChanges: boolean;
    newMentions: boolean;
    competitorActivity: boolean;
  };
}
```

### Component Implementation Examples
#### Dashboard Stats Grid Component
```typescript
interface StatsGridProps {
  stats: DashboardStats;
  timeframe: string;
  isLoading: boolean;
}

export const StatsGrid: React.FC<StatsGridProps> = ({ stats, timeframe, isLoading }) => {
  if (isLoading) return <StatsGridSkeleton />;

  return (
    <div className="grid grid-cols-3 lg:grid-cols-6 gap-4 mb-8">
      <StatCard
        title="Monitored Terms"
        value={stats.totalMonitoredTerms}
        icon="📊"
        trend={null}
      />
      <StatCard
        title="Visibility Score"
        value={`${stats.brandVisibilityScore}%`}
        icon="👁"
        trend={{ value: 12, direction: 'up' }}
      />
      <StatCard
        title="Avg Position"
        value={stats.averagePosition ? `#${stats.averagePosition}` : 'N/A'}
        icon="🎯"
        trend={{ value: 0.3, direction: 'down' }}
      />
      <StatCard
        title="Share of Voice"
        value={`${stats.shareOfVoice}%`}
        icon="📢"
        trend={{ value: 5, direction: 'up' }}
      />
      <StatCard
        title="Sentiment"
        value={`${stats.sentimentScore}%`}
        icon="😊"
        trend={{ value: 8, direction: 'up' }}
      />
      <StatCard
        title="Platform Coverage"
        value={stats.platformCoverage}
        icon="🔗"
        trend={null}
      />
    </div>
  );
};
```

#### Platform Performance Chart
```typescript
interface PerformanceChartProps {
  data: PlatformPerformance[];
  timeframe: string;
  selectedPlatforms: Platform[];
}

export const PerformanceChart: React.FC<PerformanceChartProps> = ({
  data,
  timeframe,
  selectedPlatforms
}) => {
  return (
    <Card className="p-6">
      <div className="flex items-center justify-between mb-4">
        <h3 className="text-lg font-semibold">Visibility Trends</h3>
        <div className="flex gap-2">
          {selectedPlatforms.map(platform => (
            <PlatformBadge key={platform} platform={platform} />
          ))}
        </div>
      </div>
      
      <ResponsiveContainer width="100%" height={300}>
        <LineChart data={data}>
          <CartesianGrid strokeDasharray="3 3" />
          <XAxis dataKey="date" />
          <YAxis domain={[0, 10]} />
          <Tooltip content={<CustomTooltip />} />
          <Legend />
          
          {selectedPlatforms.map(platform => (
            <Line
              key={platform}
              type="monotone"
              dataKey={platform}
              stroke={getPlatformColor(platform)}
              strokeWidth={2}
              dot={{ r: 4 }}
            />
          ))}
        </LineChart>
      </ResponsiveContainer>
    </Card>
  );
};
```

#### Rankings Table Component
```typescript
interface RankingsTableProps {
  rankings: RankingData[];
  onSortChange: (column: string, direction: 'asc' | 'desc') => void;
  isLoading: boolean;
}

export const RankingsTable: React.FC<RankingsTableProps> = ({
  rankings,
  onSortChange,
  isLoading
}) => {
  return (
    <Card>
      <Table>
        <TableHeader>
          <TableRow>
            <TableHead>Search Term</TableHead>
            <TableHead>ChatGPT</TableHead>
            <TableHead>Perplexity</TableHead>
            <TableHead>Gemini</TableHead>
            <TableHead>Claude</TableHead>
            <TableHead>Mistral AI</TableHead>
            <TableHead>DeepSeek</TableHead>
            <TableHead>Average</TableHead>
            <TableHead>Trend</TableHead>
            <TableHead>Actions</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          {rankings.map(ranking => (
            <TableRow key={ranking.searchTerm}>
              <TableCell className="font-medium">
                {ranking.searchTerm}
              </TableCell>
              <TableCell>
                <PositionBadge position={ranking.chatgptPosition} />
              </TableCell>
              <TableCell>
                <PositionBadge position={ranking.perplexityPosition} />
              </TableCell>
              <TableCell>
                <PositionBadge position={ranking.geminiPosition} />
              </TableCell>
              <TableCell>
                <PositionBadge position={ranking.claudePosition} />
              </TableCell>
              <TableCell>
                <PositionBadge position={ranking.mistralPosition} />
              </TableCell>
              <TableCell>
                <PositionBadge position={ranking.deepseekPosition} />
              </TableCell>
              <TableCell>
                <PositionBadge position={ranking.averagePosition} />
              </TableCell>
              <TableCell>
                <TrendIndicator trend={ranking.trend} />
              </TableCell>
              <TableCell>
                <Button variant="ghost" size="sm">
                  View Details
                </Button>
              </TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </Card>
  );
};
```

### Responsive Design Requirements
#### Mobile Breakpoints
- Mobile: < 768px
- Tablet: 768px - 1024px
- Desktop: > 1024px

#### Mobile Adaptations
- Collapsible sidebar that slides in from left
- Stacked card layouts instead of grids
- Simplified navigation with bottom tab bar
- Touch-friendly buttons and controls
- Reduced chart complexity for small screens

#### Tablet Adaptations
- Sidebar remains visible but narrower
- 2-column layouts instead of 3-column
- Optimized chart sizes
- Touch-friendly hover states

### Accessibility Requirements
#### WCAG 2.1 AA Compliance
- Color Contrast: Minimum 4.5:1 ratio for normal text
- Keyboard Navigation: All interactive elements accessible via keyboard
- Screen Reader Support: Proper ARIA labels and semantic HTML
- Focus Management: Clear focus indicators and logical tab order

#### Implementation Guidelines
```typescript
// Example accessible button component
export const Button: React.FC<ButtonProps> = ({
  children,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  ariaLabel,
  ...props
}) => {
  return (
    <button
      className={cn(buttonVariants({ variant, size }))}
      disabled={disabled}
      aria-label={ariaLabel}
      {...props}
    >
      {children}
    </button>
  );
};

// Example accessible chart component
export const AccessibleChart: React.FC<ChartProps> = ({ data, title }) => {
  return (
    <div role="img" aria-label={`Chart showing ${title}`}>
      <ResponsiveContainer width="100%" height={300}>
        <LineChart data={data}>
          {/* Chart implementation */}
        </LineChart>
      </ResponsiveContainer>
      
      {/* Data table alternative for screen readers */}
      <div className="sr-only">
        <table>
          <caption>{title} - Data Table</caption>
          {/* Table representation of chart data */}
        </table>
      </div>
    </div>
  );
};
```

## Authentication & Security
### JWT Authentication Implementation
```javascript
// middleware/auth.middleware.js
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

class AuthMiddleware {
  static async authenticate(req, res, next) {
    try {
      const token = req.header('Authorization')?.replace('Bearer ', '');
      
      if (!token) {
        return res.status(401).json({ error: 'Access denied. No token provided.' });
      }

      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      req.user = decoded;
      
      next();
    } catch (error) {
      res.status(400).json({ error: 'Invalid token.' });
    }
  }

  static authorize(roles = []) {
    return (req, res, next) => {
      if (!req.user) {
        return res.status(401).json({ error: 'Access denied.' });
      }

      if (roles.length && !roles.includes(req.user.role)) {
        return res.status(403).json({ error: 'Insufficient permissions.' });
      }

      next();
    };
  }

  static async hashPassword(password) {
    const salt = await bcrypt.genSalt(12);
    return bcrypt.hash(password, salt);
  }

  static async comparePassword(password, hash) {
    return bcrypt.compare(password, hash);
  }

  static generateToken(user) {
    return jwt.sign(
      {
        id: user.id,
        email: user.email,
        role: user.role,
        plan: user.plan
      },
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    );
  }
}

module.exports = AuthMiddleware;
```

### API Key Encryption
```javascript
// utils/encryption.js
const crypto = require('crypto');

class EncryptionService {
  constructor() {
    this.algorithm = 'aes-256-gcm';
    this.secretKey = process.env.ENCRYPTION_KEY; // 32 bytes key
  }

  encrypt(text) {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipher(this.algorithm, this.secretKey, iv);
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    return {
      encryptedData: encrypted,
      iv: iv.toString('hex'),
      authTag: authTag.toString('hex')
    };
  }

  decrypt(encryptedData, iv, authTag) {
    const decipher = crypto.createDecipher(this.algorithm, this.secretKey, Buffer.from(iv, 'hex'));
    decipher.setAuthTag(Buffer.from(authTag, 'hex'));
    
    let decrypted = decipher.update(encryptedData, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}

module.exports = EncryptionService;
```

### Rate Limiting
```javascript
// middleware/rateLimit.middleware.js
const rateLimit = require('express-rate-limit');

const createLimiter = (windowMs, max, message) => {
  return rateLimit({
    windowMs,
    max,
    message: { error: message },
    standardHeaders: true,
    legacyHeaders: false,
  });
};

module.exports = {
  // General API rate limiting
  apiLimiter: createLimiter(
    15 * 60 * 1000, // 15 minutes
    100, // 100 requests per window
    'Too many API requests, please try again later.'
  ),
  
  // Authentication rate limiting
  authLimiter: createLimiter(
    15 * 60 * 1000, // 15 minutes
    5, // 5 attempts per window
    'Too many authentication attempts, please try again later.'
  ),
  
  // AI platform query limiting
  queryLimiter: createLimiter(
    60 * 1000, // 1 minute
    10, // 10 queries per minute
    'Too many AI platform queries, please try again later.'
  )
};
```

## Development Setup
### Environment Configuration
```bash
# .env.example
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/wordsrank"

# JWT Configuration
JWT_SECRET="your-super-secret-jwt-key-change-this-in-production"
JWT_EXPIRES_IN="7d"

# Encryption
ENCRYPTION_KEY="your-32-byte-encryption-key-for-api-keys"

# AI Platform API Keys
OPENAI_API_KEY="sk-your-openai-key"
ANTHROPIC_API_KEY="sk-ant-your-anthropic-key"
GOOGLE_AI_API_KEY="your-google-ai-key"
PERPLEXITY_API_KEY="pplx-your-perplexity-key"
MISTRAL_API_KEY="your-mistral-key"
DEEPSEEK_API_KEY="your-deepseek-key"

# Application Settings
NODE_ENV="development"
PORT=3001
FRONTEND_URL="http://localhost:3000"

# Monitoring Settings
MONITORING_ENABLED=true
MONITORING_INTERVAL_HOURS=24
DEFAULT_QUERY_LIMIT=1000

# Email Configuration (for reports and notifications)
SMTP_HOST="smtp.example.com"
SMTP_PORT=587
SMTP_USER="your-email@example.com"
SMTP_PASS="your-email-password"

# Logging
LOG_LEVEL="info"
LOG_FILE="logs/app.log"
```

### Docker Development Setup
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  postgresql:
    image: postgres:15
    environment:
      POSTGRES_DB: wordsrank
      POSTGRES_USER: wordsrank
      POSTGRES_PASSWORD: wordsrank_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backend/migrations:/docker-entrypoint-initdb.d

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://wordsrank:wordsrank_dev@postgresql:5432/wordsrank
    volumes:
      - ./backend:/app
      - /app/node_modules
    depends_on:
      - postgresql
    command: npm run dev

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:3001/api
    volumes:
      - ./frontend:/app
      - /app/node_modules
    depends_on:
      - backend
    command: npm run dev

volumes:
  postgres_data:
```

### Package.json Files
#### Root Package.json
```json
{
  "name": "wordsrank",
  "version": "1.0.0",
  "description": "AI Platform Monitoring Tool",
  "scripts": {
    "dev": "concurrently \"npm run dev:backend\" \"npm run dev:frontend\"",
    "dev:backend": "cd backend && npm run dev",
    "dev:frontend": "cd frontend && npm run dev",
    "build": "npm run build:frontend && npm run build:backend",
    "build:backend": "cd backend && npm run build",
    "build:frontend": "cd frontend && npm run build",
    "test": "npm run test:backend && npm run test:frontend",
    "test:backend": "cd backend && npm test",
    "test:frontend": "cd frontend && npm test",
    "migrate": "cd backend && npm run migrate",
    "seed": "cd backend && npm run seed",
    "start:prod": "cd backend && npm start"
  },
  "devDependencies": {
    "concurrently": "^8.2.0"
  }
}
```

#### Backend Package.json
```json
{
  "name": "wordsrank-backend",
  "version": "1.0.0",
  "scripts": {
    "dev": "nodemon src/app.js",
    "start": "node src/app.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "migrate": "node scripts/migrate.js",
    "seed": "node scripts/seed.js",
    "lint": "eslint src/",
    "format": "prettier --write src/"
  },
  "dependencies": {
    "express": "^4.18.2",
    "postgres": "^3.3.5",
    "jsonwebtoken": "^9.0.2",
    "bcryptjs": "^2.4.3",
    "joi": "^17.9.2",
    "winston": "^3.10.0",
    "axios": "^1.5.0",
    "cors": "^2.8.5",
    "helmet": "^7.0.0",
    "express-rate-limit": "^6.10.0",
    "node-cron": "^3.0.2",
    "openai": "^4.0.0",
    "@anthropic-ai/sdk": "^0.6.0",
    "@google-ai/generativelanguage": "^2.4.0",
    "@mistralai/mistralai": "^0.1.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "jest": "^29.6.2",
    "supertest": "^6.3.3",
    "eslint": "^8.45.0",
    "prettier": "^3.0.0"
  }
}
```

#### Frontend Package.json
```json
{
  "name": "wordsrank-frontend",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "format": "prettier --write src/"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.15.0",
    "recharts": "^2.7.2",
    "axios": "^1.5.0",
    "zustand": "^4.4.1",
    "react-hook-form": "^7.45.4",
    "zod": "^3.22.2",
    "@hookform/resolvers": "^3.3.1",
    "date-fns": "^2.30.0",
    "clsx": "^2.0.0",
    "tailwind-merge": "^1.14.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.15",
    "@types/react-dom": "^18.2.7",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "@vitejs/plugin-react": "^4.0.3",
    "autoprefixer": "^10.4.14",
    "eslint": "^8.45.0",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-react-refresh": "^0.4.3",
    "postcss": "^8.4.27",
    "prettier": "^3.0.0",
    "tailwindcss": "^3.3.3",
    "typescript": "^5.0.2",
    "vite": "^4.4.5",
    "vitest": "^0.34.1",
    "@vitest/ui": "^0.34.1",
    "@testing-library/react": "^13.4.0",
    "@testing-library/jest-dom": "^5.17.0"
  }
}
```

## Testing Strategy
### Backend Testing
```javascript
// tests/auth.test.js
const request = require('supertest');
const app = require('../src/app');
const { User } = require('../src/models');

describe('Authentication', () => {
  beforeEach(async () => {
    await User.deleteAll(); // Clean database
  });

  describe('POST /api/auth/register', () => {
    it('should create a new user', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'password123',
        fullName: 'Test User',
        company: 'Test Company'
      };

      const response = await request(app)
        .post('/api/auth/register')
        .send(userData)
        .expect(201);

      expect(response.body).toHaveProperty('token');
      expect(response.body.user.email).toBe(userData.email);
      expect(response.body.user).not.toHaveProperty('password');
    });

    it('should return 400 for invalid email', async () => {
      const userData = {
        email: 'invalid-email',
        password: 'password123',
        fullName: 'Test User'
      };

      await request(app)
        .post('/api/auth/register')
        .send(userData)
        .expect(400);
    });
  });

  describe('POST /api/auth/login', () => {
    beforeEach(async () => {
      await User.create({
        email: 'test@example.com',
        password: await AuthMiddleware.hashPassword('password123'),
        fullName: 'Test User'
      });
    });

    it('should login with valid credentials', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'password123'
        })
        .expect(200);

      expect(response.body).toHaveProperty('token');
    });

    it('should return 401 for invalid credentials', async () => {
      await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'wrongpassword'
        })
        .expect(401);
    });
  });
});
```

```javascript
// tests/ai-platforms.test.js
const { OpenAIService } = require('../src/services/ai-platforms');

describe('AI Platform Services', () => {
  let openaiService;

  beforeEach(() => {
    openaiService = new OpenAIService(process.env.OPENAI_API_KEY_TEST);
  });

  describe('OpenAI Service', () => {
    it('should query platform and return structured response', async () => {
      const searchTerm = 'best electric vehicles 2024';
      const brandTerms = ['Tesla', 'Model 3'];

      const result = await openaiService.queryPlatform(searchTerm, { brandTerms });

      expect(result).toHaveProperty('platform', 'chatgpt');
      expect(result).toHaveProperty('query', searchTerm);
      expect(result).toHaveProperty('response');
      expect(result).toHaveProperty('position');
      expect(result).toHaveProperty('sentiment');
      expect(result).toHaveProperty('confidence');
      expect(typeof result.confidence).toBe('number');
      expect(result.confidence).toBeGreaterThanOrEqual(0);
      expect(result.confidence).toBeLessThanOrEqual(1);
    });

    it('should extract position correctly', () => {
      const content = 'Tesla Model 3 is the best electric vehicle. BMW i3 is also good.';
      const brandTerms = ['Tesla'];

      const position = openaiService.extractPosition(content, brandTerms);
      expect(position).toBe(1);
    });

    it('should return null position when brand not mentioned', () => {
      const content = 'BMW and Ford make great electric vehicles.';
      const brandTerms = ['Tesla'];

      const position = openaiService.extractPosition(content, brandTerms);
      expect(position).toBeNull();
    });
  });
});
```

### Frontend Testing
```typescript
// tests/Dashboard.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { vi } from 'vitest';
import { Dashboard } from '../src/pages/Dashboard';
import { AuthProvider } from '../src/contexts/AuthContext';

// Mock API calls
vi.mock('../src/services/api', () => ({
  getDashboardAnalytics: vi.fn().mockResolvedValue({
    overview: {
      totalMentions: 150,
      visibilityScore: 73,
      averagePosition: 3.2,
      top3Rankings: 12
    },
    platformPerformance: {
      chatgpt: { totalMentions: 50, averagePosition: 2.8 },
      perplexity: { totalMentions: 40, averagePosition: 3.1 }
    }
  })
}));

const renderWithAuth = (component: React.ReactElement) => {
  return render(
    <AuthProvider>
      {component}
    </AuthProvider>
  );
};

describe('Dashboard', () => {
  it('renders dashboard stats correctly', async () => {
    renderWithAuth(<Dashboard />);

    await waitFor(() => {
      expect(screen.getByText('150')).toBeInTheDocument();
      expect(screen.getByText('73%')).toBeInTheDocument();
      expect(screen.getByText('#3.2')).toBeInTheDocument();
    });
  });

  it('shows loading state initially', () => {
    renderWithAuth(<Dashboard />);
    
    expect(screen.getByTestId('stats-grid-skeleton')).toBeInTheDocument();
  });
});
```

```typescript
// tests/components/StatsGrid.test.tsx
import { render, screen } from '@testing-library/react';
import { StatsGrid } from '../src/components/dashboard/StatsGrid';

const mockStats = {
  totalMonitoredTerms: 25,
  brandVisibilityScore: 68,
  averagePosition: 3.4,
  shareOfVoice: 15,
  sentimentScore: 72,
  platformCoverage: 5
};

describe('StatsGrid', () => {
  it('displays all stats correctly', () => {
    render(<StatsGrid stats={mockStats} timeframe="30d" isLoading={false} />);

    expect(screen.getByText('25')).toBeInTheDocument();
    expect(screen.getByText('68%')).toBeInTheDocument();
    expect(screen.getByText('#3.4')).toBeInTheDocument();
    expect(screen.getByText('15%')).toBeInTheDocument();
    expect(screen.getByText('72%')).toBeInTheDocument();
    expect(screen.getByText('5')).toBeInTheDocument();
  });

  it('shows skeleton when loading', () => {
    render(<StatsGrid stats={mockStats} timeframe="30d" isLoading={true} />);
    
    expect(screen.getByTestId('stats-grid-skeleton')).toBeInTheDocument();
  });
});
```

### Test Configuration Files
```javascript
// backend/jest.config.js
module.exports = {
  testEnvironment: 'node',
  setupFilesAfterEnv: ['<rootDir>/tests/setup.js'],
  testMatch: ['**/__tests__/**/*.js', '**/?(*.)+(spec|test).js'],
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/app.js',
    '!**/node_modules/**'
  ],
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70
    }
  }
};
```

```typescript
// frontend/vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    globals: true,
    coverage: {
      reporter: ['text', 'html'],
      threshold: {
        global: {
          branches: 70,
          functions: 70,
          lines: 70,
          statements: 70
        }
      }
    }
  }
});
```

## Deployment & DevOps
### Production Docker Setup
```yaml
# docker-compose.yml
version: '3.8'

services:
  postgresql:
    image: postgres:15
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - wordsrank-network

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@postgresql:5432/${DB_NAME}
      - JWT_SECRET=${JWT_SECRET}
      - ENCRYPTION_KEY=${ENCRYPTION_KEY}
    depends_on:
      - postgresql
    restart: unless-stopped
    networks:
      - wordsrank-network

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      args:
        - REACT_APP_API_URL=${API_URL}
    restart: unless-stopped
    networks:
      - wordsrank-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - backend
      - frontend
    restart: unless-stopped
    networks:
      - wordsrank-network

volumes:
  postgres_data:

networks:
  wordsrank-network:
    driver: bridge
```

### Nginx Configuration
```nginx
# nginx/nginx.conf
events {
    worker_connections 1024;
}

http {
    upstream backend {
        server backend:3001;
    }

    upstream frontend {
        server frontend:3000;
    }

    # Redirect HTTP to HTTPS
    server {
        listen 80;
        server_name your-domain.com;
        return 301 https://$server_name$request_uri;
    }

    # HTTPS Configuration
    server {
        listen 443 ssl http2;
        server_name your-domain.com;

        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;

        # Frontend
        location / {
            proxy_pass http://frontend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # API
        location /api/ {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Rate limiting
            limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
            limit_req zone=api burst=20 nodelay;
        }

        # Security headers
        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
    }
}
```

### GitHub Actions CI/CD
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: wordsrank_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: |
          cd backend && npm ci
          cd ../frontend && npm ci
      
      - name: Run backend tests
        run: cd backend && npm test
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/wordsrank_test
          JWT_SECRET: test-secret
      
      - name: Run frontend tests
        run: cd frontend && npm test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to server
        uses: appleboy/ssh-action@v0.1.5
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /opt/wordsrank
            git pull origin main
            docker-compose down
            docker-compose build
            docker-compose up -d
            docker system prune -f
```

### Database Migration Script
```javascript
// scripts/migrate.js
const postgres = require('postgres');
const fs = require('fs');
const path = require('path');

const sql = postgres(process.env.DATABASE_URL);

async function runMigrations() {
  try {
    console.log('Starting database migrations...');
    
    // Create migrations table if it doesn't exist
    await sql`
      CREATE TABLE IF NOT EXISTS migrations (
        id SERIAL PRIMARY KEY,
        filename VARCHAR(255) NOT NULL,
        executed_at TIMESTAMP DEFAULT NOW()
      )
    `;
    
    // Get executed migrations
    const executedMigrations = await sql`
      SELECT filename FROM migrations ORDER BY id
    `;
    const executedFiles = executedMigrations.map(m => m.filename);
    
    // Read migration files
    const migrationsDir = path.join(__dirname, '../migrations');
    const migrationFiles = fs.readdirSync(migrationsDir)
      .filter(file => file.endsWith('.sql'))
      .sort();
    
    // Execute pending migrations
    for (const file of migrationFiles) {
      if (!executedFiles.includes(file)) {
        console.log(`Executing migration: ${file}`);
        
        const migrationSQL = fs.readFileSync(
          path.join(migrationsDir, file), 
          'utf8'
        );
        
        await sql.unsafe(migrationSQL);
        
        await sql`
          INSERT INTO migrations (filename) VALUES (${file})
        `;
        
        console.log(`✓ Migration ${file} completed`);
      }
    }
    
    console.log('All migrations completed successfully');
  } catch (error) {
    console.error('Migration failed:', error);
    process.exit(1);
  } finally {
    await sql.end();
  }
}

runMigrations();
```

### Monitoring & Logging
```javascript
// utils/logger.js
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'wordsrank-api' },
  transports: [
    new winston.transports.File({ 
      filename: 'logs/error.log', 
      level: 'error' 
    }),
    new winston.transports.File({ 
      filename: 'logs/combined.log' 
    }),
  ],
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

module.exports = logger;
```

### Health Check Endpoints
```javascript
// routes/health.routes.js
const express = require('express');
const router = express.Router();
const postgres = require('postgres');

const sql = postgres(process.env.DATABASE_URL);

// Basic health check
router.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});

// Detailed health check
router.get('/health/detailed', async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    checks: {}
  };

  // Database check
  try {
    await sql`SELECT 1`;
    health.checks.database = 'healthy';
  } catch (error) {
    health.checks.database = 'unhealthy';
    health.status = 'unhealthy';
  }

  // Memory check
  const memUsage = process.memoryUsage();
  health.checks.memory = {
    usage: Math.round(memUsage.heapUsed / 1024 / 1024),
    total: Math.round(memUsage.heapTotal / 1024 / 1024)
  };

  res.status(health.status === 'healthy' ? 200 : 503).json(health);
});

module.exports = router;
```

## Technical Debt Prevention
### Code Quality Standards
- ESLint + Prettier: Enforced code formatting and linting
- TypeScript: Type safety for frontend development
- Test Coverage: Minimum 70% test coverage requirement
- Code Reviews: All PRs require review before merge
- Documentation: Inline comments and README updates required

### Performance Considerations
- Database Indexing: Proper indexes on frequently queried columns
- API Rate Limiting: Prevent abuse and ensure fair usage
- Caching Strategy: Application-level caching for frequently accessed data
- Lazy Loading: Frontend components loaded on demand
- Image Optimization: Optimized images and assets

### Security Best Practices
- Input Validation: All inputs validated with Joi/Zod
- SQL Injection Prevention: Parameterized queries only
- XSS Protection: Content Security Policy headers
- Authentication: JWT with secure secret rotation
- API Key Encryption: Encrypted storage of sensitive keys

This comprehensive specification provides a complete roadmap for building WordsRank as a focused, practical AI platform monitoring tool. The architecture is designed to be implementable by a development team while maintaining room for future growth and feature expansion.
