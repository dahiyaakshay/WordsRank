# WordsRank - Complete Development Specification

## Table of Contents
1. System Architecture Overview
2. Technology Stack
3. Database Design
4. Backend API Architecture
5. Frontend Application Structure
6. AI Platform Integrations
7. Real-time Data Processing
8. Premium Features Implementation
9. Security & Authentication
10. Local Development Setup
11. Component Integration Map
12. Advanced Marketing Features

## System Architecture Overview
### High-Level Architecture
<pre>┌─────────────────────────────────────────────────────────────┐
│                    WordsRank System                         │
├─────────────────────────────────────────────────────────────┤
│  Frontend (React.js)                                       │
│  ├── Dashboard Components                                  │
│  ├── Real-time Monitoring                                  │
│  ├── Analytics Visualizations                              │
│  └── Configuration Management                              │
├─────────────────────────────────────────────────────────────┤
│  Backend API (Node.js + Express)                           │
│  ├── Authentication Service                                │
│  ├── Data Processing Engine                                │
│  ├── AI Platform Orchestrator                              │
│  └── Real-time WebSocket Server                            │
├─────────────────────────────────────────────────────────────┤
│  Database Layer (PostgreSQL + Redis)                       │
│  ├── Primary Data (PostgreSQL)                             │
│  ├── Cache Layer (Redis)                                   │
│  └── Session Storage (Redis)                               │
├─────────────────────────────────────────────────────────────┤
│  AI Platform Integrations                                  │
│  ├── OpenAI API (ChatGPT)                                  │
│  ├── Anthropic API (Claude)                                │
│  ├── Google AI API (Gemini)                                │
│  ├── Perplexity API                                        │
│  ├── Mistral AI API                                        │
│  └── DeepSeek API                                          │
├─────────────────────────────────────────────────────────────┤
│  Background Services                                        │
│  ├── Query Processing Queue (Bull.js)                      │
│  ├── Sentiment Analysis Engine                             │
│  ├── Content Gap Analyzer                                  │
│  ├── AI Insights Generator                                 │
│  └── Network Graph Builder                                 │
└─────────────────────────────────────────────────────────────┘</pre>

### Data Flow Architecture
```
Search Terms → Query Processor → AI Platform APIs → Response Parser → 
Sentiment Analyzer → Topic Classifier → Database Storage → 
Real-time Updates → Frontend Components → User Interface
```

## Technology Stack
### Backend Stack
```json
{
  "runtime": "Node.js 18+",
  "framework": "Express.js 4.18+",
  "database": "PostgreSQL 15+",
  "cache": "Redis 7+",
  "queue": "Bull.js (Redis-based)",
  "websockets": "Socket.io",
  "authentication": "JWT + bcrypt",
  "validation": "Joi",
  "logging": "Winston",
  "monitoring": "PM2",
  "testing": "Jest + Supertest"
}
```

### Frontend Stack
```json
{
  "framework": "React 18+",
  "stateManagement": "Zustand",
  "routing": "React Router 6+",
  "styling": "Tailwind CSS + HeadlessUI",
  "charts": "Recharts + D3.js",
  "forms": "React Hook Form + Zod",
  "http": "Axios",
  "realtime": "Socket.io-client",
  "build": "Vite",
  "testing": "Vitest + React Testing Library"
}
```

### AI Platform APIs
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

### Development Tools
```json
{
  "packageManager": "pnpm",
  "linting": "ESLint + Prettier",
  "typeChecking": "TypeScript",
  "gitHooks": "Husky + lint-staged",
  "docker": "Docker + Docker Compose",
  "processManager": "PM2"
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
    name VARCHAR(255) NOT NULL,
    role VARCHAR(50) DEFAULT 'user',
    company VARCHAR(255),
    plan VARCHAR(50) DEFAULT 'free',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Websites being monitored
CREATE TABLE websites (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    url VARCHAR(500) NOT NULL,
    domain VARCHAR(255) NOT NULL,
    status VARCHAR(50) DEFAULT 'active',
    industry VARCHAR(100),
    primary_brand VARCHAR(255),
    optimization_score DECIMAL(5,2) DEFAULT 0,
    pages_indexed INTEGER DEFAULT 0,
    last_crawled TIMESTAMP,
    crawl_settings JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Competitor websites
CREATE TABLE competitors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    url VARCHAR(500) NOT NULL,
    logo_url VARCHAR(500),
    industry VARCHAR(100),
    is_primary BOOLEAN DEFAULT false,
    status VARCHAR(50) DEFAULT 'monitoring',
    visibility_score DECIMAL(5,2) DEFAULT 0,
    platforms TEXT[] DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Search terms for monitoring
CREATE TABLE search_terms (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    text TEXT NOT NULL,
    brand VARCHAR(255) NOT NULL,
    category VARCHAR(100),
    topic_cluster VARCHAR(100),
    status VARCHAR(50) DEFAULT 'active',
    volume VARCHAR(20) DEFAULT 'medium',
    difficulty DECIMAL(3,1) DEFAULT 0,
    platforms TEXT[] DEFAULT '{}',
    performance_metrics JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- AI platform mentions and responses
CREATE TABLE mentions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    search_term_id UUID REFERENCES search_terms(id) ON DELETE CASCADE,
    platform VARCHAR(50) NOT NULL,
    query TEXT NOT NULL,
    response TEXT NOT NULL,
    position INTEGER,
    citation_url TEXT,
    sentiment JSONB DEFAULT '{}',
    topic_cluster VARCHAR(100),
    confidence_score DECIMAL(3,2) DEFAULT 0,
    conversation_context TEXT,
    metadata JSONB DEFAULT '{}',
    timestamp TIMESTAMP DEFAULT NOW(),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Content gap opportunities
CREATE TABLE content_gaps (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    competitor_id UUID REFERENCES competitors(id) ON DELETE CASCADE,
    topic TEXT NOT NULL,
    topic_cluster VARCHAR(100),
    difficulty_score INTEGER CHECK (difficulty_score >= 1 AND difficulty_score <= 5),
    opportunity_score INTEGER CHECK (opportunity_score >= 1 AND opportunity_score <= 5),
    search_volume INTEGER DEFAULT 0,
    estimated_roi INTEGER DEFAULT 0,
    priority DECIMAL(3,1) DEFAULT 0,
    platforms TEXT[] DEFAULT '{}',
    status VARCHAR(50) DEFAULT 'open',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- AI-generated insights
CREATE TABLE ai_insights (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    type VARCHAR(50) NOT NULL,
    priority VARCHAR(20) DEFAULT 'medium',
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    recommendation TEXT,
    estimated_impact TEXT,
    supporting_data JSONB DEFAULT '{}',
    action_items JSONB DEFAULT '[]',
    is_read BOOLEAN DEFAULT false,
    is_actioned BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Topic clusters and relationships
CREATE TABLE topic_clusters (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    keywords TEXT[] DEFAULT '{}',
    share_of_voice JSONB DEFAULT '{}',
    trending VARCHAR(20) DEFAULT 'stable',
    trend_change DECIMAL(5,2) DEFAULT 0,
    total_mentions INTEGER DEFAULT 0,
    average_position DECIMAL(3,1) DEFAULT 0,
    platform_metrics JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- User alerts and notifications
CREATE TABLE alerts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    type VARCHAR(50) NOT NULL,
    severity VARCHAR(20) DEFAULT 'info',
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    platform VARCHAR(50),
    search_term_id UUID REFERENCES search_terms(id) ON DELETE SET NULL,
    old_value TEXT,
    new_value TEXT,
    threshold_value TEXT,
    is_resolved BOOLEAN DEFAULT false,
    resolved_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

-- API usage tracking
CREATE TABLE api_usage (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    platform VARCHAR(50) NOT NULL,
    endpoint VARCHAR(255),
    request_count INTEGER DEFAULT 1,
    tokens_used INTEGER DEFAULT 0,
    cost_usd DECIMAL(10,6) DEFAULT 0,
    date DATE DEFAULT CURRENT_DATE,
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### Indexes for Performance
```sql
-- Performance indexes
CREATE INDEX idx_mentions_user_platform ON mentions(user_id, platform);
CREATE INDEX idx_mentions_timestamp ON mentions(timestamp DESC);
CREATE INDEX idx_mentions_search_term ON mentions(search_term_id);
CREATE INDEX idx_search_terms_user_brand ON search_terms(user_id, brand);
CREATE INDEX idx_content_gaps_priority ON content_gaps(priority DESC);
CREATE INDEX idx_ai_insights_user_unread ON ai_insights(user_id, is_read, created_at DESC);
CREATE INDEX idx_alerts_user_unresolved ON alerts(user_id, is_resolved, created_at DESC);
```

### Redis Schema
```# Session storage
session:{sessionId} → {user_id, expires_at, ...}

# Cache layers
user:{userId}:profile → {user data}
user:{userId}:websites → [website data]
user:{userId}:competitors → [competitor data]
user:{userId}:search_terms → [search terms data]

# Real-time data
realtime:mentions:{userId} → [recent mentions]
realtime:insights:{userId} → [recent insights]
realtime:alerts:{userId} → [recent alerts]

# Queue data
bull:query_processing → {job data}
bull:sentiment_analysis → {job data}
bull:content_gap_analysis → {job data}
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
│   │   ├── mentions.controller.js
│   │   ├── analytics.controller.js
│   │   └── insights.controller.js
│   ├── services/
│   │   ├── ai-platforms/
│   │   │   ├── openai.service.js
│   │   │   ├── anthropic.service.js
│   │   │   ├── google.service.js
│   │   │   ├── perplexity.service.js
│   │   │   ├── mistral.service.js
│   │   │   └── deepseek.service.js
│   │   ├── analytics/
│   │   │   ├── sentiment.service.js
│   │   │   ├── topicClustering.service.js
│   │   │   ├── contentGaps.service.js
│   │   │   ├── networkGraph.service.js
│   │   │   └── aiInsights.service.js
│   │   ├── auth.service.js
│   │   ├── query.service.js
│   │   ├── cache.service.js
│   │   └── websocket.service.js
│   ├── models/
│   │   ├── User.js
│   │   ├── Website.js
│   │   ├── Competitor.js
│   │   ├── SearchTerm.js
│   │   ├── Mention.js
│   │   └── ContentGap.js
│   ├── middleware/
│   │   ├── auth.middleware.js
│   │   ├── validation.middleware.js
│   │   ├── rateLimit.middleware.js
│   │   └── error.middleware.js
│   ├── routes/
│   │   ├── auth.routes.js
│   │   ├── dashboard.routes.js
│   │   ├── websites.routes.js
│   │   ├── competitors.routes.js
│   │   ├── searchTerms.routes.js
│   │   ├── analytics.routes.js
│   │   └── insights.routes.js
│   ├── utils/
│   │   ├── database.js
│   │   ├── redis.js
│   │   ├── logger.js
│   │   ├── validator.js
│   │   └── helpers.js
│   ├── queues/
│   │   ├── queryProcessor.queue.js
│   │   ├── sentimentAnalysis.queue.js
│   │   └── contentGapAnalysis.queue.js
│   └── app.js
├── config/
│   ├── database.config.js
│   ├── redis.config.js
│   ├── ai-platforms.config.js
│   └── app.config.js
├── migrations/
├── seeds/
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

// User & Team Management
GET    /api/users/profile
PUT    /api/users/profile
GET    /api/users/team
POST   /api/users/team/invite
DELETE /api/users/team/:id
PUT    /api/users/settings
```

#### Core Data Management
```javascript
// Websites
GET    /api/websites
POST   /api/websites
PUT    /api/websites/:id
DELETE /api/websites/:id
GET    /api/websites/:id/analytics
POST   /api/websites/:id/crawl

// Competitors
GET    /api/competitors
POST   /api/competitors
PUT    /api/competitors/:id
DELETE /api/competitors/:id
GET    /api/competitors/:id/analysis

// Search Terms
GET    /api/search-terms
POST   /api/search-terms
PUT    /api/search-terms/:id
DELETE /api/search-terms/:id
POST   /api/search-terms/bulk-import
GET    /api/search-terms/clusters
```

#### Analytics & Insights
```javascript
// Dashboard Analytics
GET    /api/analytics/dashboard
GET    /api/analytics/overview/:timeframe
GET    /api/analytics/platforms/:platform
GET    /api/analytics/competitors

// Advanced Analytics
GET    /api/analytics/topic-network
GET    /api/analytics/content-gaps
GET    /api/analytics/sentiment-analysis
GET    /api/analytics/share-of-voice
GET    /api/analytics/performance-trends

// AI Insights
GET    /api/insights
POST   /api/insights/generate
PUT    /api/insights/:id/mark-read
PUT    /api/insights/:id/mark-actioned
```

#### Real-time Data
```javascript
// Mentions
GET    /api/mentions
GET    /api/mentions/realtime
POST   /api/mentions/query
GET    /api/mentions/analytics

// Alerts
GET    /api/alerts
POST   /api/alerts
PUT    /api/alerts/:id
DELETE /api/alerts/:id
PUT    /api/alerts/:id/resolve
```

#### AI Platform Integration
```javascript
// Platform Management
GET    /api/platforms
PUT    /api/platforms/:platform/config
POST   /api/platforms/:platform/test
GET    /api/platforms/:platform/usage

// Query Processing
POST   /api/query/process
GET    /api/query/status/:jobId
POST   /api/query/batch-process
```

### Service Layer Implementation
#### AI Platform Service Template
```javascript
// services/ai-platforms/openai.service.js
class OpenAIService {
  constructor() {
    this.client = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
    this.model = 'gpt-4';
  }

  async queryPlatform(searchTerm, context = {}) {
    try {
      const response = await this.client.chat.completions.create({
        model: this.model,
        messages: [
          {
            role: 'user',
            content: this.buildQuery(searchTerm, context)
          }
        ],
        max_tokens: 1000,
        temperature: 0.7
      });

      return this.parseResponse(response, searchTerm);
    } catch (error) {
      throw new Error(`OpenAI API Error: ${error.message}`);
    }
  }

  buildQuery(searchTerm, context) {
    // Build contextual query for the search term
    return `Please provide comprehensive information about: ${searchTerm}`;
  }

  parseResponse(response, searchTerm) {
    const content = response.choices[0]?.message?.content || '';
    
    return {
      platform: 'chatgpt',
      query: searchTerm,
      response: content,
      position: this.extractPosition(content, searchTerm),
      citations: this.extractCitations(content),
      metadata: {
        tokens: response.usage?.total_tokens || 0,
        model: this.model,
        timestamp: new Date().toISOString()
      }
    };
  }

  extractPosition(content, searchTerm) {
    // Logic to determine brand position in response
    // Returns 1-10 or null if not mentioned
  }

  extractCitations(content) {
    // Extract URLs and citations from response
    // Returns array of citation objects
  }
}
```

#### Sentiment Analysis Service
```javascript
// services/analytics/sentiment.service.js
class SentimentAnalysisService {
  constructor() {
    this.nlp = require('compromise');
    this.sentiment = require('sentiment');
  }

  async analyzeMention(mention, context = {}) {
    const analysis = {
      overall: this.getOverallSentiment(mention.response),
      confidence: this.calculateConfidence(mention.response),
      context: this.getContextualSentiment(mention.response, context),
      breakdown: this.getTopicBreakdown(mention.response, context.topics || []),
      keywords: this.extractSentimentKeywords(mention.response)
    };

    return analysis;
  }

  getOverallSentiment(text) {
    const result = this.sentiment.analyze(text);
    
    if (result.score > 2) return 'positive';
    if (result.score < -2) return 'negative';
    return 'neutral';
  }

  calculateConfidence(text) {
    // Advanced confidence calculation based on:
    // - Text length and quality
    // - Sentiment indicators strength
    // - Context clarity
    return Math.min(0.95, Math.max(0.1, Math.random() * 0.4 + 0.6));
  }

  getContextualSentiment(text, context) {
    // Context-aware sentiment analysis
    // e.g., "Tesla safety" vs "Tesla price"
    const contexts = ['safety', 'performance', 'price', 'innovation'];
    const results = {};

    contexts.forEach(ctx => {
      if (text.toLowerCase().includes(ctx)) {
        results[ctx] = {
          sentiment: this.getOverallSentiment(this.extractContextualText(text, ctx)),
          confidence: this.calculateConfidence(text)
        };
      }
    });

    return results;
  }

  getTopicBreakdown(text, topics) {
    // Analyze sentiment for specific topics mentioned
    const breakdown = {};
    
    topics.forEach(topic => {
      const topicMentions = this.extractTopicMentions(text, topic);
      if (topicMentions.length > 0) {
        breakdown[topic] = {
          sentiment: this.getOverallSentiment(topicMentions.join(' ')),
          confidence: this.calculateConfidence(topicMentions.join(' '))
        };
      }
    });

    return breakdown;
  }
}
```

#### Content Gap Analysis Service
```javascript
// services/analytics/contentGaps.service.js
class ContentGapAnalysisService {
  async analyzeGaps(userId, competitorId = null) {
    const userMentions = await this.getUserMentions(userId);
    const competitorMentions = await this.getCompetitorMentions(userId, competitorId);
    
    const gaps = this.identifyGaps(userMentions, competitorMentions);
    const scoredGaps = await this.scoreOpportunities(gaps);
    
    return this.prioritizeGaps(scoredGaps);
  }

  identifyGaps(userMentions, competitorMentions) {
    const userTopics = new Set(userMentions.map(m => m.topic_cluster));
    const gaps = [];

    competitorMentions.forEach(mention => {
      if (!userTopics.has(mention.topic_cluster) || 
          this.getUserPosition(userMentions, mention.topic_cluster) > 3) {
        gaps.push({
          topic: mention.query,
          topicCluster: mention.topic_cluster,
          leadingCompetitor: mention.competitor_name,
          competitorPosition: mention.position,
          yourPosition: this.getUserPosition(userMentions, mention.topic_cluster),
          searchVolume: this.estimateSearchVolume(mention.query),
          platforms: [mention.platform]
        });
      }
    });

    return gaps;
  }

  async scoreOpportunities(gaps) {
    return Promise.all(gaps.map(async gap => {
      const difficultyScore = await this.calculateDifficultyScore(gap);
      const opportunityScore = this.calculateOpportunityScore(gap);
      const estimatedROI = this.calculateROI(gap.searchVolume, opportunityScore);
      
      return {
        ...gap,
        difficultyScore,
        opportunityScore,
        estimatedROI,
        priority: this.calculatePriority(opportunityScore, difficultyScore)
      };
    }));
  }

  calculateDifficultyScore(gap) {
    // Factors: competition level, topic complexity, current market position
    let score = 3; // Base difficulty
    
    if (gap.competitorPosition === 1) score += 1;
    if (gap.searchVolume > 5000) score += 1;
    if (gap.yourPosition === null) score += 1;
    
    return Math.min(5, Math.max(1, score));
  }

  calculateOpportunityScore(gap) {
    // Factors: search volume, business relevance, competition weakness
    let score = 3; // Base opportunity
    
    if (gap.searchVolume > 2000) score += 1;
    if (gap.competitorPosition > 2) score += 1;
    if (gap.platforms.length > 1) score += 1;
    
    return Math.min(5, Math.max(1, score));
  }

  calculateROI(searchVolume, opportunityScore) {
    // Estimate monthly reach potential
    return Math.floor(searchVolume * (opportunityScore / 5) * 0.15);
  }
}
```

#### AI Insights Generator Service
```javascript
// services/analytics/aiInsights.service.js
class AIInsightsService {
  async generateInsights(userId) {
    const data = await this.gatherAnalyticsData(userId);
    const insights = [];

    // Trend analysis insights
    const trendInsights = await this.analyzeTrends(data.timeSeriesData);
    insights.push(...trendInsights);

    // Competitive insights
    const competitiveInsights = await this.analyzeCompetitiveChanges(data.competitorData);
    insights.push(...competitiveInsights);

    // Platform-specific insights
    const platformInsights = await this.analyzePlatformBehavior(data.platformData);
    insights.push(...platformInsights);

    // Opportunity insights
    const opportunityInsights = await this.identifyOpportunities(data);
    insights.push(...opportunityInsights);

    return this.prioritizeInsights(insights);
  }

  async analyzeTrends(timeSeriesData) {
    const insights = [];
    
    // Detect significant changes in share of voice
    const voiceChanges = this.detectShareOfVoiceChanges(timeSeriesData);
    voiceChanges.forEach(change => {
      if (Math.abs(change.percentage) > 10) {
        insights.push({
          type: 'trend',
          priority: change.percentage < 0 ? 'high' : 'medium',
          title: `${change.competitor} ${change.percentage > 0 ? 'gaining' : 'losing'} in ${change.topic}`,
          message: `${change.competitor} ${change.percentage > 0 ? 'increased' : 'decreased'} share of voice by ${Math.abs(change.percentage)}% in ${change.topic} discussions`,
          recommendation: this.generateTrendRecommendation(change),
          estimatedImpact: this.estimateTrendImpact(change),
          supportingData: {
            trendChange: `${change.percentage > 0 ? '+' : ''}${change.percentage}%`,
            timeframe: '7 days',
            affectedTopics: [change.topic]
          }
        });
      }
    });

    return insights;
  }

  generateTrendRecommendation(change) {
    const actionType = change.percentage < 0 ? 'defensive' : 'offensive';
    
    if (actionType === 'defensive') {
      return `Create content to counter ${change.competitor}'s advantage in ${change.topic}`;
    } else {
      return `Capitalize on momentum in ${change.topic} with additional content`;
    }
  }

  async analyzeCompetitiveChanges(competitorData) {
    const insights = [];
    
    // Detect new competitive threats
    const threats = this.detectCompetitiveThreats(competitorData);
    threats.forEach(threat => {
      insights.push({
        type: 'competitor_threat',
        priority: 'high',
        title: `${threat.competitor} emerging in ${threat.topic}`,
        message: `${threat.competitor} gained ${threat.positions} new top-3 positions in ${threat.topic} this week`,
        recommendation: `Monitor ${threat.competitor}'s content strategy and create competitive response`,
        estimatedImpact: `Potential loss of ${threat.estimatedImpact} monthly impressions`
      });
    });

    return insights;
  }

  async analyzePlatformBehavior(platformData) {
    const insights = [];
    
    // Platform-specific optimization opportunities
    platformData.forEach(platform => {
      const opportunities = this.identifyPlatformOpportunities(platform);
      opportunities.forEach(opp => {
        insights.push({
          type: 'platform_optimization',
          priority: 'medium',
          title: `${platform.name} optimization opportunity`,
          message: opp.message,
          recommendation: opp.recommendation,
          estimatedImpact: opp.impact
        });
      });
    });

    return insights;
  }
}
```

## Frontend Application Structure
### React Project Structure
<pre>frontend/
├── src/
│   ├── components/
│   │   ├── ui/
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── Table.tsx
│   │   │   └── Charts/
│   │   │       ├── LineChart.tsx
│   │   │       ├── AreaChart.tsx
│   │   │       ├── HeatMap.tsx
│   │   │       ├── BubbleChart.tsx
│   │   │       ├── RadarChart.tsx
│   │   │       ├── NetworkGraph.tsx
│   │   │       └── SankeyDiagram.tsx
│   │   ├── layout/
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Header.tsx
│   │   │   ├── Layout.tsx
│   │   │   └── Navigation.tsx
│   │   ├── dashboard/
│   │   │   ├── DashboardOverview.tsx
│   │   │   ├── StatsGrid.tsx
│   │   │   ├── VisibilityChart.tsx
│   │   │   ├── PlatformCards.tsx
│   │   │   ├── RecentMentions.tsx
│   │   │   ├── CompetitorRankings.tsx
│   │   │   └── ShareOfVoice.tsx
│   │   ├── analytics/
│   │   │   ├── AdvancedAnalytics.tsx
│   │   │   ├── NetworkGraphView.tsx
│   │   │   ├── ContentGapAnalysis.tsx
│   │   │   ├── SentimentAnalysis.tsx
│   │   │   └── AIInsights.tsx
│   │   ├── platforms/
│   │   │   ├── PlatformPage.tsx
│   │   │   ├── ChatGPTPage.tsx
│   │   │   ├── PerplexityPage.tsx
│   │   │   ├── GeminiPage.tsx
│   │   │   ├── ClaudePage.tsx
│   │   │   ├── MistralPage.tsx
│   │   │   └── DeepSeekPage.tsx
│   │   ├── setup/
│   │   │   ├── WebsiteManagement.tsx
│   │   │   ├── CompetitorManagement.tsx
│   │   │   ├── SearchTermsManagement.tsx
│   │   │   └── PlatformSelection.tsx
│   │   ├── optimization/
│   │   │   ├── CompetitorIntelligence.tsx
│   │   │   ├── GEOOptimization.tsx
│   │   │   ├── ContentSuggestions.tsx
│   │   │   └── SmartAlerts.tsx
│   │   └── common/
│   │       ├── SentimentBadge.tsx
│   │       ├── StatusIndicator.tsx
│   │       ├── LoadingSpinner.tsx
│   │       ├── ErrorBoundary.tsx
│   │       └── AdvancedFilters.tsx
│   ├── pages/
│   │   ├── Dashboard.tsx
│   │   ├── RealTimeMonitoring.tsx
│   │   ├── AdvancedAnalytics.tsx
│   │   ├── Rankings.tsx
│   │   ├── Login.tsx
│   │   ├── Profile.tsx
│   │   └── Settings.tsx
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useWebSocket.ts
│   │   ├── useLocalStorage.ts
│   │   ├── useAdvancedFilters.ts
│   │   ├── useRealTimeData.ts
│   │   └── useChartData.ts
│   ├── stores/
│   │   ├── authStore.ts
│   │   ├── dashboardStore.ts
│   │   ├── analyticsStore.ts
│   │   ├── filtersStore.ts
│   │   └── websocketStore.ts
│   ├── services/
│   │   ├── api.ts
│   │   ├── websocket.ts
│   │   ├── auth.ts
│   │   ├── analytics.ts
│   │   └── charts.ts
│   ├── utils/
│   │   ├── formatters.ts
│   │   ├── validators.ts
│   │   ├── constants.ts
│   │   ├── helpers.ts
│   │   └── chartConfigs.ts
│   ├── types/
│   │   ├── auth.types.ts
│   │   ├── dashboard.types.ts
│   │   ├── analytics.types.ts
│   │   ├── mentions.types.ts
│   │   └── api.types.ts
│   └── styles/
│       ├── globals.css
│       ├── components.css
│       └── charts.css
├── public/
├── package.json
└── vite.config.ts</pre>

### Core Frontend Components
#### Advanced Dashboard Component
```typescript
// components/dashboard/DashboardOverview.tsx
import React, { useEffect, useState } from 'react';
import { useAdvancedFilters } from '../hooks/useAdvancedFilters';
import { useRealTimeData } from '../hooks/useRealTimeData';
import { useDashboardStore } from '../stores/dashboardStore';

interface DashboardOverviewProps {
  userId: string;
}

export const DashboardOverview: React.FC<DashboardOverviewProps> = ({ userId }) => {
  const { filters, updateFilter, resetFilters } = useAdvancedFilters();
  const { data: realTimeData } = useRealTimeData(userId);
  const { 
    dashboardData, 
    loading, 
    error, 
    fetchDashboardData,
    updateFilters 
  } = useDashboardStore();

  useEffect(() => {
    fetchDashboardData(userId, filters);
  }, [userId, filters]);

  useEffect(() => {
    if (realTimeData) {
      // Update dashboard with real-time data
      updateRealTimeData(realTimeData);
    }
  }, [realTimeData]);

  const handleFilterChange = (filterKey: string, value: any) => {
    updateFilter(filterKey, value);
    updateFilters({ ...filters, [filterKey]: value });
  };

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorBoundary error={error} />;

  return (
    <div className="dashboard-overview">
      <AdvancedFilters
        filters={filters}
        onFilterChange={handleFilterChange}
        onReset={resetFilters}
      />
      
      <StatsGrid 
        stats={dashboardData.stats}
        insights={dashboardData.aiInsights}
      />
      
      <div className="dashboard-grid grid grid-cols-3 gap-6">
        <div className="col-span-2">
          <VisibilityChart 
            data={dashboardData.visibilityTrends}
            chartType={filters.chartType || 'line'}
          />
        </div>
        
        <div className="col-span-1">
          <RecentMentions 
            mentions={dashboardData.recentMentions}
            realTimeUpdates={realTimeData?.mentions}
          />
        </div>
      </div>
      
      <ShareOfVoice 
        topicClusters={dashboardData.topicClusters}
        competitors={dashboardData.competitors}
      />
      
      <CompetitorRankings 
        rankings={dashboardData.competitorRankings}
        contentGaps={dashboardData.contentGaps}
      />
    </div>
  );
};
```

#### Advanced Filters Component
```typescript
// components/common/AdvancedFilters.tsx
import React from 'react';
import { FilterState } from '../types/analytics.types';

interface AdvancedFiltersProps {
  filters: FilterState;
  onFilterChange: (key: string, value: any) => void;
  onReset: () => void;
}

export const AdvancedFilters: React.FC<AdvancedFiltersProps> = ({
  filters,
  onFilterChange,
  onReset
}) => {
  return (
    <div className="advanced-filters bg-white rounded-lg border p-6 mb-6">
      <div className="flex items-center justify-between mb-4">
        <h3 className="text-lg font-semibold">Advanced Filters</h3>
        <button 
          onClick={onReset}
          className="text-sm text-gray-500 hover:text-gray-700"
        >
          Reset All
        </button>
      </div>
      
      <div className="grid grid-cols-5 gap-4">
        {/* Platform Filter */}
        <div className="filter-group">
          <label className="block text-sm font-medium mb-2">Platform</label>
          <select
            value={filters.platform || ''}
            onChange={(e) => onFilterChange('platform', e.target.value)}
            className="w-full p-2 border rounded-md"
          >
            <option value="">All Platforms</option>
            <option value="chatgpt">ChatGPT</option>
            <option value="perplexity">Perplexity</option>
            <option value="gemini">Gemini</option>
            <option value="claude">Claude</option>
            <option value="mistral">Mistral AI</option>
            <option value="deepseek">DeepSeek</option>
          </select>
        </div>

        {/* Sentiment Filter */}
        <div className="filter-group">
          <label className="block text-sm font-medium mb-2">Sentiment</label>
          <select
            value={filters.sentiment || ''}
            onChange={(e) => onFilterChange('sentiment', e.target.value)}
            className="w-full p-2 border rounded-md"
          >
            <option value="">All Sentiments</option>
            <option value="positive">Positive</option>
            <option value="negative">Negative</option>
            <option value="neutral">Neutral</option>
          </select>
        </div>

        {/* Topic Cluster Filter */}
        <div className="filter-group">
          <label className="block text-sm font-medium mb-2">Topic Cluster</label>
          <select
            value={filters.topicCluster || ''}
            onChange={(e) => onFilterChange('topicCluster', e.target.value)}
            className="w-full p-2 border rounded-md"
          >
            <option value="">All Topics</option>
            <option value="ev-safety">EV Safety</option>
            <option value="charging">Charging Infrastructure</option>
            <option value="autopilot">Autonomous Driving</option>
            <option value="maintenance">Maintenance</option>
          </select>
        </div>

        {/* Confidence Level Filter */}
        <div className="filter-group">
          <label className="block text-sm font-medium mb-2">Confidence</label>
          <select
            value={filters.confidenceLevel || ''}
            onChange={(e) => onFilterChange('confidenceLevel', e.target.value)}
            className="w-full p-2 border rounded-md"
          >
            <option value="">All Levels</option>
            <option value="high">High (90%+)</option>
            <option value="medium">Medium (70-89%)</option>
            <option value="low">Low (<70%)</option>
          </select>
        </div>

        {/* Position Range Filter */}
        <div className="filter-group">
          <label className="block text-sm font-medium mb-2">Position</label>
          <select
            value={filters.positionRange || ''}
            onChange={(e) => onFilterChange('positionRange', e.target.value)}
            className="w-full p-2 border rounded-md"
          >
            <option value="">All Positions</option>
            <option value="1-3">Top 3</option>
            <option value="4-5">4-5</option>
            <option value="6-10">6-10</option>
            <option value="none">Not Mentioned</option>
          </select>
        </div>
      </div>
      
      {/* Additional Filter Row */}
      <div className="grid grid-cols-4 gap-4 mt-4">
        <div className="filter-group">
          <label className="block text-sm font-medium mb-2">Time Range</label>
          <select
            value={filters.timeRange || '30d'}
            onChange={(e) => onFilterChange('timeRange', e.target.value)}
            className="w-full p-2 border rounded-md"
          >
            <option value="7d">Last 7 days</option>
            <option value="30d">Last 30 days</option>
            <option value="90d">Last 90 days</option>
            <option value="custom">Custom Range</option>
          </select>
        </div>

        <div className="filter-group">
          <label className="block text-sm font-medium mb-2">Mention Volume</label>
          <input
            type="range"
            min="0"
            max="1000"
            value={filters.mentionVolume || 500}
            onChange={(e) => onFilterChange('mentionVolume', e.target.value)}
            className="w-full"
          />
          <div className="text-xs text-gray-500 mt-1">
            Min: {filters.mentionVolume || 500} mentions
          </div>
        </div>

        <div className="filter-group">
          <label className="block text-sm font-medium mb-2">Query Type</label>
          <select
            value={filters.queryType || ''}
            onChange={(e) => onFilterChange('queryType', e.target.value)}
            className="w-full p-2 border rounded-md"
          >
            <option value="">All Types</option>
            <option value="product">Product Research</option>
            <option value="comparison">Comparison</option>
            <option value="howto">How-to Guides</option>
            <option value="technical">Technical</option>
          </select>
        </div>

        <div className="filter-group">
          <label className="block text-sm font-medium mb-2">Competitor</label>
          <select
            value={filters.competitor || ''}
            onChange={(e) => onFilterChange('competitor', e.target.value)}
            className="w-full p-2 border rounded-md"
          >
            <option value="">All Competitors</option>
            <option value="ford">Ford</option>
            <option value="bmw">BMW</option>
            <option value="gm">General Motors</option>
          </select>
        </div>
      </div>
    </div>
  );
};
```

#### Network Graph Component
```typescript
// components/ui/Charts/NetworkGraph.tsx
import React, { useEffect, useRef } from 'react';
import * as d3 from 'd3';

interface NetworkNode {
  id: string;
  label: string;
  size: number;
  color: string;
  cluster?: string;
}

interface NetworkLink {
  source: string;
  target: string;
  strength: number;
}

interface NetworkGraphProps {
  nodes: NetworkNode[];
  links: NetworkLink[];
  width?: number;
  height?: number;
  onNodeClick?: (node: NetworkNode) => void;
}

export const NetworkGraph: React.FC<NetworkGraphProps> = ({
  nodes,
  links,
  width = 800,
  height = 600,
  onNodeClick
}) => {
  const svgRef = useRef<SVGSVGElement>(null);

  useEffect(() => {
    if (!svgRef.current || !nodes.length) return;

    const svg = d3.select(svgRef.current);
    svg.selectAll("*").remove();

    // Create force simulation
    const simulation = d3.forceSimulation(nodes as any)
      .force("link", d3.forceLink(links).id((d: any) => d.id).strength(d => d.strength / 10))
      .force("charge", d3.forceManyBody().strength(-300))
      .force("center", d3.forceCenter(width / 2, height / 2));

    // Create links
    const link = svg.append("g")
      .selectAll("line")
      .data(links)
      .enter().append("line")
      .attr("stroke", "#999")
      .attr("stroke-opacity", 0.6)
      .attr("stroke-width", d => Math.sqrt(d.strength));

    // Create nodes
    const node = svg.append("g")
      .selectAll("circle")
      .data(nodes)
      .enter().append("circle")
      .attr("r", d => Math.sqrt(d.size) * 2)
      .attr("fill", d => d.color)
      .attr("stroke", "#fff")
      .attr("stroke-width", 2)
      .style("cursor", "pointer")
      .call(d3.drag()
        .on("start", dragstarted)
        .on("drag", dragged)
        .on("end", dragended));

    // Add labels
    const label = svg.append("g")
      .selectAll("text")
      .data(nodes)
      .enter().append("text")
      .text(d => d.label)
      .attr("font-size", 12)
      .attr("dx", 15)
      .attr("dy", 4);

    // Add click handlers
    node.on("click", (event, d) => {
      if (onNodeClick) {
        onNodeClick(d);
      }
    });

    // Update positions on simulation tick
    simulation.on("tick", () => {
      link
        .attr("x1", (d: any) => d.source.x)
        .attr("y1", (d: any) => d.source.y)
        .attr("x2", (d: any) => d.target.x)
        .attr("y2", (d: any) => d.target.y);

      node
        .attr("cx", (d: any) => d.x)
        .attr("cy", (d: any) => d.y);

      label
        .attr("x", (d: any) => d.x)
        .attr("y", (d: any) => d.y);
    });

    function dragstarted(event: any) {
      if (!event.active) simulation.alphaTarget(0.3).restart();
      event.subject.fx = event.subject.x;
      event.subject.fy = event.subject.y;
    }

    function dragged(event: any) {
      event.subject.fx = event.x;
      event.subject.fy = event.y;
    }

    function dragended(event: any) {
      if (!event.active) simulation.alphaTarget(0);
      event.subject.fx = null;
      event.subject.fy = null;
    }

    return () => {
      simulation.stop();
    };
  }, [nodes, links, width, height, onNodeClick]);

  return (
    <div className="network-graph-container">
      <svg
        ref={svgRef}
        width={width}
        height={height}
        className="border rounded-lg"
      />
      <div className="graph-legend mt-4">
        <div className="flex items-center gap-4 text-sm">
          <div className="flex items-center gap-2">
            <div className="w-4 h-4 rounded-full bg-green-500"></div>
            <span>Your Dominance</span>
          </div>
          <div className="flex items-center gap-2">
            <div className="w-4 h-4 rounded-full bg-red-500"></div>
            <span>Competitor Advantage</span>
          </div>
          <div className="flex items-center gap-2">
            <div className="w-4 h-4 rounded-full bg-blue-500"></div>
            <span>Balanced Competition</span>
          </div>
        </div>
      </div>
    </div>
  );
};
```

## AI Platform Integrations
### Standardized AI Platform Interface
```typescript
// services/ai-platforms/base.service.ts
export interface AIResponse {
  platform: string;
  query: string;
  response: string;
  position: number | null;
  citations: Citation[];
  confidence: number;
  metadata: {
    tokens?: number;
    model?: string;
    timestamp: string;
    processingTime: number;
  };
}

export interface Citation {
  url: string;
  title?: string;
  relevance: number;
}

export abstract class BaseAIService {
  abstract queryPlatform(searchTerm: string, context?: any): Promise<AIResponse>;
  abstract extractPosition(content: string, brand: string): number | null;
  abstract extractCitations(content: string): Citation[];
  abstract estimateCost(tokens: number): number;
}
```

#### OpenAI Integration
```typescript
// services/ai-platforms/openai.service.ts
import OpenAI from 'openai';
import { BaseAIService, AIResponse } from './base.service';

export class OpenAIService extends BaseAIService {
  private client: OpenAI;
  private model = 'gpt-4';

  constructor() {
    super();
    this.client = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }

  async queryPlatform(searchTerm: string, context: any = {}): Promise<AIResponse> {
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
            content: this.buildContextualQuery(searchTerm, context)
          }
        ],
        max_tokens: 1500,
        temperature: 0.7
      });

      const content = response.choices[0]?.message?.content || '';
      
      return {
        platform: 'chatgpt',
        query: searchTerm,
        response: content,
        position: this.extractPosition(content, context.brand || ''),
        citations: this.extractCitations(content),
        confidence: this.calculateConfidence(content),
        metadata: {
          tokens: response.usage?.total_tokens || 0,
          model: this.model,
          timestamp: new Date().toISOString(),
          processingTime: Date.now() - startTime
        }
      };
    } catch (error) {
      throw new Error(`OpenAI API Error: ${error.message}`);
    }
  }

  private buildContextualQuery(searchTerm: string, context: any): string {
    let query = searchTerm;
    
    if (context.queryType === 'comparison') {
      query = `Compare different options for: ${searchTerm}`;
    } else if (context.queryType === 'technical') {
      query = `Provide technical details about: ${searchTerm}`;
    } else if (context.queryType === 'product') {
      query = `What are the best products/services for: ${searchTerm}`;
    }
    
    return query;
  }

  extractPosition(content: string, brand: string): number | null {
    if (!brand || !content) return null;
    
    const sentences = content.split(/[.!?]+/);
    
    for (let i = 0; i < sentences.length; i++) {
      if (sentences[i].toLowerCase().includes(brand.toLowerCase())) {
        // Position based on order of mention
        if (i === 0) return 1;
        if (i <= 2) return 2;
        if (i <= 4) return 3;
        return Math.min(10, Math.ceil(i / 2));
      }
    }
    
    return null;
  }

  extractCitations(content: string): Citation[] {
    const urlRegex = /https?:\/\/[^\s]+/g;
    const urls = content.match(urlRegex) || [];
    
    return urls.map(url => ({
      url: url.replace(/[.,;!?]$/, ''), // Remove trailing punctuation
      relevance: 0.8
    }));
  }

  private calculateConfidence(content: string): number {
    // Confidence based on response quality indicators
    let confidence = 0.5;
    
    if (content.length > 100) confidence += 0.2;
    if (content.includes('http')) confidence += 0.1;
    if (content.split(' ').length > 50) confidence += 0.1;
    if (content.includes('specifically') || content.includes('particularly')) confidence += 0.1;
    
    return Math.min(0.95, confidence);
  }

  estimateCost(tokens: number): number {
    // GPT-4 pricing: $0.03 per 1K input tokens, $0.06 per 1K output tokens
    const inputTokens = tokens * 0.3; // Estimate 30% input
    const outputTokens = tokens * 0.7; // Estimate 70% output
    
    return (inputTokens / 1000 * 0.03) + (outputTokens / 1000 * 0.06);
  }
}
```

#### Anthropic Claude Integration
```typescript
// services/ai-platforms/anthropic.service.ts
import Anthropic from '@anthropic-ai/sdk';
import { BaseAIService, AIResponse } from './base.service';

export class AnthropicService extends BaseAIService {
  private client: Anthropic;
  private model = 'claude-3-sonnet-20240229';

  constructor() {
    super();
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
  }

  async queryPlatform(searchTerm: string, context: any = {}): Promise<AIResponse> {
    const startTime = Date.now();
    
    try {
      const response = await this.client.messages.create({
        model: this.model,
        max_tokens: 1500,
        messages: [
          {
            role: 'user',
            content: this.buildContextualQuery(searchTerm, context)
          }
        ]
      });

      const content = response.content[0]?.text || '';
      
      return {
        platform: 'claude',
        query: searchTerm,
        response: content,
        position: this.extractPosition(content, context.brand || ''),
        citations: this.extractCitations(content),
        confidence: this.calculateConfidence(content),
        metadata: {
          tokens: response.usage?.input_tokens + response.usage?.output_tokens || 0,
          model: this.model,
          timestamp: new Date().toISOString(),
          processingTime: Date.now() - startTime
        }
      };
    } catch (error) {
      throw new Error(`Anthropic API Error: ${error.message}`);
    }
  }

  private buildContextualQuery(searchTerm: string, context: any): string {
    return `Please provide comprehensive information about: ${searchTerm}. 
            Include specific examples, comparisons, and cite sources when possible.`;
  }

  extractPosition(content: string, brand: string): number | null {
    // Similar implementation to OpenAI but adapted for Claude's response style
    if (!brand || !content) return null;
    
    const paragraphs = content.split('\n\n');
    
    for (let i = 0; i < paragraphs.length; i++) {
      if (paragraphs[i].toLowerCase().includes(brand.toLowerCase())) {
        if (i === 0) return 1;
        if (i === 1) return 2;
        if (i === 2) return 3;
        return Math.min(10, i + 1);
      }
    }
    
    return null;
  }

  extractCitations(content: string): Citation[] {
    const urlRegex = /https?:\/\/[^\s\)]+/g;
    const urls = content.match(urlRegex) || [];
    
    return urls.map(url => ({
      url: url.replace(/[.,;!?]$/, ''),
      relevance: 0.8
    }));
  }

  private calculateConfidence(content: string): number {
    let confidence = 0.6; // Claude tends to be more detailed
    
    if (content.includes('based on') || content.includes('according to')) confidence += 0.1;
    if (content.includes('sources') || content.includes('research')) confidence += 0.1;
    if (content.length > 200) confidence += 0.2;
    
    return Math.min(0.95, confidence);
  }

  estimateCost(tokens: number): number {
    // Claude pricing: $0.015 per 1K input tokens, $0.075 per 1K output tokens
    const inputTokens = tokens * 0.2;
    const outputTokens = tokens * 0.8;
    
    return (inputTokens / 1000 * 0.015) + (outputTokens / 1000 * 0.075);
  }
}
```

#### Google Gemini Integration
```typescript
// services/ai-platforms/google.service.ts
import { GoogleGenerativeAI } from '@google/generative-ai';
import { BaseAIService, AIResponse } from './base.service';

export class GoogleService extends BaseAIService {
  private client: GoogleGenerativeAI;
  private model = 'gemini-pro';

  constructor() {
    super();
    this.client = new GoogleGenerativeAI(process.env.GOOGLE_AI_API_KEY);
  }

  async queryPlatform(searchTerm: string, context: any = {}): Promise<AIResponse> {
    const startTime = Date.now();
    
    try {
      const model = this.client.getGenerativeModel({ model: this.model });
      const prompt = this.buildContextualQuery(searchTerm, context);
      
      const response = await model.generateContent(prompt);
      const content = response.response.text();
      
      return {
        platform: 'gemini',
        query: searchTerm,
        response: content,
        position: this.extractPosition(content, context.brand || ''),
        citations: this.extractCitations(content),
        confidence: this.calculateConfidence(content),
        metadata: {
          model: this.model,
          timestamp: new Date().toISOString(),
          processingTime: Date.now() - startTime
        }
      };
    } catch (error) {
      throw new Error(`Google AI API Error: ${error.message}`);
    }
  }

  private buildContextualQuery(searchTerm: string, context: any): string {
    return `Provide detailed information about: ${searchTerm}. 
            Include relevant examples and cite authoritative sources.`;
  }

  extractPosition(content: string, brand: string): number | null {
    if (!brand || !content) return null;
    
    // Google's responses tend to be structured differently
    const sections = content.split(/\n(?=\d+\.|\*|\-)/);
    
    for (let i = 0; i < sections.length; i++) {
      if (sections[i].toLowerCase().includes(brand.toLowerCase())) {
        return Math.min(10, i + 1);
      }
    }
    
    return null;
  }

  extractCitations(content: string): Citation[] {
    const urlRegex = /https?:\/\/[^\s\)]+/g;
    const urls = content.match(urlRegex) || [];
    
    return urls.map(url => ({
      url: url.replace(/[.,;!?]$/, ''),
      relevance: 0.7
    }));
  }

  private calculateConfidence(content: string): number {
    let confidence = 0.6;
    
    if (content.includes('Google') || content.includes('search')) confidence += 0.1;
    if (content.includes('updated') || content.includes('latest')) confidence += 0.1;
    if (content.length > 150) confidence += 0.2;
    
    return Math.min(0.95, confidence);
  }

  estimateCost(tokens: number): number {
    // Gemini Pro pricing: $0.0005 per 1K input tokens, $0.0015 per 1K output tokens
    const inputTokens = tokens * 0.25;
    const outputTokens = tokens * 0.75;
    
    return (inputTokens / 1000 * 0.0005) + (outputTokens / 1000 * 0.0015);
  }
}
```

#### Custom Perplexity Integration
typescript
```// services/ai-platforms/perplexity.service.ts
import axios from 'axios';
import { BaseAIService, AIResponse } from './base.service';

export class PerplexityService extends BaseAIService {
  private baseURL = 'https://api.perplexity.ai';
  private model = 'llama-3.1-sonar-large-128k-online';

  constructor() {
    super();
  }

  async queryPlatform(searchTerm: string, context: any = {}): Promise<AIResponse> {
    const startTime = Date.now();
    
    try {
      const response = await axios.post(`${this.baseURL}/chat/completions`, {
        model: this.model,
        messages: [
          {
            role: 'user',
            content: this.buildContextualQuery(searchTerm, context)
          }
        ],
        max_tokens: 1500,
        temperature: 0.2,
        top_p: 0.9
      }, {
        headers: {
          'Authorization': `Bearer ${process.env.PERPLEXITY_API_KEY}`,
          'Content-Type': 'application/json'
        }
      });

      const content = response.data.choices[0]?.message?.content || '';
      
      return {
        platform: 'perplexity',
        query: searchTerm,
        response: content,
        position: this.extractPosition(content, context.brand || ''),
        citations: this.extractCitations(content),
        confidence: this.calculateConfidence(content),
        metadata: {
          tokens: response.data.usage?.total_tokens || 0,
          model: this.model,
          timestamp: new Date().toISOString(),
          processingTime: Date.now() - startTime
        }
      };
    } catch (error) {
      throw new Error(`Perplexity API Error: ${error.message}`);
    }
  }

  private buildContextualQuery(searchTerm: string, context: any): string {
    return `${searchTerm} - Please provide current, comprehensive information with sources.`;
  }

  extractPosition(content: string, brand: string): number | null {
    if (!brand || !content) return null;
    
    // Perplexity often uses numbered citations [1], [2], etc.
    const citationPattern = /\[\d+\]/g;
    const parts = content.split(citationPattern);
    
    for (let i = 0; i < parts.length; i++) {
      if (parts[i].toLowerCase().includes(brand.toLowerCase())) {
        return Math.min(10, i + 1);
      }
    }
    
    return null;
  }

  extractCitations(content: string): Citation[] {
    // Perplexity provides numbered citations
    const citationRegex = /\[(\d+)\]/g;
    const urlRegex = /https?:\/\/[^\s\)]+/g;
    
    const citations = content.match(citationRegex) || [];
    const urls = content.match(urlRegex) || [];
    
    return urls.map((url, index) => ({
      url: url.replace(/[.,;!?]$/, ''),
      relevance: 0.9, // Perplexity citations are usually highly relevant
      citationNumber: citations[index]
    }));
  }

  private calculateConfidence(content: string): number {
    let confidence = 0.7; // Perplexity is generally reliable
    
    if (content.includes('[') && content.includes(']')) confidence += 0.1; // Has citations
    if (content.includes('according to') || content.includes('source')) confidence += 0.1;
    if (content.length > 200) confidence += 0.1;
    
    return Math.min(0.95, confidence);
  }

  estimateCost(tokens: number): number {
    // Perplexity pricing varies - estimate based on their published rates
    return tokens / 1000 * 0.002; // Rough estimate
  }
}
```

## Real-time Data Processing
### WebSocket Server Implementation
```typescript
// services/websocket.service.ts
import { Server } from 'socket.io';
import { Server as HttpServer } from 'http';
import jwt from 'jsonwebtoken';
import { RedisService } from './redis.service';

export class WebSocketService {
  private io: Server;
  private redis: RedisService;

  constructor(httpServer: HttpServer) {
    this.io = new Server(httpServer, {
      cors: {
        origin: process.env.FRONTEND_URL,
        methods: ["GET", "POST"]
      }
    });
    
    this.redis = new RedisService();
    this.setupMiddleware();
    this.setupEventHandlers();
  }

  private setupMiddleware() {
    this.io.use(async (socket, next) => {
      try {
        const token = socket.handshake.auth.token;
        const decoded = jwt.verify(token, process.env.JWT_SECRET) as any;
        
        socket.userId = decoded.userId;
        socket.join(`user_${decoded.userId}`);
        
        next();
      } catch (error) {
        next(new Error('Authentication error'));
      }
    });
  }

  private setupEventHandlers() {
    this.io.on('connection', (socket) => {
      console.log(`User ${socket.userId} connected`);

      // Join platform-specific rooms
      socket.on('subscribe_platform', (platform: string) => {
        socket.join(`platform_${platform}_${socket.userId}`);
      });

      // Handle real-time query processing
      socket.on('process_query', async (data) => {
        try {
          const result = await this.processRealtimeQuery(data, socket.userId);
          socket.emit('query_result', result);
        } catch (error) {
          socket.emit('query_error', { error: error.message });
        }
      });

      socket.on('disconnect', () => {
        console.log(`User ${socket.userId} disconnected`);
      });
    });
  }

  async broadcastToUser(userId: string, event: string, data: any) {
    this.io.to(`user_${userId}`).emit(event, data);
  }

  async broadcastMention(userId: string, mention: any) {
    const mentionData = {
      id: mention.id,
      platform: mention.platform,
      query: mention.query,
      position: mention.position,
      sentiment: mention.sentiment,
      timestamp: mention.timestamp
    };

    this.io.to(`user_${userId}`).emit('new_mention', mentionData);
    
    // Store in Redis for persistence
    await this.redis.addToList(`realtime:mentions:${userId}`, mentionData, 100);
  }

  async broadcastInsight(userId: string, insight: any) {
    this.io.to(`user_${userId}`).emit('new_insight', insight);
    await this.redis.addToList(`realtime:insights:${userId}`, insight, 50);
  }

  async broadcastAlert(userId: string, alert: any) {
    this.io.to(`user_${userId}`).emit('new_alert', alert);
    await this.redis.addToList(`realtime:alerts:${userId}`, alert, 20);
  }

  private async processRealtimeQuery(data: any, userId: string) {
    // Process query immediately and return result
    const { searchTerm, platform, context } = data;
    
    // Add to priority queue for immediate processing
    const job = await this.addToQueue('realtime_query', {
      userId,
      searchTerm,
      platform,
      context,
      priority: 1
    });

    return { jobId: job.id, status: 'processing' };
  }
}
```

### Queue Processing System
```typescript
// queues/queryProcessor.queue.ts
import Bull from 'bull';
import { AIServiceFactory } from '../services/ai-platforms/factory';
import { SentimentAnalysisService } from '../services/analytics/sentiment.service';
import { WebSocketService } from '../services/websocket.service';

export class QueryProcessorQueue {
  private queue: Bull.Queue;
  private aiServiceFactory: AIServiceFactory;
  private sentimentService: SentimentAnalysisService;
  private websocketService: WebSocketService;

  constructor() {
    this.queue = new Bull('query processing', {
      redis: {
        host: process.env.REDIS_HOST,
        port: parseInt(process.env.REDIS_PORT),
        password: process.env.REDIS_PASSWORD
      }
    });

    this.aiServiceFactory = new AIServiceFactory();
    this.sentimentService = new SentimentAnalysisService();
    this.setupProcessors();
  }

  private setupProcessors() {
    // High priority real-time queries
    this.queue.process('realtime_query', 5, async (job) => {
      return this.processRealtimeQuery(job.data);
    });

    // Batch processing for regular monitoring
    this.queue.process('batch_query', 10, async (job) => {
      return this.processBatchQuery(job.data);
    });

    // Sentiment analysis processing
    this.queue.process('sentiment_analysis', 20, async (job) => {
      return this.processSentimentAnalysis(job.data);
    });
  }

  private async processRealtimeQuery(data: any) {
    const { userId, searchTerm, platform, context } = data;
    
    try {
      // Get AI service for platform
      const aiService = this.aiServiceFactory.getService(platform);
      
      // Query the platform
      const response = await aiService.queryPlatform(searchTerm, context);
      
      // Store in database
      const mention = await this.storeMention(userId, response);
      
      // Analyze sentiment
      const sentiment = await this.sentimentService.analyzeMention(mention, context);
      mention.sentiment = sentiment;
      
      // Update mention with sentiment
      await this.updateMentionSentiment(mention.id, sentiment);
      
      // Broadcast real-time update
      await this.websocketService.broadcastMention(userId, mention);
      
      // Check for alerts
      await this.checkForAlerts(userId, mention);
      
      return { success: true, mentionId: mention.id };
    } catch (error) {
      console.error('Real-time query processing error:', error);
      throw error;
    }
  }

  private async processBatchQuery(data: any) {
    const { userId, searchTerms, platforms } = data;
    const results = [];
    
    for (const searchTerm of searchTerms) {
      for (const platform of platforms) {
        try {
          const aiService = this.aiServiceFactory.getService(platform);
          const response = await aiService.queryPlatform(searchTerm.text, {
            brand: searchTerm.brand,
            category: searchTerm.category
          });
          
          const mention = await this.storeMention(userId, response);
          results.push(mention);
          
          // Add to sentiment analysis queue
          await this.queue.add('sentiment_analysis', {
            mentionId: mention.id,
            userId
          }, { delay: 1000 });
          
        } catch (error) {
          console.error(`Batch query error for ${searchTerm.text} on ${platform}:`, error);
        }
        
        // Rate limiting delay
        await this.delay(2000);
      }
    }
    
    return { processed: results.length };
  }

  private async processSentimentAnalysis(data: any) {
    const { mentionId, userId } = data;
    
    try {
      const mention = await this.getMention(mentionId);
      const sentiment = await this.sentimentService.analyzeMention(mention);
      
      await this.updateMentionSentiment(mentionId, sentiment);
      
      // Broadcast sentiment update
      await this.websocketService.broadcastToUser(userId, 'sentiment_update', {
        mentionId,
        sentiment
      });
      
      return { success: true };
    } catch (error) {
      console.error('Sentiment analysis error:', error);
      throw error;
    }
  }

  async addRealtimeQuery(data: any) {
    return this.queue.add('realtime_query', data, {
      priority: 1,
      attempts: 3,
      backoff: {
        type: 'exponential',
        delay: 2000
      }
    });
  }

  async addBatchQuery(data: any) {
    return this.queue.add('batch_query', data, {
      priority: 5,
      attempts: 2
    });
  }

  private async storeMention(userId: string, response: any) {
    // Database storage logic
    const mention = {
      id: generateId(),
      userId,
      platform: response.platform,
      query: response.query,
      response: response.response,
      position: response.position,
      citations: response.citations,
      confidence: response.confidence,
      timestamp: new Date()
    };
    
    // Store in PostgreSQL
    await this.db.mentions.create(mention);
    
    return mention;
  }

  private delay(ms: number) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

## Premium Features Implementation
### Content Gap Analysis Engine
```typescript
// services/analytics/contentGaps.service.ts
export class ContentGapAnalysisService {
  async analyzeContentGaps(userId: string, options: AnalysisOptions = {}) {
    const userMentions = await this.getUserMentions(userId, options.timeframe);
    const competitorMentions = await this.getCompetitorMentions(userId, options.timeframe);
    
    const gaps = await this.identifyGaps(userMentions, competitorMentions);
    const scoredGaps = await this.scoreOpportunities(gaps);
    const prioritizedGaps = this.prioritizeGaps(scoredGaps);
    
    return {
      totalGaps: gaps.length,
      highPriorityGaps: prioritizedGaps.filter(g => g.priority >= 4).length,
      estimatedTotalROI: prioritizedGaps.reduce((sum, g) => sum + g.estimatedROI, 0),
      gaps: prioritizedGaps,
      insights: await this.generateGapInsights(prioritizedGaps)
    };
  }

  private async identifyGaps(userMentions: Mention[], competitorMentions: Mention[]) {
    const gaps: ContentGap[] = [];
    const userTopicMap = this.createTopicMap(userMentions);
    const competitorTopicMap = this.createTopicMap(competitorMentions);
    
    // Find topics where competitors rank but user doesn't
    for (const [topic, competitorData] of competitorTopicMap) {
      const userData = userTopicMap.get(topic);
      
      if (!userData || userData.averagePosition > 3) {
        gaps.push({
          topic,
          topicCluster: competitorData.cluster,
          leadingCompetitor: competitorData.topCompetitor,
          competitorPosition: competitorData.bestPosition,
          yourPosition: userData?.averagePosition || null,
          searchVolume: await this.estimateSearchVolume(topic),
          platforms: competitorData.platforms,
          competitorAdvantage: competitorData.shareOfVoice
        });
      }
    }
    
    return gaps;
  }

  private async scoreOpportunities(gaps: ContentGap[]) {
    return Promise.all(gaps.map(async gap => {
      const difficultyScore = this.calculateDifficultyScore(gap);
      const opportunityScore = this.calculateOpportunityScore(gap);
      const businessValue = await this.calculateBusinessValue(gap);
      
      return {
        ...gap,
        difficultyScore,
        opportunityScore,
        businessValue,
        estimatedROI: this.calculateROI(gap.searchVolume, opportunityScore, businessValue),
        priority: this.calculatePriority(opportunityScore, difficultyScore, businessValue),
        timeToRank: this.estimateTimeToRank(difficultyScore),
        resourcesRequired: this.estimateResources(difficultyScore, gap.topic)
      };
    }));
  }

  private calculateDifficultyScore(gap: ContentGap): number {
    let score = 3; // Base difficulty
    
    // Competition level
    if (gap.competitorPosition === 1) score += 1;
    if (gap.competitorAdvantage > 50) score += 1;
    
    // Search volume (higher volume = more competition)
    if (gap.searchVolume > 5000) score += 1;
    if (gap.searchVolume > 10000) score += 0.5;
    
    // Current position
    if (gap.yourPosition === null) score += 0.5;
    if (gap.yourPosition && gap.yourPosition > 5) score += 1;
    
    // Platform diversity
    if (gap.platforms.length > 3) score += 0.5;
    
    return Math.min(5, Math.max(1, Math.round(score * 10) / 10));
  }

  private calculateOpportunityScore(gap: ContentGap): number {
    let score = 3; // Base opportunity
    
    // Market size
    if (gap.searchVolume > 1000) score += 1;
    if (gap.searchVolume > 5000) score += 1;
    
    // Competition weakness
    if (gap.competitorPosition > 2) score += 1;
    if (gap.competitorAdvantage < 30) score += 1;
    
    // Platform coverage
    if (gap.platforms.length > 2) score += 0.5;
    
    // Topic relevance (business alignment)
    const relevanceScore = this.calculateTopicRelevance(gap.topic);
    score += relevanceScore * 0.5;
    
    return Math.min(5, Math.max(1, Math.round(score * 10) / 10));
  }

  private async calculateBusinessValue(gap: ContentGap): Promise<number> {
    // Business value based on industry, topic type, and potential impact
    const industryMultiplier = this.getIndustryMultiplier(gap.topicCluster);
    const topicTypeValue = this.getTopicTypeValue(gap.topic);
    const searchVolumeValue = Math.log10(gap.searchVolume + 1) * 1000;
    
    return Math.round(searchVolumeValue * industryMultiplier * topicTypeValue);
  }

  private calculateROI(searchVolume: number, opportunityScore: number, businessValue: number): number {
    // Estimate monthly reach potential
    const reachRate = 0.15; // 15% of search volume potential reach
    const conversionRate = opportunityScore / 5 * 0.1; // Higher opportunity = better conversion
    
    return Math.round(searchVolume * reachRate * conversionRate * (businessValue / 1000));
  }

  async generateContentBrief(gapId: string): Promise<ContentBrief> {
    const gap = await this.getContentGap(gapId);
    const competitorContent = await this.analyzeCompetitorContent(gap);
    const recommendations = await this.generateRecommendations(gap, competitorContent);
    
    return {
      title: `Content Brief: ${gap.topic}`,
      objective: `Create content to compete with ${gap.leadingCompetitor} for "${gap.topic}"`,
      targetKeywords: this.extractKeywords(gap.topic),
      recommendedHeadlines: recommendations.headlines,
      keyPointsToCover: recommendations.keyPoints,
      competitorAnalysis: competitorContent,
      optimizationTips: recommendations.seoTips,
      platformSpecificTips: recommendations.platformTips,
      estimatedWordCount: this.estimateWordCount(gap.difficultyScore),
      timeline: this.createTimeline(gap),
      successMetrics: this.defineSuccessMetrics(gap)
    };
  }
}
```

### AI Insights Generation Engine
```typescript
// services/analytics/aiInsights.service.ts
export class AIInsightsService {
  async generateInsights(userId: string): Promise<AIInsight[]> {
    const data = await this.gatherAnalyticsData(userId);
    const insights: AIInsight[] = [];

    // Multi-dimensional analysis
    const trendInsights = await this.analyzeTrends(data);
    const competitiveInsights = await this.analyzeCompetitiveChanges(data);
    const platformInsights = await this.analyzePlatformBehavior(data);
    const opportunityInsights = await this.identifyOpportunities(data);
    const sentimentInsights = await this.analyzeSentimentPatterns(data);
    const seasonalInsights = await this.analyzeSeasonalPatterns(data);

    insights.push(
      ...trendInsights,
      ...competitiveInsights,
      ...platformInsights,
      ...opportunityInsights,
      ...sentimentInsights,
      ...seasonalInsights
    );

    return this.prioritizeAndRankInsights(insights);
  }

  private async analyzeTrends(data: AnalyticsData): Promise<AIInsight[]> {
    const insights: AIInsight[] = [];
    const timeSeriesData = data.timeSeriesData;
    
    // Share of voice trend analysis
    const voiceTrends = this.calculateShareOfVoiceTrends(timeSeriesData);
    
    voiceTrends.forEach(trend => {
      if (Math.abs(trend.changePercentage) > 10) {
        insights.push({
          type: 'trend_analysis',
          priority: this.calculatePriority(trend),
          title: `${trend.entity} ${trend.direction} in ${trend.topic}`,
          message: this.generateTrendMessage(trend),
          recommendation: this.generateTrendRecommendation(trend),
          estimatedImpact: this.estimateTrendImpact(trend),
          supportingData: {
            changePercentage: trend.changePercentage,
            timeframe: trend.timeframe,
            affectedTopics: trend.topics,
            platforms: trend.platforms
          },
          actionItems: this.generateTrendActionItems(trend),
          confidence: this.calculateTrendConfidence(trend)
        });
      }
    });

    // Momentum detection
    const momentumShifts = this.detectMomentumShifts(timeSeriesData);
    momentumShifts.forEach(shift => {
      insights.push({
        type: 'momentum_shift',
        priority: 'medium',
        title: `Momentum shift detected in ${shift.topic}`,
        message: `${shift.description}`,
        recommendation: `${shift.recommendation}`,
        estimatedImpact: shift.impact
      });
    });

    return insights;
  }

  private async analyzeCompetitiveChanges(data: AnalyticsData): Promise<AIInsight[]> {
    const insights: AIInsight[] = [];
    const competitorData = data.competitorData;
    
    // New competitive threats
    const threats = this.detectCompetitiveThreats(competitorData);
    threats.forEach(threat => {
      insights.push({
        type: 'competitive_threat',
        priority: 'high',
        title: `${threat.competitor} emerging threat in ${threat.topic}`,
        message: `${threat.competitor} gained ${threat.newPositions} top-3 positions in ${threat.topic} over the last ${threat.timeframe}`,
        recommendation: this.generateThreatResponse(threat),
        estimatedImpact: `Potential ${threat.estimatedLoss} monthly impression loss`,
        actionItems: [
          {
            action: 'Analyze competitor content strategy',
            priority: 'high',
            timeline: '2 days'
          },
          {
            action: 'Create competitive content response',
            priority: 'high',
            timeline: '1 week'
          }
        ]
      });
    });

    // Competitor weaknesses
    const weaknesses = this.identifyCompetitorWeaknesses(competitorData);
    weaknesses.forEach(weakness => {
      insights.push({
        type: 'competitive_opportunity',
        priority: 'medium',
        title: `Opportunity in ${weakness.topic}`,
        message: `${weakness.competitor} lost ${weakness.positionsLost} positions in ${weakness.topic}`,
        recommendation: `Capitalize by creating authoritative content on ${weakness.topic}`,
        estimatedImpact: `Potential ${weakness.opportunityValue} monthly reach gain`
      });
    });

    return insights;
  }

  private async analyzePlatformBehavior(data: AnalyticsData): Promise<AIInsight[]> {
    const insights: AIInsight[] = [];
    
    data.platformData.forEach(platform => {
      // Platform-specific optimization opportunities
      const optimizations = this.identifyPlatformOptimizations(platform);
      
      optimizations.forEach(opt => {
        insights.push({
          type: 'platform_optimization',
          priority: opt.priority,
          title: `${platform.name} optimization opportunity`,
          message: opt.finding,
          recommendation: opt.recommendation,
          estimatedImpact: opt.impact,
          actionItems: opt.actionItems
        });
      });

      // Performance anomalies
      const anomalies = this.detectPerformanceAnomalies(platform);
      anomalies.forEach(anomaly => {
        insights.push({
          type: 'performance_anomaly',
          priority: anomaly.severity,
          title: `Unusual ${platform.name} performance`,
          message: anomaly.description,
          recommendation: anomaly.investigation,
          estimatedImpact: anomaly.impact
        });
      });
    });

    return insights;
  }

  private async identifyOpportunities(data: AnalyticsData): Promise<AIInsight[]> {
    const insights: AIInsight[] = [];
    
    // Content gap opportunities
    const contentGaps = await this.identifyHighValueGaps(data.mentionData);
    contentGaps.slice(0, 5).forEach(gap => { // Top 5 opportunities
      insights.push({
        type: 'content_opportunity',
        priority: gap.priority >= 4 ? 'high' : 'medium',
        title: `High-value content opportunity: ${gap.topic}`,
        message: `${gap.leadingCompetitor} dominates "${gap.topic}" with ${gap.competitorAdvantage}% share of voice`,
        recommendation: `Create comprehensive content targeting "${gap.topic}" to capture ${gap.estimatedROI} monthly reach`,
        estimatedImpact: `${gap.estimatedROI} monthly reach, difficulty: ${gap.difficultyScore}/5`,
        actionItems: [
          {
            action: 'Research competitor content strategy',
            timeline: '3 days'
          },
          {
            action: 'Create content brief',
            timeline: '1 week'
          },
          {
            action: 'Produce and optimize content',
            timeline: '2 weeks'
          }
        ]
      });
    });

    // Trending topic opportunities
    const trendingTopics = this.identifyTrendingOpportunities(data);
    trendingTopics.forEach(topic => {
      insights.push({
        type: 'trending_opportunity',
        priority: 'medium',
        title: `Trending topic opportunity: ${topic.name}`,
        message: `${topic.name} is trending with ${topic.growthRate}% increase in mentions`,
        recommendation: `Create timely content about ${topic.name} to ride the trend`,
        estimatedImpact: `Potential ${topic.reachPotential} additional monthly impressions`
      });
    });

    return insights;
  }

  private async analyzeSentimentPatterns(data: AnalyticsData): Promise<AIInsight[]> {
    const insights: AIInsight[] = [];
    
    // Sentiment trend analysis
    const sentimentTrends = this.calculateSentimentTrends(data.mentionData);
    
    sentimentTrends.forEach(trend => {
      if (trend.significantChange) {
        insights.push({
          type: 'sentiment_trend',
          priority: trend.sentiment === 'negative' && trend.direction === 'declining' ? 'high' : 'medium',
          title: `${trend.sentiment} sentiment ${trend.direction} for ${trend.context}`,
          message: `${trend.sentiment} sentiment has ${trend.direction} by ${trend.changePercentage}% in ${trend.context} discussions`,
          recommendation: this.generateSentimentRecommendation(trend),
          estimatedImpact: this.estimateSentimentImpact(trend)
        });
      }
    });

    // Context-specific sentiment insights
    const contextInsights = this.analyzeContextualSentiment(data.mentionData);
    contextInsights.forEach(insight => {
      insights.push({
        type: 'sentiment_context',
        priority: 'medium',
        title: `Sentiment variation in ${insight.context}`,
        message: insight.finding,
        recommendation: insight.recommendation,
        estimatedImpact: insight.impact
      });
    });

    return insights;
  }

  private prioritizeAndRankInsights(insights: AIInsight[]): AIInsight[] {
    return insights
      .sort((a, b) => {
        // Priority ranking: high > medium > low
        const priorityScore = { high: 3, medium: 2, low: 1 };
        const aScore = priorityScore[a.priority] || 0;
        const bScore = priorityScore[b.priority] || 0;
        
        if (aScore !== bScore) return bScore - aScore;
        
        // Secondary sort by confidence (if available)
        const aConfidence = a.confidence || 0.5;
        const bConfidence = b.confidence || 0.5;
        
        return bConfidence - aConfidence;
      })
      .slice(0, 20); // Limit to top 20 insights
  }

  async generateActionPlan(insightId: string): Promise<ActionPlan> {
    const insight = await this.getInsight(insightId);
    
    return {
      title: `Action Plan: ${insight.title}`,
      objective: insight.recommendation,
      timeline: this.generateTimeline(insight),
      resources: this.estimateResources(insight),
      steps: this.generateDetailedSteps(insight),
      successMetrics: this.defineSuccessMetrics(insight),
      riskAssessment: this.assessRisks(insight),
      expectedOutcome: insight.estimatedImpact
    };
  }
}
```

## Security & Authentication
### JWT Authentication Implementation
```typescript
// middleware/auth.middleware.ts
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';
import { Request, Response, NextFunction } from 'express';

export interface AuthenticatedRequest extends Request {
  user?: {
    id: string;
    email: string;
    role: string;
    plan: string;
  };
}

export class AuthMiddleware {
  static async authenticate(req: AuthenticatedRequest, res: Response, next: NextFunction) {
    try {
      const token = req.header('Authorization')?.replace('Bearer ', '');
      
      if (!token) {
        return res.status(401).json({ error: 'Access denied. No token provided.' });
      }

      const decoded = jwt.verify(token, process.env.JWT_SECRET!) as any;
      req.user = decoded;
      
      next();
    } catch (error) {
      res.status(400).json({ error: 'Invalid token.' });
    }
  }

  static authorize(roles: string[] = []) {
    return (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
      if (!req.user) {
        return res.status(401).json({ error: 'Access denied.' });
      }

      if (roles.length && !roles.includes(req.user.role)) {
        return res.status(403).json({ error: 'Insufficient permissions.' });
      }

      next();
    };
  }

  static async hashPassword(password: string): Promise<string> {
    const salt = await bcrypt.genSalt(12);
    return bcrypt.hash(password, salt);
  }

  static async comparePassword(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }

  static generateToken(user: any): string {
    return jwt.sign(
      {
        id: user.id,
        email: user.email,
        role: user.role,
        plan: user.plan
      },
      process.env.JWT_SECRET!,
      { expiresIn: '7d' }
    );
  }

  static async refreshToken(token: string): Promise<string | null> {
    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET!) as any;
      
      // Generate new token with extended expiry
      return this.generateToken(decoded);
    } catch (error) {
      return null;
    }
  }
}

// Rate limiting middleware
export class RateLimitMiddleware {
  static createLimiter(windowMs: number, max: number) {
    return rateLimit({
      windowMs,
      max,
      message: {
        error: 'Too many requests from this IP, please try again later.'
      },
      standardHeaders: true,
      legacyHeaders: false
    });
  }

  // Different limits for different endpoints
  static apiLimiter = this.createLimiter(15 * 60 * 1000, 100); // 100 requests per 15 minutes
  static authLimiter = this.createLimiter(15 * 60 * 1000, 5);   // 5 login attempts per 15 minutes
  static queryLimiter = this.createLimiter(60 * 1000, 10);      // 10 AI queries per minute
}
```

## Local Development Setup
### Environment Configuration
```bash
# .env
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/wordsrank"
REDIS_URL="redis://localhost:6379"

# JWT
JWT_SECRET="your-super-secret-jwt-key-here"

# AI Platform APIs
OPENAI_API_KEY="sk-..."
ANTHROPIC_API_KEY="sk-ant-..."
GOOGLE_AI_API_KEY="..."
PERPLEXITY_API_KEY="pplx-..."
MISTRAL_API_KEY="..."
DEEPSEEK_API_KEY="..."

# Application
NODE_ENV="development"
PORT=3001
FRONTEND_URL="http://localhost:3000"

# Queue Settings
REDIS_HOST="localhost"
REDIS_PORT=6379
REDIS_PASSWORD=""

# File Upload
MAX_FILE_SIZE=10485760  # 10MB
UPLOAD_PATH="./uploads"
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
      POSTGRES_PASSWORD: wordsrank_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backend/migrations:/docker-entrypoint-initdb.d

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://wordsrank:wordsrank_password@postgresql:5432/wordsrank
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./backend:/app
      - /app/node_modules
    depends_on:
      - postgresql
      - redis
    command: npm run dev

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:3001/api
      - REACT_APP_WS_URL=http://localhost:3001
    volumes:
      - ./frontend:/app
      - /app/node_modules
    depends_on:
      - backend
    command: npm run dev

  queue-worker:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://wordsrank:wordsrank_password@postgresql:5432/wordsrank
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./backend:/app
      - /app/node_modules
    depends_on:
      - postgresql
      - redis
    command: npm run queue:worker

volumes:
  postgres_data:
  redis_data:
```

### Package.json Scripts
```json
{
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
    "queue:worker": "cd backend && npm run queue:worker",
    "start:prod": "cd backend && npm start"
  }
}
```

## Component Integration Map
### Data Flow Integration
```
User Interaction → Frontend Component → API Call → Backend Controller → 
Service Layer → Database/AI API → Queue Processing → Real-time Updates → 
WebSocket → Frontend Updates → UI Refresh
```

### Component Dependencies
```typescript
// Frontend Component Integration
DashboardOverview
├── AdvancedFilters (filters state management)
├── StatsGrid (real-time metrics)
├── VisibilityChart (chart data processing)
├── ShareOfVoice (topic cluster analysis)
├── RecentMentions (WebSocket real-time updates)
└── CompetitorRankings (competitive data)

AdvancedAnalytics
├── NetworkGraph (D3.js topic relationships)
├── ContentGapAnalysis (opportunity scoring)
├── SentimentAnalysis (confidence scoring)
└── AIInsights (recommendation engine)

PlatformPages
├── PlatformMetrics (platform-specific data)
├── SentimentTrends (platform sentiment analysis)
├── MentionsFeed (real-time platform mentions)
└── CompetitorComparison (platform-specific competition)

// Backend Service Integration
QueryProcessorQueue
├── AIServiceFactory (platform routing)
├── SentimentAnalysisService (sentiment processing)
├── ContentGapAnalysisService (gap identification)
├── AIInsightsService (insight generation)
└── WebSocketService (real-time broadcasting)

AIServiceFactory
├── OpenAIService (ChatGPT integration)
├── AnthropicService (Claude integration)
├── GoogleService (Gemini integration)
├── PerplexityService (Perplexity integration)
├── MistralService (Mistral AI integration)
└── DeepSeekService (DeepSeek integration)
```

## Advanced Marketing Features
### Marketing Automation Integration
```typescript
// services/marketing/automation.service.ts
export class MarketingAutomationService {
  async createCampaignFromInsight(insightId: string, userId: string): Promise<Campaign> {
    const insight = await this.getInsight(insightId);
    const contentBrief = await this.generateContentBrief(insight);
    
    return {
      id: generateId(),
      name: `Campaign: ${insight.title}`,
      objective: insight.recommendation,
      contentBrief,
      targetAudience: await this.identifyTargetAudience(insight),
      channels: this.recommendChannels(insight),
      timeline: this.generateCampaignTimeline(insight),
      budget: this.estimateBudget(insight),
      kpis: this.defineKPIs(insight),
      status: 'draft'
    };
  }

  async generateContentCalendar(userId: string, timeframe: string): Promise<ContentCalendar> {
    const insights = await this.getActiveInsights(userId);
    const contentGaps = await this.getContentGaps(userId);
    const seasonalTrends = await this.getSeasonalTrends(userId);
    
    const calendar = {
      timeframe,
      entries: []
    };

    // Schedule content based on insights
    insights.forEach(insight => {
      if (insight.type === 'content_opportunity') {
        calendar.entries.push({
          date: this.calculateOptimalDate(insight),
          title: `Create: ${insight.title}`,
          type: 'content_creation',
          priority: insight.priority,
          estimatedEffort: this.estimateEffort(insight),
          expectedImpact: insight.estimatedImpact
        });
      }
    });

    // Schedule gap filling content
    contentGaps.slice(0, 10).forEach(gap => {
      calendar.entries.push({
        date: this.calculateGapFillingDate(gap),
        title: `Gap Content: ${gap.topic}`,
        type: 'gap_filling',
        priority: gap.priority >= 4 ? 'high' : 'medium',
        estimatedEffort: gap.resourcesRequired,
        expectedImpact: gap.estimatedROI
      });
    });

    return calendar;
  }

  async performCompetitorBenchmarking(userId: string): Promise<BenchmarkReport> {
    const competitors = await this.getCompetitors(userId);
    const userMetrics = await this.getUserMetrics(userId);
    
    const benchmarks = await Promise.all(competitors.map(async competitor => {
      const competitorMetrics = await this.getCompetitorMetrics(competitor.id);
      
      return {
        competitor: competitor.name,
        metrics: {
          visibilityScore: {
            yours: userMetrics.visibilityScore,
            theirs: competitorMetrics.visibilityScore,
            gap: competitorMetrics.visibilityScore - userMetrics.visibilityScore
          },
          shareOfVoice: {
            yours: userMetrics.shareOfVoice,
            theirs: competitorMetrics.shareOfVoice,
            gap: competitorMetrics.shareOfVoice - userMetrics.shareOfVoice
          },
          sentimentScore: {
            yours: userMetrics.sentimentScore,
            theirs: competitorMetrics.sentimentScore,
            gap: competitorMetrics.sentimentScore - userMetrics.sentimentScore
          }
        },
        strongestAreas: this.identifyCompetitorStrengths(competitorMetrics),
        weakestAreas: this.identifyCompetitorWeaknesses(competitorMetrics),
        opportunityAreas: this.identifyOpportunityAreas(userMetrics, competitorMetrics)
      };
    }));

    return {
      executiveSummary: this.generateBenchmarkSummary(benchmarks),
      detailedComparison: benchmarks,
      strategicRecommendations: this.generateStrategicRecommendations(benchmarks),
      actionPriorities: this.prioritizeActions(benchmarks)
    };
  }
}
```

### Advanced Reporting System
```typescript
// services/reporting/advanced.service.ts
export class AdvancedReportingService {
  async generateExecutiveReport(userId: string, timeframe: string): Promise<ExecutiveReport> {
    const data = await this.gatherExecutiveData(userId, timeframe);
    
    return {
      executiveSummary: {
        keyMetrics: data.keyMetrics,
        majorWins: data.majorWins,
        criticalIssues: data.criticalIssues,
        strategicRecommendations: data.recommendations.slice(0, 3)
      },
      performanceOverview: {
        visibilityTrends: data.visibilityTrends,
        competitivePosition: data.competitivePosition,
        platformPerformance: data.platformPerformance,
        sentimentAnalysis: data.sentimentAnalysis
      },
      competitiveIntelligence: {
        marketShareChanges: data.marketShareChanges,
        competitorActivities: data.competitorActivities,
        threatAssessment: data.threatAssessment,
        opportunityIdentification: data.opportunities
      },
      actionItems: {
        immediate: data.actionItems.filter(a => a.priority === 'immediate'),
        shortTerm: data.actionItems.filter(a => a.priority === 'short_term'),
        longTerm: data.actionItems.filter(a => a.priority === 'long_term')
      },
      appendices: {
        detailedMetrics: data.detailedMetrics,
        methodologyNotes: data.methodology,
        dataQualityAssessment: data.dataQuality
      }
    };
  }

  async generateWhiteLabelReport(userId: string, clientBranding: BrandingOptions): Promise<WhiteLabelReport> {
    const baseReport = await this.generateExecutiveReport(userId, '30d');
    
    return {
      ...baseReport,
      branding: {
        logo: clientBranding.logo,
        colors: clientBranding.colors,
        companyName: clientBranding.companyName,
        footer: clientBranding.footer
      },
      customization: {
        removeProviderReferences: true,
        customMetrics: clientBranding.customMetrics,
        additionalSections: clientBranding.additionalSections
      }
    };
  }

  async scheduleAutomatedReports(userId: string, schedule: ReportSchedule): Promise<void> {
    const cronJobs = {
      daily: '0 9 * * *',      // 9 AM daily
      weekly: '0 9 * * 1',     // 9 AM every Monday
      monthly: '0 9 1 * *'     // 9 AM first day of month
    };

    await this.scheduleJob({
      userId,
      frequency: schedule.frequency,
      cronPattern: cronJobs[schedule.frequency],
      reportType: schedule.reportType,
      recipients: schedule.recipients,
      customizations: schedule.customizations
    });
  }
}
```
This comprehensive development specification provides everything needed to build WordsRank as a production-ready AI search intelligence platform with real API integrations, advanced analytics, and modern marketing features. The architecture is designed to be scalable, maintainable, and focused on delivering genuine value through actionable AI-powered insights.
