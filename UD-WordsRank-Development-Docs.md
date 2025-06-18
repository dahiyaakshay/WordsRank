# WordsRank - Local AI Platform Monitoring Tool

## Table of Contents
- Project Overview
- System Architecture
- Technology Stack
- Database Design
- Backend API Architecture
- Frontend Application Structure
- AI Platform Integrations
- User Interface Specifications
- Development Setup
- Error Handling & Offline Mode
- Backup & Data Export
- Testing Strategy
- Deployment & Setup

## Project Overview
### Business Purpose
WordsRank is a local AI platform monitoring tool designed for freelancers and consultants who need to track how their clients' brands and competitors appear in AI-powered search responses across Perplexity, Gemini, and Claude. It provides actionable insights for content strategy and competitive intelligence without the complexity of multi-user SaaS infrastructure.

### Core Value Proposition
- Client Brand Monitoring: Track client brand mentions across 3 major AI platforms with live search capabilities
- Competitive Intelligence: Understand how competitors rank in AI responses for strategic advantage
- Content Gap Analysis: Identify opportunities to improve client AI platform visibility
- Professional Reporting: Generate client-ready reports with insights and recommendations
- Cost-Effective: Local tool with no subscription fees - use your own API keys

### Target User
- Primary: Freelancers and consultants serving multiple clients
- Secondary: Marketing professionals managing multiple brands
- Use Case: Generate professional monitoring reports for client deliverables

### Key Features (Optimized for Freelancer Use)
1. Dashboard Analytics: Visual overview of brand performance across AI platforms
2. AI Platform Monitoring: Individual tracking for Perplexity, Gemini, and Claude
3. Search Terms Management: Monitor specific keywords and brand terms for multiple clients
4. Competitive Analysis: Track competitor performance and identify content gaps
5. Website & Brand Management: Configure multiple client brands and websites to monitor
6. Professional Reporting: Generate client-ready reports in multiple formats
7. Manual Data Refresh: On-demand queries to save API costs and tokens

## System Architecture
### High-Level Architecture
<pre>┌─────────────────────────────────────────────────────────────┐
│                    WordsRank Local Application              │
├─────────────────────────────────────────────────────────────┤
│  Frontend (React + TypeScript)                             │
│  ├── Dashboard & Analytics                                 │
│  ├── AI Platform Pages                                     │
│  ├── Management Pages                                      │
│  └── Settings                                              │
├─────────────────────────────────────────────────────────────┤
│  Backend API (Node.js + Express)                           │
│  ├── AI Platform Services                                  │
│  ├── Analytics & Reporting                                 │
│  ├── Data Processing                                       │
│  └── Export Services                                       │
├─────────────────────────────────────────────────────────────┤
│  Database Layer (SQLite)                                   │
│  ├── Website & Search Terms                                │
│  ├── Competitor Data                                       │
│  ├── AI Platform Responses                                 │
│  └── Analytics Data                                        │
├─────────────────────────────────────────────────────────────┤
│  Configuration                                             │
│  ├── config.json (API Keys)                               │
│  ├── settings.json (App Preferences)                      │
│  └── Local File Storage                                    │
├─────────────────────────────────────────────────────────────┤
│  AI Platform APIs                                          │
│  ├── Anthropic API (Claude)                                │
│  ├── Google AI API (Gemini)                                │
│  └── Perplexity API                                        │
└─────────────────────────────────────────────────────────────┘</pre>

### Data Flow
```
Manual Refresh → Search Terms → AI Platform APIs → Response Analysis → 
Position Detection → SQLite Storage → Analytics Processing → 
Dashboard Updates → Report Generation → Client Deliverables
```

## Technology Stack
### Backend Stack
```json
{
  "runtime": "Node.js 18+",
  "framework": "Express.js 4.18+",
  "database": "SQLite 3.40+",
  "validation": "Joi",
  "logging": "Simple console logging",
  "testing": "Jest",
  "http": "axios",
  "file_operations": "fs/promises"
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
  "anthropic": "@anthropic-ai/sdk",
  "google": "@google-ai/generativelanguage",
  "perplexity": "Custom HTTP client"
}
```

## Database Design
### SQLite Schema
#### Core Tables
```sql
-- Websites to monitor (clients)
CREATE TABLE websites (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    url TEXT NOT NULL,
    industry TEXT,
    description TEXT,
    is_active INTEGER DEFAULT 1,
    platforms_monitored TEXT DEFAULT '[]', -- JSON array ['perplexity', 'gemini', 'claude']
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Competitor websites
CREATE TABLE competitors (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    website_id INTEGER REFERENCES websites(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    url TEXT NOT NULL,
    is_active INTEGER DEFAULT 1,
    platforms_monitored TEXT DEFAULT '[]', -- JSON array
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Search terms to monitor
CREATE TABLE search_terms (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    website_id INTEGER REFERENCES websites(id) ON DELETE CASCADE,
    term TEXT NOT NULL,
    platforms_monitored TEXT DEFAULT '[]', -- JSON array of platforms to check
    is_active INTEGER DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- AI platform responses and mentions
CREATE TABLE ai_mentions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    search_term_id INTEGER REFERENCES search_terms(id) ON DELETE CASCADE,
    platform TEXT NOT NULL, -- 'perplexity', 'gemini', 'claude'
    query_text TEXT NOT NULL,
    response_text TEXT NOT NULL,
    position INTEGER, -- 1-10 ranking position, NULL if not mentioned
    sentiment TEXT DEFAULT 'neutral', -- 'positive', 'negative', 'neutral'
    confidence_score REAL DEFAULT 0.5,
    citations TEXT DEFAULT '[]', -- JSON array of URLs cited
    response_length INTEGER,
    processing_time INTEGER, -- milliseconds
    api_cost REAL DEFAULT 0,
    queried_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Analytics snapshots for performance tracking
CREATE TABLE analytics_snapshots (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    website_id INTEGER REFERENCES websites(id) ON DELETE CASCADE,
    snapshot_date DATE DEFAULT CURRENT_DATE,
    platform TEXT NOT NULL,
    total_terms_monitored INTEGER DEFAULT 0,
    terms_ranking_top3 INTEGER DEFAULT 0,
    terms_ranking_4to10 INTEGER DEFAULT 0,
    terms_not_mentioned INTEGER DEFAULT 0,
    average_position REAL,
    sentiment_positive INTEGER DEFAULT 0,
    sentiment_negative INTEGER DEFAULT 0,
    sentiment_neutral INTEGER DEFAULT 0,
    total_mentions INTEGER DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(website_id, snapshot_date, platform)
);
```

#### Indexes for Performance
```sql
-- Performance indexes
CREATE INDEX idx_ai_mentions_search_term_platform ON ai_mentions(search_term_id, platform);
CREATE INDEX idx_ai_mentions_queried_at ON ai_mentions(queried_at DESC);
CREATE INDEX idx_analytics_snapshots_website_date ON analytics_snapshots(website_id, snapshot_date DESC);
CREATE INDEX idx_search_terms_website_active ON search_terms(website_id, is_active);
```

#### Sample Configuration Files
```json
// config/api-keys.json
{
  "anthropic": "sk-ant-your-anthropic-key",
  "google": "your-google-ai-key",
  "perplexity": "pplx-your-perplexity-key"
}

// config/settings.json
{
  "app": {
    "name": "WordsRank",
    "version": "1.0.0",
    "data_retention_days": 365
  },
  "monitoring": {
    "default_platforms": ["perplexity", "gemini", "claude"],
    "request_delay_ms": 2000,
    "max_retries": 3
  },
  "reporting": {
    "default_format": "pdf",
    "include_charts": true,
    "include_competitor_analysis": true
  }
}
```

#### Configuration Security
```bash
# .gitignore
config/api-keys.json
data/wordsrank.sqlite
exports/*.pdf
exports/*.xlsx
exports/*.csv
node_modules/
.env*
```

#### Important Notes:
- Keep config/api-keys.example.json as a template in version control
- Never commit actual API keys to version control
- The config/api-keys.json file is created during installation from the example template
- All application settings are stored in config/settings.json and read/written by the application
- No sensitive configuration is stored in the database

## Backend API Architecture
### Project Structure
<pre>backend/
├── src/
│   ├── controllers/
│   │   ├── dashboard.controller.js
│   │   ├── websites.controller.js
│   │   ├── competitors.controller.js
│   │   ├── searchTerms.controller.js
│   │   ├── analytics.controller.js
│   │   ├── platforms.controller.js
│   │   └── reports.controller.js
│   ├── services/
│   │   ├── ai-platforms/
│   │   │   ├── base.service.js
│   │   │   ├── anthropic.service.js
│   │   │   ├── google.service.js
│   │   │   └── perplexity.service.js
│   │   ├── analytics.service.js
│   │   ├── monitoring.service.js
│   │   ├── export.service.js
│   │   └── config.service.js
│   ├── models/
│   │   ├── Website.js
│   │   ├── Competitor.js
│   │   ├── SearchTerm.js
│   │   └── AIMention.js
│   ├── middleware/
│   │   ├── validation.middleware.js
│   │   └── error.middleware.js
│   ├── routes/
│   │   ├── dashboard.routes.js
│   │   ├── websites.routes.js
│   │   ├── analytics.routes.js
│   │   ├── platforms.routes.js
│   │   └── reports.routes.js
│   ├── utils/
│   │   ├── database.js
│   │   ├── logger.js
│   │   ├── helpers.js
│   │   └── config.js
│   └── app.js
├── config/
│   ├── api-keys.example.json
│   ├── api-keys.json         # gitignored
│   ├── settings.example.json
│   └── settings.json
├── data/
│   └── wordsrank.sqlite
├── exports/
├── tests/
└── package.json</pre>

### Core API Endpoints
#### Website & Data Management
```javascript
// Websites
GET    /api/websites                 // Get all client websites
POST   /api/websites                 // Add new client website
PUT    /api/websites/:id             // Update website
DELETE /api/websites/:id             // Delete website

// Competitors
GET    /api/competitors/:websiteId   // Get competitors for website
POST   /api/competitors              // Add competitor
PUT    /api/competitors/:id          // Update competitor
DELETE /api/competitors/:id          // Delete competitor

// Search Terms
GET    /api/search-terms/:websiteId  // Get search terms for website
POST   /api/search-terms             // Add search term
PUT    /api/search-terms/:id         // Update search term
DELETE /api/search-terms/:id         // Delete search term
POST   /api/search-terms/bulk-import // Bulk import from CSV
```

#### AI Platform Management
```javascript
// Platform Queries (Manual Refresh)
POST   /api/platforms/query          // Query specific platform for term
POST   /api/platforms/query-batch    // Query multiple platforms/terms
GET    /api/platforms/status         // Check API key validity

// Platform Analytics
GET    /api/platforms/:platform/analytics  // Platform-specific analytics
GET    /api/platforms/:platform/mentions   // Recent mentions
```

#### Analytics & Reporting
```javascript
// Dashboard Analytics
GET    /api/analytics/dashboard/:websiteId     // Dashboard overview
GET    /api/analytics/overview/:websiteId      // Detailed overview
GET    /api/analytics/rankings/:websiteId      // Rankings analysis
GET    /api/analytics/competitors/:websiteId   // Competitive analysis
GET    /api/analytics/trends/:websiteId        // Historical trends

// Reports & Exports
POST   /api/reports/generate        // Generate client report
GET    /api/reports/export/:format  // Export data (PDF, CSV, Excel)
GET    /api/reports/templates       // Available report templates
```

#### Configuration
```javascript
// Settings (from JSON files)
GET    /api/config/settings         // Get app settings from config/settings.json
PUT    /api/config/settings         // Update settings.json file
GET    /api/config/api-keys         // Get API key status (masked)
PUT    /api/config/api-keys         // Update api-keys.json file
POST   /api/config/test-api-key     // Test specific API key
```

### Service Layer Implementation
#### Base AI Platform Service
```javascript
// services/ai-platforms/base.service.js
class BaseAIService {
  constructor(platform, apiKey) {
    this.platform = platform;
    this.apiKey = apiKey;
    this.requestDelay = 2000; // 2 second delay between requests
  }

  async queryPlatform(searchTerm, context = {}) {
    throw new Error('queryPlatform method must be implemented');
  }

  extractPosition(content, brandTerms) {
    // Extract position based on mention order in response
    const sentences = content.split(/[.!?]+/);
    
    for (let i = 0; i < sentences.length; i++) {
      const sentence = sentences[i].toLowerCase();
      if (brandTerms.some(term => sentence.includes(term.toLowerCase()))) {
        // Position scoring: 1st sentence = position 1, etc.
        if (i === 0) return 1;
        if (i <= 2) return 2;
        if (i <= 4) return 3;
        return Math.min(10, Math.ceil((i + 1) / 2));
      }
    }
    return null; // Not mentioned
  }

  extractCitations(content) {
    // Extract URLs from response
    const urlRegex = /https?:\/\/[^\s\)]+/g;
    const urls = content.match(urlRegex) || [];
    return urls.map(url => url.replace(/[.,;!?]$/, ''));
  }

  analyzeSentiment(content, brandTerms) {
    // Simple sentiment analysis around brand mentions
    const positiveWords = ['excellent', 'great', 'best', 'outstanding', 'superior', 'innovative', 'leading', 'top', 'premier', 'quality'];
    const negativeWords = ['poor', 'bad', 'worst', 'terrible', 'inferior', 'disappointing', 'failing', 'weak', 'subpar', 'inadequate'];
    
    // Extract sentences containing brand mentions
    const sentences = content.split(/[.!?]+/);
    const brandSentences = sentences.filter(sentence => 
      brandTerms.some(term => sentence.toLowerCase().includes(term.toLowerCase()))
    );
    
    if (brandSentences.length === 0) return 'neutral';
    
    let positiveScore = 0;
    let negativeScore = 0;
    
    brandSentences.forEach(sentence => {
      const words = sentence.toLowerCase().split(/\s+/);
      words.forEach(word => {
        if (positiveWords.includes(word)) positiveScore++;
        if (negativeWords.includes(word)) negativeScore++;
      });
    });
    
    if (positiveScore > negativeScore) return 'positive';
    if (negativeScore > positiveScore) return 'negative';
    return 'neutral';
  }

  calculateConfidence(content, position, citations) {
    let confidence = 0.5;
    
    // Higher confidence for longer, more detailed responses
    if (content.length > 200) confidence += 0.15;
    if (content.length > 500) confidence += 0.15;
    
    // Higher confidence for top positions
    if (position && position <= 3) confidence += 0.2;
    
    // Higher confidence if citations are included
    if (citations && citations.length > 0) confidence += 0.1;
    
    // Higher confidence for responses with specific details
    if (content.includes('according to') || content.includes('based on')) confidence += 0.1;
    
    return Math.min(0.95, confidence);
  }

  async delay() {
    return new Promise(resolve => setTimeout(resolve, this.requestDelay));
  }
}

module.exports = BaseAIService;
```

#### Anthropic Service Implementation
```javascript
// services/ai-platforms/anthropic.service.js
const Anthropic = require('@anthropic-ai/sdk');
const BaseAIService = require('./base.service');

class AnthropicService extends BaseAIService {
  constructor(apiKey) {
    super('claude', apiKey);
    this.client = new Anthropic({ apiKey });
    this.model = 'claude-3-sonnet-20240229';
  }

  async queryPlatform(searchTerm, context = {}) {
    const startTime = Date.now();
    
    try {
      const response = await this.client.messages.create({
        model: this.model,
        max_tokens: 1000,
        messages: [
          {
            role: 'user',
            content: `Please provide comprehensive information about: ${searchTerm}. Include specific brands, companies, and products when relevant. Provide detailed, well-researched responses with multiple perspectives and current information.`
          }
        ]
      });

      const content = response.content[0]?.text || '';
      const brandTerms = context.brandTerms || [];
      const citations = this.extractCitations(content);
      const position = this.extractPosition(content, brandTerms);
      
      return {
        platform: this.platform,
        query: searchTerm,
        response: content,
        position: position,
        sentiment: this.analyzeSentiment(content, brandTerms),
        confidence: this.calculateConfidence(content, position, citations),
        citations: citations,
        responseLength: content.length,
        processingTime: Date.now() - startTime,
        tokensUsed: response.usage?.output_tokens || 0,
        apiCost: this.calculateCost(response.usage?.input_tokens || 0, response.usage?.output_tokens || 0)
      };
    } catch (error) {
      throw new Error(`Anthropic API Error: ${error.message}`);
    }
  }

  calculateCost(inputTokens, outputTokens) {
    // Claude-3 Sonnet pricing: $3 per million input tokens, $15 per million output tokens
    const inputCost = (inputTokens / 1000000) * 3;
    const outputCost = (outputTokens / 1000000) * 15;
    return inputCost + outputCost;
  }
}

module.exports = AnthropicService;
```

#### Perplexity Service Implementation
```javascript
// services/ai-platforms/perplexity.service.js
const axios = require('axios');
const BaseAIService = require('./base.service');

class PerplexityService extends BaseAIService {
  constructor(apiKey) {
    super('perplexity', apiKey);
    this.client = axios.create({
      baseURL: 'https://api.perplexity.ai',
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json'
      }
    });
    this.model = 'llama-3.1-sonar-large-128k-online';
  }

  async queryPlatform(searchTerm, context = {}) {
    const startTime = Date.now();
    
    try {
      const response = await this.client.post('/chat/completions', {
        model: this.model,
        messages: [
          {
            role: 'system',
            content: 'You are a helpful assistant that provides comprehensive, up-to-date information with citations. Always include relevant brands, companies, and products in your responses.'
          },
          {
            role: 'user',
            content: searchTerm
          }
        ],
        max_tokens: 1000,
        temperature: 0.7,
        search_domain_filter: ["perplexity.ai"],
        return_citations: true,
        search_recency_filter: "month"
      });

      const content = response.data.choices[0]?.message?.content || '';
      const brandTerms = context.brandTerms || [];
      const citations = this.extractCitations(content);
      const position = this.extractPosition(content, brandTerms);
      
      return {
        platform: this.platform,
        query: searchTerm,
        response: content,
        position: position,
        sentiment: this.analyzeSentiment(content, brandTerms),
        confidence: this.calculateConfidence(content, position, citations),
        citations: citations,
        responseLength: content.length,
        processingTime: Date.now() - startTime,
        tokensUsed: response.data.usage?.total_tokens || 0,
        apiCost: this.calculateCost(response.data.usage?.total_tokens || 0)
      };
    } catch (error) {
      throw new Error(`Perplexity API Error: ${error.message}`);
    }
  }

  calculateCost(tokens) {
    // Perplexity pricing: ~$1 per million tokens for sonar models
    return (tokens / 1000000) * 1;
  }
}

module.exports = PerplexityService;
```

#### Google Service Implementation
```javascript
// services/ai-platforms/google.service.js
const { GenerativeServiceClient } = require('@google-ai/generativelanguage');
const BaseAIService = require('./base.service');

class GoogleAIService extends BaseAIService {
  constructor(apiKey) {
    super('gemini', apiKey);
    this.client = new GenerativeServiceClient({
      authClient: {
        getAccessToken: () => Promise.resolve({ access_token: apiKey })
      }
    });
    this.model = 'models/gemini-1.5-pro-latest';
  }

  async queryPlatform(searchTerm, context = {}) {
    const startTime = Date.now();
    
    try {
      const request = {
        model: this.model,
        contents: [{
          parts: [{
            text: `Please provide comprehensive information about: ${searchTerm}. Include specific brands, companies, and products when relevant. Provide detailed, well-researched responses with multiple perspectives and current information where possible.`
          }]
        }],
        generationConfig: {
          maxOutputTokens: 1000,
          temperature: 0.7,
        }
      };

      const response = await this.client.generateContent(request);
      const content = response[0].candidates[0]?.content?.parts[0]?.text || '';
      const brandTerms = context.brandTerms || [];
      const citations = this.extractCitations(content);
      const position = this.extractPosition(content, brandTerms);
      
      return {
        platform: this.platform,
        query: searchTerm,
        response: content,
        position: position,
        sentiment: this.analyzeSentiment(content, brandTerms),
        confidence: this.calculateConfidence(content, position, citations),
        citations: citations,
        responseLength: content.length,
        processingTime: Date.now() - startTime,
        tokensUsed: response[0].usageMetadata?.totalTokenCount || 0,
        apiCost: this.calculateCost(response[0].usageMetadata?.totalTokenCount || 0)
      };
    } catch (error) {
      throw new Error(`Google AI API Error: ${error.message}`);
    }
  }

  calculateCost(tokens) {
    // Gemini Pro pricing: ~$0.5 per million tokens
    return (tokens / 1000000) * 0.5;
  }
}

module.exports = GoogleAIService;
```

#### AI Service Factory
```javascript
// services/ai-platforms/index.js
const AnthropicService = require('./anthropic.service');
const GoogleAIService = require('./google.service');
const PerplexityService = require('./perplexity.service');

class AIServiceFactory {
  getService(platform, apiKey) {
    if (!apiKey) {
      throw new Error(`API key not configured for ${platform}`);
    }

    switch (platform.toLowerCase()) {
      case 'claude':
        return new AnthropicService(apiKey);
      case 'gemini':
        return new GoogleAIService(apiKey);
      case 'perplexity':
        return new PerplexityService(apiKey);
      default:
        throw new Error(`Unsupported platform: ${platform}`);
    }
  }

  getSupportedPlatforms() {
    return ['claude', 'gemini', 'perplexity'];
  }
}

module.exports = { AIServiceFactory };
```

#### Monitoring Service
```javascript
// services/monitoring.service.js
const { AIServiceFactory } = require('./ai-platforms');
const { SearchTerm, Website, AIMention } = require('../models');
const ConfigService = require('./config.service');

class MonitoringService {
  constructor() {
    this.aiServiceFactory = new AIServiceFactory();
  }

  async queryTermsForWebsite(websiteId, platformsToQuery = null) {
    try {
      const website = await Website.findById(websiteId);
      if (!website) throw new Error('Website not found');

      const searchTerms = await SearchTerm.findByWebsiteId(websiteId);
      const brandTerms = [website.name, ...website.name.split(' ')];
      
      const platforms = platformsToQuery || website.platforms_monitored || ['perplexity', 'gemini', 'claude'];
      
      const results = [];
      
      for (const searchTerm of searchTerms) {
        for (const platform of platforms) {
          const result = await this.queryPlatform(searchTerm, platform, brandTerms);
          if (result) results.push(result);
          
          // Rate limiting delay between requests
          await this.delay(2000);
        }
      }
      
      return results;
    } catch (error) {
      console.error(`Monitoring error for website ${websiteId}:`, error);
      throw error;
    }
  }

  async queryPlatform(searchTerm, platform, brandTerms) {
    try {
      const ConfigService = require('./config.service');
      const apiKeys = await ConfigService.getApiKeys(); // Reads from config/api-keys.json
      const aiService = this.aiServiceFactory.getService(platform, apiKeys[platform]);
      
      if (!aiService) {
        throw new Error(`No API key configured for ${platform}`);
      }

      const result = await aiService.queryPlatform(searchTerm.term, { brandTerms });
      
      // Store result in database
      const mentionId = await AIMention.create({
        search_term_id: searchTerm.id,
        platform: result.platform,
        query_text: result.query,
        response_text: result.response,
        position: result.position,
        sentiment: result.sentiment,
        confidence_score: result.confidence,
        citations: JSON.stringify(result.citations),
        response_length: result.responseLength,
        processing_time: result.processingTime,
        api_cost: result.apiCost
      });

      return { ...result, id: mentionId };
    } catch (error) {
      console.error(`Platform query error for ${platform}:`, error);
      throw error;
    }
  }

  async batchQuery(queries) {
    // Process multiple platform/term combinations
    const results = [];
    
    for (const query of queries) {
      try {
        const result = await this.queryPlatform(query.searchTerm, query.platform, query.brandTerms);
        results.push(result);
        
        // Rate limiting delay
        await this.delay(2000);
      } catch (error) {
        results.push({
          error: error.message,
          searchTerm: query.searchTerm,
          platform: query.platform
        });
      }
    }
    
    return results;
  }

  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

module.exports = MonitoringService;
```

#### Configuration Service
```javascript
// services/config.service.js
const fs = require('fs').promises;
const path = require('path');

class ConfigService {
  constructor() {
    this.configDir = path.join(__dirname, '../../config');
    this.apiKeysPath = path.join(this.configDir, 'api-keys.json');
    this.settingsPath = path.join(this.configDir, 'settings.json');
  }

  async getApiKeys() {
    try {
      const data = await fs.readFile(this.apiKeysPath, 'utf8');
      return JSON.parse(data);
    } catch (error) {
      console.error('Error reading API keys:', error);
      return {};
    }
  }

  async setApiKey(platform, apiKey) {
    const apiKeys = await this.getApiKeys();
    apiKeys[platform] = apiKey;
    
    await fs.writeFile(this.apiKeysPath, JSON.stringify(apiKeys, null, 2));
    return true;
  }

  async getSettings() {
    try {
      const data = await fs.readFile(this.settingsPath, 'utf8');
      return JSON.parse(data);
    } catch (error) {
      console.error('Error reading settings:', error);
      return this.getDefaultSettings();
    }
  }

  async updateSettings(newSettings) {
    const currentSettings = await this.getSettings();
    const mergedSettings = { ...currentSettings, ...newSettings };
    
    await fs.writeFile(this.settingsPath, JSON.stringify(mergedSettings, null, 2));
    return mergedSettings;
  }

  getDefaultSettings() {
    return {
      app: {
        name: "WordsRank",
        version: "1.0.0",
        data_retention_days: 365
      },
      monitoring: {
        default_platforms: ["perplexity", "gemini", "claude"],
        request_delay_ms: 2000,
        max_retries: 3
      },
      reporting: {
        default_format: "pdf",
        include_charts: true,
        include_competitor_analysis: true
      }
    };
  }

  async testApiKey(platform, apiKey) {
    // Test API key validity by making a simple request
    const { AIServiceFactory } = require('./ai-platforms');
    const factory = new AIServiceFactory();
    
    try {
      const service = factory.getService(platform, apiKey);
      // Make a minimal test query
      await service.queryPlatform('test', { brandTerms: [] });
      return { valid: true, error: null };
    } catch (error) {
      return { valid: false, error: error.message };
    }
  }
}

module.exports = ConfigService;
```

#### Analytics Service
```javascript
// services/analytics.service.js
const { AIMention, AnalyticsSnapshot, Website, Competitor } = require('../models');

class AnalyticsService {
  async getDashboardAnalytics(websiteId, timeframe = '30d') {
    const timeframeDays = this.parseTimeframe(timeframe);
    const startDate = new Date();
    startDate.setDate(startDate.getDate() - timeframeDays);

    const mentions = await AIMention.findByWebsiteAndTimeframe(websiteId, startDate);
    const website = await Website.findById(websiteId);
    const competitors = await Competitor.findByWebsiteId(websiteId);
    
    return {
      overview: await this.calculateOverviewStats(mentions),
      platformPerformance: await this.calculatePlatformPerformance(mentions),
      rankingTrends: await this.calculateRankingTrends(websiteId, timeframeDays),
      sentimentAnalysis: await this.calculateSentimentAnalysis(mentions),
      competitiveAnalysis: await this.calculateCompetitiveAnalysis(websiteId, competitors, startDate),
      contentGaps: await this.identifyContentGaps(mentions, competitors)
    };
  }

  async calculateOverviewStats(mentions) {
    const totalMentions = mentions.length;
    const mentionsWithPosition = mentions.filter(m => m.position !== null);
    const top3Mentions = mentions.filter(m => m.position && m.position <= 3);
    const top5Mentions = mentions.filter(m => m.position && m.position <= 5);
    
    const averagePosition = mentionsWithPosition.length > 0 
      ? mentionsWithPosition.reduce((sum, m) => sum + m.position, 0) / mentionsWithPosition.length 
      : null;

    const positiveSentiment = mentions.filter(m => m.sentiment === 'positive').length;
    const negativeSentiment = mentions.filter(m => m.sentiment === 'negative').length;

    return {
      totalMentions,
      visibilityScore: totalMentions > 0 ? (mentionsWithPosition.length / totalMentions) * 100 : 0,
      averagePosition: averagePosition ? Math.round(averagePosition * 10) / 10 : null,
      top3Rankings: top3Mentions.length,
      top5Rankings: top5Mentions.length,
      platformCoverage: [...new Set(mentionsWithPosition.map(m => m.platform))].length,
      sentimentScore: totalMentions > 0 ? (positiveSentiment / totalMentions) * 100 : 0,
      totalApiCost: mentions.reduce((sum, m) => sum + (m.api_cost || 0), 0)
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
          top5Count: 0,
          averagePosition: 0,
          sentimentPositive: 0,
          sentimentNegative: 0,
          sentimentNeutral: 0,
          totalCost: 0,
          avgConfidence: 0
        };
      }
      
      const stats = platformStats[mention.platform];
      stats.totalMentions++;
      stats.totalCost += mention.api_cost || 0;
      stats.avgConfidence += mention.confidence_score || 0;
      
      if (mention.position) {
        stats.positionedMentions++;
        stats.averagePosition += mention.position;
        if (mention.position <= 3) stats.top3Count++;
        if (mention.position <= 5) stats.top5Count++;
      }
      
      if (mention.sentiment === 'positive') stats.sentimentPositive++;
      if (mention.sentiment === 'negative') stats.sentimentNegative++;
      if (mention.sentiment === 'neutral') stats.sentimentNeutral++;
    });

    // Calculate averages for each platform
    Object.keys(platformStats).forEach(platform => {
      const stats = platformStats[platform];
      if (stats.positionedMentions > 0) {
        stats.averagePosition = stats.averagePosition / stats.positionedMentions;
      } else {
        stats.averagePosition = null;
      }
      
      if (stats.totalMentions > 0) {
        stats.avgConfidence = stats.avgConfidence / stats.totalMentions;
      }
      
      stats.visibilityScore = stats.totalMentions > 0 
        ? (stats.positionedMentions / stats.totalMentions) * 100 
        : 0;
    });

    return platformStats;
  }

  async calculateRankingTrends(websiteId, timeframeDays) {
    // Calculate ranking trends over time across Perplexity, Gemini, and Claude
    const trends = {};
    const platforms = ['perplexity', 'gemini', 'claude'];
    
    for (const platform of platforms) {
      const snapshots = await AnalyticsSnapshot.findByWebsitePlatformAndDays(
        websiteId, 
        platform, 
        timeframeDays
      );
      
      trends[platform] = {
        visibilityTrend: this.calculateTrendFromSnapshots(snapshots, 'average_position'),
        positionTrend: this.calculateTrendFromSnapshots(snapshots, 'average_position'),
        mentionsTrend: this.calculateTrendFromSnapshots(snapshots, 'total_mentions')
      };
    }
    
    return trends;
  }

  async calculateSentimentAnalysis(mentions) {
    const sentimentByPlatform = {};
    const platforms = ['perplexity', 'gemini', 'claude'];
    
    platforms.forEach(platform => {
      const platformMentions = mentions.filter(m => m.platform === platform);
      const total = platformMentions.length;
      
      if (total > 0) {
        sentimentByPlatform[platform] = {
          positive: platformMentions.filter(m => m.sentiment === 'positive').length,
          negative: platformMentions.filter(m => m.sentiment === 'negative').length,
          neutral: platformMentions.filter(m => m.sentiment === 'neutral').length,
          positivePercentage: (platformMentions.filter(m => m.sentiment === 'positive').length / total) * 100,
          negativePercentage: (platformMentions.filter(m => m.sentiment === 'negative').length / total) * 100,
          neutralPercentage: (platformMentions.filter(m => m.sentiment === 'neutral').length / total) * 100
        };
      } else {
        sentimentByPlatform[platform] = {
          positive: 0, negative: 0, neutral: 0,
          positivePercentage: 0, negativePercentage: 0, neutralPercentage: 0
        };
      }
    });
    
    return sentimentByPlatform;
  }

  async calculateCompetitiveAnalysis(websiteId, competitors, startDate) {
    const competitorAnalysis = {};
    
    for (const competitor of competitors) {
      const competitorMentions = await AIMention.findByCompetitorAndTimeframe(competitor.id, startDate);
      
      competitorAnalysis[competitor.name] = {
        totalMentions: competitorMentions.length,
        averagePosition: this.calculateAveragePosition(competitorMentions),
        visibilityScore: this.calculateVisibilityScore(competitorMentions),
        top3Count: competitorMentions.filter(m => m.position && m.position <= 3).length,
        platformCoverage: [...new Set(competitorMentions.map(m => m.platform))].length,
        platformPerformance: this.calculateCompetitorPlatformBreakdown(competitorMentions)
      };
    }
    
    return competitorAnalysis;
  }

  calculateCompetitorPlatformBreakdown(mentions) {
    const platforms = ['perplexity', 'gemini', 'claude'];
    const breakdown = {};
    
    platforms.forEach(platform => {
      const platformMentions = mentions.filter(m => m.platform === platform);
      breakdown[platform] = {
        mentions: platformMentions.length,
        averagePosition: this.calculateAveragePosition(platformMentions),
        top3Count: platformMentions.filter(m => m.position && m.position <= 3).length
      };
    });
    
    return breakdown;
  }

  async identifyContentGaps(mentions, competitors) {
    // Analyze where competitors rank but client doesn't across Perplexity, Gemini, and Claude
    const gaps = [];
    
    // Group client mentions by search term and platform
    const clientMentionMap = new Map();
    mentions.forEach(mention => {
      const key = `${mention.search_term_id}-${mention.platform}`;
      clientMentionMap.set(key, mention);
    });
    
    // Check competitor performance vs client for same terms
    for (const competitor of competitors) {
      const competitorMentions = await AIMention.findByCompetitor(competitor.id);
      
      competitorMentions.forEach(competitorMention => {
        const key = `${competitorMention.search_term_id}-${competitorMention.platform}`;
        const clientMention = clientMentionMap.get(key);
        
        // Gap identified: competitor ranks but client doesn't, or competitor ranks higher
        if (!clientMention && competitorMention.position && competitorMention.position <= 5) {
          gaps.push({
            searchTermId: competitorMention.search_term_id,
            platform: competitorMention.platform,
            competitor: competitor.name,
            competitorPosition: competitorMention.position,
            clientPosition: null,
            gapType: 'not_mentioned',
            opportunity: 'high'
          });
        } else if (clientMention && competitorMention.position && 
                   clientMention.position && competitorMention.position < clientMention.position) {
          gaps.push({
            searchTermId: competitorMention.search_term_id,
            platform: competitorMention.platform,
            competitor: competitor.name,
            competitorPosition: competitorMention.position,
            clientPosition: clientMention.position,
            gapType: 'outranked',
            opportunity: competitorMention.position <= 3 ? 'high' : 'medium'
          });
        }
      });
    }
    
    return gaps;
  }

  calculateAveragePosition(mentions) {
    const positioned = mentions.filter(m => m.position !== null);
    return positioned.length > 0 
      ? positioned.reduce((sum, m) => sum + m.position, 0) / positioned.length 
      : null;
  }

  calculateVisibilityScore(mentions) {
    const total = mentions.length;
    const positioned = mentions.filter(m => m.position !== null).length;
    return total > 0 ? (positioned / total) * 100 : 0;
  }

  calculateTrendFromSnapshots(snapshots, metric) {
    if (snapshots.length < 2) return 0;
    
    const sorted = snapshots.sort((a, b) => new Date(a.snapshot_date) - new Date(b.snapshot_date));
    const oldest = sorted[0][metric] || 0;
    const newest = sorted[sorted.length - 1][metric] || 0;
    
    return oldest > 0 ? ((newest - oldest) / oldest) * 100 : 0;
  }

  parseTimeframe(timeframe) {
    switch (timeframe) {
      case '7d': return 7;
      case '30d': return 30;
      case '90d': return 90;
      case '180d': return 180;
      case '1y': return 365;
      default: return 30;
    }
  }
}

module.exports = AnalyticsService;
```

#### Export Service
```javascript
// services/export.service.js
const PDFDocument = require('pdfkit');
const ExcelJS = require('exceljs');
const fs = require('fs').promises;
const path = require('path');

class ExportService {
  constructor() {
    this.exportDir = path.join(__dirname, '../../exports');
    this.supportedPlatforms = ['perplexity', 'gemini', 'claude'];
  }

  async generateClientReport(websiteId, options = {}) {
    const {
      format = 'pdf',
      timeframe = '30d',
      includeCompetitors = true,
      includeCharts = true,
      includePlatformBreakdown = true,
      clientBranding = {}
    } = options;

    const analytics = await this.getReportData(websiteId, timeframe);
    const website = await this.getWebsiteInfo(websiteId);
    
    switch (format) {
      case 'pdf':
        return this.generatePDFReport(analytics, website, options);
      case 'excel':
        return this.generateExcelReport(analytics, website, options);
      case 'csv':
        return this.generateCSVReport(analytics, website, options);
      default:
        throw new Error(`Unsupported export format: ${format}`);
    }
  }

  async generatePDFReport(analytics, website, options) {
    const doc = new PDFDocument({ margin: 50 });
    const filename = `${website.name}-report-${Date.now()}.pdf`;
    const filepath = path.join(this.exportDir, filename);
    
    doc.pipe(require('fs').createWriteStream(filepath));
    
    // Report Header
    doc.fontSize(24).text('AI Platform Monitoring Report', 50, 50);
    doc.fontSize(16).text(`Client: ${website.name}`, 50, 85);
    doc.fontSize(12).text(`Generated: ${new Date().toLocaleDateString()}`, 50, 110);
    doc.text(`Monitoring Period: ${options.timeframe || '30 days'}`, 50, 125);
    
    // Executive Summary
    doc.fontSize(18).text('Executive Summary', 50, 160);
    
    const overview = analytics.overview;
    doc.fontSize(12)
       .text(`• Total Brand Mentions: ${overview.totalMentions}`, 70, 190)
       .text(`• Brand Visibility Score: ${overview.visibilityScore.toFixed(1)}%`, 70, 210)
       .text(`• Average Position: ${overview.averagePosition ? `#${overview.averagePosition}` : 'Not ranked'}`, 70, 230)
       .text(`• Top 3 Rankings: ${overview.top3Rankings} (${overview.totalMentions > 0 ? (overview.top3Rankings / overview.totalMentions * 100).toFixed(1) : 0}%)`, 70, 250);
    
    // Platform Performance Section
    let yPosition = 290;
    doc.fontSize(18).text('Platform Performance Analysis', 50, yPosition);
    yPosition += 35;
    
    // Performance across Perplexity, Gemini, and Claude
    Object.entries(analytics.platformPerformance).forEach(([platform, stats]) => {
      const platformName = this.getPlatformDisplayName(platform);
      
      doc.fontSize(14).text(`${platformName}`, 70, yPosition);
      yPosition += 20;
      
      doc.fontSize(11)
         .text(`Mentions: ${stats.totalMentions}`, 90, yPosition)
         .text(`Visibility: ${stats.visibilityScore.toFixed(1)}%`, 200, yPosition)
         .text(`Avg Position: ${stats.averagePosition ? `#${stats.averagePosition.toFixed(1)}` : 'N/A'}`, 300, yPosition)
         .text(`Top 3: ${stats.top3Count}`, 420, yPosition);
      
      yPosition += 15;
      
      // Sentiment breakdown
      doc.fontSize(10)
         .fillColor('green').text(`Positive: ${stats.sentimentPositive}`, 90, yPosition)
         .fillColor('red').text(`Negative: ${stats.sentimentNegative}`, 150, yPosition)
         .fillColor('gray').text(`Neutral: ${stats.sentimentNeutral}`, 210, yPosition)
         .fillColor('black').text(`Cost: $${stats.totalCost.toFixed(2)}`, 270, yPosition);
      
      yPosition += 25;
      
      // Add page break if needed
      if (yPosition > 700) {
        doc.addPage();
        yPosition = 50;
      }
    });
    
    // Competitive Analysis (if included)
    if (options.includeCompetitors && analytics.competitiveAnalysis) {
      doc.addPage();
      doc.fontSize(18).text('Competitive Analysis', 50, 50);
      
      yPosition = 85;
      Object.entries(analytics.competitiveAnalysis).forEach(([competitor, data]) => {
        doc.fontSize(14).text(`${competitor}`, 70, yPosition);
        yPosition += 20;
        
        doc.fontSize(11)
           .text(`Mentions: ${data.totalMentions}`, 90, yPosition)
           .text(`Visibility: ${data.visibilityScore.toFixed(1)}%`, 200, yPosition)
           .text(`Avg Position: ${data.averagePosition ? `#${data.averagePosition.toFixed(1)}` : 'N/A'}`, 300, yPosition)
           .text(`Top 3: ${data.top3Count}`, 420, yPosition);
        
        yPosition += 30;
      });
    }
    
    // Strategic Recommendations
    doc.addPage();
    doc.fontSize(18).text('Strategic Recommendations', 50, 50);
    
    const recommendations = this.generateRecommendations(analytics);
    yPosition = 85;
    
    recommendations.forEach((rec, index) => {
      doc.fontSize(12).text(`${index + 1}. ${rec}`, 70, yPosition);
      yPosition += 25;
    });
    
    // Footer
    doc.fontSize(10)
       .fillColor('gray')
       .text('Generated by WordsRank - AI Platform Monitoring Tool', 50, doc.page.height - 50);
    
    doc.end();
    
    return {
      filepath,
      filename,
      format: 'pdf'
    };
  }

  async generateExcelReport(analytics, website, options) {
    const workbook = new ExcelJS.Workbook();
    
    // Metadata
    workbook.creator = 'WordsRank';
    workbook.created = new Date();
    
    // Executive Summary Sheet
    const summarySheet = workbook.addWorksheet('Executive Summary');
    summarySheet.addRow(['AI Platform Monitoring Report']);
    summarySheet.addRow([`Client: ${website.name}`]);
    summarySheet.addRow([`Generated: ${new Date().toLocaleDateString()}`]);
    summarySheet.addRow([]);
    summarySheet.addRow(['Metric', 'Value']);
    summarySheet.addRow(['Total Mentions', analytics.overview.totalMentions]);
    summarySheet.addRow(['Visibility Score', `${analytics.overview.visibilityScore.toFixed(1)}%`]);
    summarySheet.addRow(['Average Position', analytics.overview.averagePosition || 'Not ranked']);
    summarySheet.addRow(['Top 3 Rankings', analytics.overview.top3Rankings]);
    summarySheet.addRow(['Platform Coverage', `${analytics.overview.platformCoverage}/3 platforms`]);
    summarySheet.addRow(['Sentiment Score', `${analytics.overview.sentimentScore.toFixed(1)}%`]);
    summarySheet.addRow(['Total API Cost', `$${analytics.overview.totalApiCost.toFixed(2)}`]);
    
    // Platform Performance Sheet
    const platformSheet = workbook.addWorksheet('Platform Performance');
    platformSheet.addRow([
      'Platform', 'Total Mentions', 'Visibility Score', 'Average Position', 
      'Top 3 Count', 'Top 5 Count', 'Positive Sentiment', 'Negative Sentiment', 
      'Neutral Sentiment', 'Total Cost', 'Avg Confidence'
    ]);
    
    Object.entries(analytics.platformPerformance).forEach(([platform, stats]) => {
      platformSheet.addRow([
        this.getPlatformDisplayName(platform),
        stats.totalMentions,
        `${stats.visibilityScore.toFixed(1)}%`,
        stats.averagePosition ? stats.averagePosition.toFixed(1) : 'N/A',
        stats.top3Count,
        stats.top5Count,
        stats.sentimentPositive,
        stats.sentimentNegative,
        stats.sentimentNeutral,
        `$${stats.totalCost.toFixed(2)}`,
        stats.avgConfidence.toFixed(2)
      ]);
    });
    
    // Competitive Analysis Sheet (if included)
    if (options.includeCompetitors && analytics.competitiveAnalysis) {
      const competitorSheet = workbook.addWorksheet('Competitive Analysis');
      competitorSheet.addRow([
        'Competitor', 'Total Mentions', 'Visibility Score', 'Average Position', 
        'Top 3 Count', 'Platform Coverage'
      ]);
      
      Object.entries(analytics.competitiveAnalysis).forEach(([competitor, data]) => {
        competitorSheet.addRow([
          competitor,
          data.totalMentions,
          `${data.visibilityScore.toFixed(1)}%`,
          data.averagePosition ? data.averagePosition.toFixed(1) : 'N/A',
          data.top3Count,
          data.platformCoverage
        ]);
      });
    }
    
    // Sentiment Analysis Sheet
    const sentimentSheet = workbook.addWorksheet('Sentiment Analysis');
    sentimentSheet.addRow(['Platform', 'Positive', 'Negative', 'Neutral', 'Positive %', 'Negative %', 'Neutral %']);
    
    Object.entries(analytics.sentimentAnalysis).forEach(([platform, sentiment]) => {
      sentimentSheet.addRow([
        this.getPlatformDisplayName(platform),
        sentiment.positive,
        sentiment.negative,
        sentiment.neutral,
        `${sentiment.positivePercentage.toFixed(1)}%`,
        `${sentiment.negativePercentage.toFixed(1)}%`,
        `${sentiment.neutralPercentage.toFixed(1)}%`
      ]);
    });
    
    // Apply styling
    [summarySheet, platformSheet].forEach(sheet => {
      if (sheet.getRow(1)) {
        sheet.getRow(1).font = { bold: true, size: 14 };
      }
      sheet.columns.forEach(column => {
        column.width = 15;
      });
    });
    
    const filename = `${website.name}-report-${Date.now()}.xlsx`;
    const filepath = path.join(this.exportDir, filename);
    
    await workbook.xlsx.writeFile(filepath);
    
    return {
      filepath,
      filename,
      format: 'excel'
    };
  }

  async generateCSVReport(analytics, website, options) {
    const csvData = [];
    
    // Header
    csvData.push(['AI Platform Monitoring Report']);
    csvData.push([`Client: ${website.name}`]);
    csvData.push([`Generated: ${new Date().toLocaleDateString()}`]);
    csvData.push([]);
    
    // Platform Performance
    csvData.push(['Platform Performance']);
    csvData.push(['Platform', 'Mentions', 'Visibility %', 'Avg Position', 'Top 3', 'Cost']);
    
    Object.entries(analytics.platformPerformance).forEach(([platform, stats]) => {
      csvData.push([
        this.getPlatformDisplayName(platform),
        stats.totalMentions,
        stats.visibilityScore.toFixed(1),
        stats.averagePosition ? stats.averagePosition.toFixed(1) : 'N/A',
        stats.top3Count,
        stats.totalCost.toFixed(2)
      ]);
    });
    
    const csvContent = csvData.map(row => row.join(',')).join('\n');
    const filename = `${website.name}-report-${Date.now()}.csv`;
    const filepath = path.join(this.exportDir, filename);
    
    await fs.writeFile(filepath, csvContent);
    
    return {
      filepath,
      filename,
      format: 'csv'
    };
  }

  getPlatformDisplayName(platform) {
    const names = {
      'perplexity': 'Perplexity',
      'gemini': 'Google Gemini',
      'claude': 'Anthropic Claude'
    };
    return names[platform] || platform;
  }

  generateRecommendations(analytics) {
    const recommendations = [];
    const platforms = Object.keys(analytics.platformPerformance);
    
    // Visibility recommendations
    if (analytics.overview.visibilityScore < 50) {
      recommendations.push('Focus on increasing brand visibility across all AI platforms - currently mentioned in less than 50% of queries');
    }
    
    // Platform-specific recommendations
    platforms.forEach(platform => {
      const stats = analytics.platformPerformance[platform];
      const platformName = this.getPlatformDisplayName(platform);
      
      if (stats.visibilityScore < 30) {
        recommendations.push(`Improve presence on ${platformName} - low visibility score of ${stats.visibilityScore.toFixed(1)}%`);
      }
      
      if (stats.averagePosition > 5 && stats.totalMentions > 0) {
        recommendations.push(`Optimize content strategy for ${platformName} - average position is ${stats.averagePosition.toFixed(1)}`);
      }
      
      if (stats.sentimentNegative > stats.sentimentPositive) {
        recommendations.push(`Address negative sentiment on ${platformName} - more negative than positive mentions`);
      }
    });
    
    // Competitive recommendations
    if (analytics.contentGaps && analytics.contentGaps.length > 0) {
      recommendations.push(`Address ${analytics.contentGaps.length} content gaps where competitors outrank your brand`);
    }
    
    // Cost optimization
    const totalCost = analytics.overview.totalApiCost;
    if (totalCost > 50) {
      recommendations.push(`Consider optimizing API usage - monthly cost is $${totalCost.toFixed(2)}`);
    }
    
    return recommendations.length > 0 ? recommendations : ['Continue current strategy - performance is strong across all platforms'];
  }

  async getReportData(websiteId, timeframe) {
    const AnalyticsService = require('./analytics.service');
    const analyticsService = new AnalyticsService();
    
    return analyticsService.getDashboardAnalytics(websiteId, timeframe);
  }

  async getWebsiteInfo(websiteId) {
    const { Website } = require('../models');
    return Website.findById(websiteId);
  }
}

module.exports = ExportService;
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
│   │   │   ├── RecentMentions.tsx
│   │   │   └── RefreshButton.tsx
│   │   ├── analytics/
│   │   │   ├── AdvancedAnalytics.tsx
│   │   │   ├── RankingsPositions.tsx
│   │   │   ├── CompetitiveAnalysis.tsx
│   │   │   └── ContentGapAnalysis.tsx
│   │   ├── platforms/
│   │   │   ├── PlatformPage.tsx
│   │   │   ├── PerplexityPage.tsx
│   │   │   ├── GeminiPage.tsx
│   │   │   └── ClaudePage.tsx
│   │   ├── management/
│   │   │   ├── WebsiteManagement.tsx
│   │   │   ├── CompetitorManagement.tsx
│   │   │   └── SearchTermsManagement.tsx
│   │   ├── settings/
│   │   │   ├── APIKeys.tsx
│   │   │   └── Settings.tsx
│   │   ├── reports/
│   │   │   ├── ReportGenerator.tsx
│   │   │   ├── ExportModal.tsx
│   │   │   └── ReportPreview.tsx
│   │   └── common/
│   │       ├── Button.tsx
│   │       ├── Card.tsx
│   │       ├── Table.tsx
│   │       ├── LoadingSpinner.tsx
│   │       └── Charts/
│   ├── pages/
│   │   ├── Dashboard.tsx
│   │   ├── AdvancedAnalytics.tsx
│   │   ├── Rankings.tsx
│   │   ├── Platform.tsx
│   │   ├── Websites.tsx
│   │   ├── Competitors.tsx
│   │   ├── SearchTerms.tsx
│   │   ├── APIKeys.tsx
│   │   └── Settings.tsx
│   ├── hooks/
│   │   ├── useApi.ts
│   │   ├── useAnalytics.ts
│   │   ├── usePlatforms.ts
│   │   └── useRefresh.ts
│   ├── stores/
│   │   ├── websiteStore.ts
│   │   ├── analyticsStore.ts
│   │   ├── settingsStore.ts
│   │   └── platformStore.ts
│   ├── services/
│   │   ├── api.ts
│   │   ├── analytics.ts
│   │   ├── export.ts
│   │   └── config.ts
│   ├── types/
│   │   ├── website.types.ts
│   │   ├── analytics.types.ts
│   │   ├── platform.types.ts
│   │   └── export.types.ts
│   └── utils/
│       ├── constants.ts
│       ├── formatters.ts
│       ├── chartHelpers.ts
│       └── helpers.ts
├── public/
└── package.json</pre>

## AI Platform Integrations
All AI platform integrations follow the same interface pattern defined in the Base AI Service. Each platform service implements:
1. Query Method: Send search term to platform and get response
2. Position Extraction: Determine brand ranking in response (1-10 or null)
3. Sentiment Analysis: Analyze tone of brand mentions in context
4. Citation Extraction: Find URLs referenced in responses
5. Cost Calculation: Track API usage costs for budget management

### Supported Platforms
- Anthropic (Claude): Claude-3 for detailed analytical responses with reasoning
- Google AI (Gemini): Gemini Pro for comprehensive responses
- Perplexity: Real-time search with citations and current data

### Platform-Specific Features
Each platform page provides:
- Platform performance overview
- Recent queries and responses
- Manual refresh functionality
- Platform-specific insights
- Cost tracking per platform
- Historical performance trends

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
│ 🔍 Perplexity               │
│ ◆ Gemini                    │
│ 🎭 Claude                   │
├─────────────────────────────┤
│ SETUP & CONFIGURATION       │
│ 🌐 My Websites             │
│ 🏢 Competitor Websites      │
│ 🔍 Search Terms Management  │
├─────────────────────────────┤
│ SETTINGS                    │
│ 🔑 API Keys                │
│ ⚙ Settings                │
└─────────────────────────────┘</pre>

#### Styling Requirements:
- Clean white background with subtle shadows
- Active page: Blue highlight (#3B82F6) with white text
- Section headers: Gray uppercase text (text-gray-500)
- Menu items: Gray text (text-gray-700) with hover effects
- Icons: Simple monochrome line icons
- Manual refresh buttons on each page

#### Header Layout
<pre>┌─────────────────────────────────────────────────────────────────┐
│ Dashboard - Overview                              [Refresh All] │
│                                                   Last Updated: │
│                                                   2 hours ago    │
└─────────────────────────────────────────────────────────────────┘</pre>

### Page Specifications
#### 1. Dashboard - Overview
**Filter Section (Full Width)**
```typescript
interface FilterOptions {
  websiteId: string;           // Client website selector
  searchTerms: string[];       // Dropdown of monitored terms
  platform: Platform[];       // AI platforms
  timeRange: '7d' | '30d' | '90d' | '180d' | '1y';
  sentiment: 'all' | 'positive' | 'negative' | 'neutral';
  position: 'all' | 'top3' | '4-5' | '6-10' | 'not-mentioned';
}
```

**Key Performance Stats (Grid Layout)**
```typescript
interface DashboardStats {
  totalMonitoredTerms: number;
  brandVisibilityScore: number;  // 0-100%
  averagePosition: number | null; // 1-10
  top3Rankings: number;
  sentimentScore: number;        // positive sentiment %
  platformCoverage: number;      // platforms with mentions
  totalApiCost: number;          // Cost tracking
}
```

**Main Content Grid**
<pre>┌─────────────────────────────────────────────────────────────────┐
│ Performance Overview Cards + Manual Refresh (Full Width)       │
├─────────────────────────────────┬───────────────────────────────┤
│ Visibility Trends Chart (2/3)  │ Recent Activity (1/3)         │
│ - Time series visualization    │ - Latest mentions             │
│ - Multi-platform comparison    │ - Sentiment indicators        │
│ - Interactive tooltips         │ - Manual refresh button       │
├─────────────────┬─────────────────┬─────────────────────────────┤
│ Search Terms    │ Platform        │ Competitive Intelligence    │
│ Performance     │ Breakdown       │ - Competitor rankings       │
│ - Top performers│ - Performance   │ - Content opportunities     │
│ - Manual refresh│ - Cost tracking │ - Market position analysis  │
└─────────────────┴─────────────────┴─────────────────────────────┘</pre>

#### 2. Advanced Analytics
**Executive Summary (Full Width)**
- AI-generated insights highlighting key trends and opportunities across Perplexity, Gemini, and Claude
- Strategic recommendations based on data analysis from all 3 platforms
- Content gap identification and prioritization

**Intelligence Grid (2 Columns)**
<pre>┌─────────────────────────────────┬─────────────────────────────────┐
│ Market Position Analysis        │ Performance Breakdown           │
│ - Share of voice analysis      │ - 3-platform performance heatmap│
│ - Competitive positioning      │ - Sentiment trends chart        │
│ - Market opportunity scoring   │ - Historical performance         │
└─────────────────────────────────┴─────────────────────────────────┘</pre>
**Content Strategy Intelligence (Full Width)**
- Content gap opportunity matrix with difficulty scoring across all platforms
- ROI predictions for different optimization strategies per platform
- Actionable recommendations with implementation timelines

#### 3. Rankings & Positions
**Rankings Table (Full Width)**
```typescript
interface RankingData {
  searchTerm: string;
  perplexityPosition: number | null;
  geminiPosition: number | null;
  claudePosition: number | null;
  averagePosition: number;
  trend: 'up' | 'down' | 'stable';
  lastUpdated: Date;
  refreshButton: boolean;
}
```

**Platform Performance Cards (Grid)**
- Individual cards for Perplexity, Gemini, and Claude
- Platform-specific metrics and trends
- Position distribution charts for each platform
- Manual refresh buttons for each platform
- Platform-specific insights and optimization recommendations

#### 4. Individual Platform Pages
Each platform page follows this structure:

**Platform Summary Header**
```typescript
interface PlatformSummary {
  performance: string;        // "67% visibility, #3.2 average"
  monthlyTrend: string;      // "18% improvement this month"
  keyInsight: string;        // Main optimization opportunity
  lastQueried: Date;         // When data was last refreshed
  totalCost: number;         // API costs for this platform
}
```

**Performance Intelligence (2 Columns)**
- Left: Current performance analysis and trends
- Right: Optimization recommendations and competitive insights

**Platform Data (3 Columns)**
- Column 1: Recent queries and responses
- Column 2: Position tracking and sentiment analysis
- Column 3: Citation analysis and content insights

**Manual Actions (Full Width)**
- Refresh data for specific search terms
- Bulk query options
- Export platform-specific reports

#### 5. Setup & Configuration Pages
**Website Management**
```typescript
interface Website {
  id: string;
  name: string;
  url: string;
  industry: string;
  description?: string;
  platformsMonitored: Platform[];  // Perplexity, Gemini, Claude
  isActive: boolean;
  createdAt: Date;
}
```

**Simple form with essential fields:**
- Website Name (Client identifier)
- Website URL
- Industry dropdown for context
- Description (optional)
- Platform monitoring checkboxes (Perplexity, Gemini, Claude)
- Active/Inactive toggle

**Competitor Management**
```typescript
interface Competitor {
  id: string;
  websiteId: string;
  name: string;
  url: string;
  platformsMonitored: Platform[];  // Select from 3 available platforms
  isActive: boolean;
  lastAnalyzed: Date;
}
```

**Search Terms Management**
```typescript
interface SearchTerm {
  id: string;
  websiteId: string;
  term: string;
  platformsMonitored: Platform[];  // Assign to specific platforms
  isActive: boolean;
  lastQueried: Date;
  avgPosition: number;
}
```

**Features:**
- Website selection dropdown
- Terms table with 3-platform selection options
- Bulk CSV import capability
- Manual query buttons for testing across selected platforms

#### 6. Settings Pages
**API Keys**
```typescript
interface PlatformConfig {
  platform: Platform;       // 'perplexity' | 'gemini' | 'claude'
  apiKey: string;           // Masked input
  isConfigured: boolean;
  lastTested: Date;
  status: 'valid' | 'invalid' | 'untested';
  monthlyUsage: number;
  estimatedCost: number;
}
```

**Configuration for each of the 3 platforms:
- Perplexity API key configuration with masked input
- Google AI (Gemini) API key setup with validation
- Anthropic (Claude) API key management
- Test connection button for each platform
- Usage and cost tracking per platform
- Connection status indicator

**Settings**
```typescript
interface AppSettings {
  dataRetention: '6months' | '1year' | '2years';
  defaultPlatforms: Platform[];    // Default: ['perplexity', 'gemini', 'claude']
  requestDelay: number;            // Delay between API calls (default: 2000ms)
  reportPreferences: {
    defaultFormat: 'pdf' | 'excel' | 'csv';
    includeCharts: boolean;
    includeCompetitorAnalysis: boolean;
  };
  costAlerts: {
    enabled: boolean;
    monthlyLimit: number;
  };
}
```

**Platform Type Definition:**
```typescript
type Platform = 'perplexity' | 'gemini' | 'claude';
```

## Component Implementation Examples
### Manual Refresh Component
```typescript
interface RefreshButtonProps {
  onRefresh: () => Promise<void>;
  isLoading: boolean;
  lastUpdated?: Date;
  label?: string;
}

export const RefreshButton: React.FC<RefreshButtonProps> = ({
  onRefresh,
  isLoading,
  lastUpdated,
  label = 'Refresh Data'
}) => {
  return (
    <div className="flex items-center gap-3">
      <Button
        onClick={onRefresh}
        disabled={isLoading}
        className="flex items-center gap-2"
      >
        {isLoading ? (
          <LoadingSpinner size="sm" />
        ) : (
          <RefreshIcon className="w-4 h-4" />
        )}
        {label}
      </Button>
      
      {lastUpdated && (
        <span className="text-sm text-gray-500">
          Last updated: {formatRelativeTime(lastUpdated)}
        </span>
      )}
    </div>
  );
};

```

### Dashboard Stats Grid Component
```typescript
interface StatsGridProps {
  stats: DashboardStats;
  websiteId: string;
  isLoading: boolean;
  onRefresh: () => Promise<void>;
}

export const StatsGrid: React.FC<StatsGridProps> = ({ 
  stats, 
  websiteId, 
  isLoading, 
  onRefresh 
}) => {
  if (isLoading) return <StatsGridSkeleton />;

  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center">
        <h2 className="text-lg font-semibold">Performance Overview</h2>
        <RefreshButton onRefresh={onRefresh} isLoading={isLoading} />
      </div>
      
      <div className="grid grid-cols-3 lg:grid-cols-6 gap-4">
        <StatCard
          title="Monitored Terms"
          value={stats.totalMonitoredTerms}
          icon="📊"
        />
        <StatCard
          title="Visibility Score"
          value={`${stats.brandVisibilityScore.toFixed(1)}%`}
          icon="👁"
          trend={getTrendData(websiteId, 'visibility')}
        />
        <StatCard
          title="Avg Position"
          value={stats.averagePosition ? `#${stats.averagePosition.toFixed(1)}` : 'N/A'}
          icon="🎯"
          trend={getTrendData(websiteId, 'position')}
        />
        <StatCard
          title="Top 3 Rankings"
          value={stats.top3Rankings}
          icon="🏆"
        />
        <StatCard
          title="Sentiment"
          value={`${stats.sentimentScore.toFixed(1)}%`}
          icon="😊"
        />
        <StatCard
          title="API Cost (30d)"
          value={`$${stats.totalApiCost.toFixed(2)}`}
          icon="💰"
        />
      </div>
    </div>
  );
};
```

### Platform Query Component
```typescript
interface PlatformQueryProps {
  platform: Platform;
  searchTerms: SearchTerm[];
  onQuery: (platform: Platform, termIds: string[]) => Promise<void>;
}

export const PlatformQuery: React.FC<PlatformQueryProps> = ({
  platform,
  searchTerms,
  onQuery
}) => {
  const [selectedTerms, setSelectedTerms] = useState<string[]>([]);
  const [isQuerying, setIsQuerying] = useState(false);

  const handleQuery = async () => {
    if (selectedTerms.length === 0) return;
    
    setIsQuerying(true);
    try {
      await onQuery(platform, selectedTerms);
    } finally {
      setIsQuerying(false);
    }
  };

  return (
    <Card className="p-6">
      <div className="space-y-4">
        <h3 className="text-lg font-semibold">
          Query {platform.toUpperCase()}
        </h3>
        
        <div className="space-y-2">
          <label className="text-sm font-medium">Select Search Terms:</label>
          <div className="grid grid-cols-2 gap-2">
            {searchTerms.map(term => (
              <label key={term.id} className="flex items-center space-x-2">
                <input
                  type="checkbox"
                  checked={selectedTerms.includes(term.id)}
                  onChange={(e) => {
                    if (e.target.checked) {
                      setSelectedTerms([...selectedTerms, term.id]);
                    } else {
                      setSelectedTerms(selectedTerms.filter(id => id !== term.id));
                    }
                  }}
                />
                <span className="text-sm">{term.term}</span>
              </label>
            ))}
          </div>
        </div>
        
        <Button
          onClick={handleQuery}
          disabled={selectedTerms.length === 0 || isQuerying}
          className="w-full"
        >
          {isQuerying ? (
            <>
              <LoadingSpinner size="sm" />
              Querying {platform}...
            </>
          ) : (
            `Query Selected Terms (${selectedTerms.length})`
          )}
        </Button>
        
        {isQuerying && (
          <div className="text-sm text-gray-600">
            This may take a few minutes due to rate limiting...
          </div>
        )}
      </div>
    </Card>
  );
};
```

### Responsive Design Requirements
#### Breakpoints
- Mobile: < 768px
- Tablet: 768px - 1024px
- Desktop: > 1024px

#### Mobile Adaptations
- Collapsible sidebar with hamburger menu
- Stacked card layouts instead of grids
- Simplified charts optimized for small screens
- Touch-friendly buttons and controls
- Bottom navigation for core functions

#### Desktop Optimizations
- Full sidebar always visible
- Multi-column layouts for maximum information density
- Advanced chart interactions
- Keyboard shortcuts for common actions
- Drag-and-drop functionality for management

## Development Setup
### Environment Configuration
```bash
# .env.local
# Database
DATABASE_PATH="./data/wordsrank.sqlite"

# Application Settings
NODE_ENV="development"
PORT=3001
FRONTEND_URL="http://localhost:3000"

# Logging
LOG_LEVEL="info"

# Default Settings
DEFAULT_REQUEST_DELAY=2000
MAX_RETRIES=3
```

### Local Development Setup
```bash
# Clone and setup
git clone <repository-url>
cd wordsrank

# Install dependencies
npm install
cd backend && npm install
cd ../frontend && npm install
cd ..

# Setup configuration files
cp config/api-keys.example.json config/api-keys.json
cp config/settings.example.json config/settings.json

# Edit API keys (add your actual keys)
nano config/api-keys.json

# Initialize database
cd backend && npm run migrate
cd ..

# Start development servers
npm run dev
```

#### Configuration Files:
- config/api-keys.json - Your actual API keys (gitignored)
- config/api-keys.example.json - Template file (committed to git)
- config/settings.json - Application settings (committed to git)
- All settings are managed via JSON files, no database storage

### Package.json Files
#### Root Package.json
```json
{
  "name": "wordsrank-local",
  "version": "1.0.0",
  "description": "Local AI Platform Monitoring Tool for Freelancers",
  "scripts": {
    "dev": "concurrently \"npm run dev:backend\" \"npm run dev:frontend\"",
    "dev:backend": "cd backend && npm run dev",
    "dev:frontend": "cd frontend && npm run dev",
    "build": "npm run build:frontend && npm run build:backend",
    "build:backend": "cd backend && npm run build",
    "build:frontend": "cd frontend && npm run build",
    "start": "cd backend && npm start",
    "migrate": "cd backend && npm run migrate",
    "seed": "cd backend && npm run seed"
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
    "migrate": "node scripts/migrate.js",
    "seed": "node scripts/seed.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "sqlite3": "^5.1.6",
    "joi": "^17.9.2",
    "axios": "^1.5.0",
    "cors": "^2.8.5",
    "@anthropic-ai/sdk": "^0.6.0",
    "@google-ai/generativelanguage": "^2.4.0",
    "pdfkit": "^0.13.0",
    "exceljs": "^4.3.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "jest": "^29.6.2"
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
    "test": "vitest"
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
    "tailwind-merge": "^1.14.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.15",
    "@vitejs/plugin-react": "^4.0.3",
    "autoprefixer": "^10.4.14",
    "tailwindcss": "^3.3.3",
    "typescript": "^5.0.2",
    "vite": "^4.4.5",
    "vitest": "^0.34.1"
  }
}
```

## Error Handling & Offline Mode
### API Key Management
The application should gracefully handle missing or invalid API keys:
```typescript
interface ApiKeyStatus {
  platform: Platform;
  isConfigured: boolean;
  isValid: boolean;
  lastTested: Date | null;
  error?: string;
}

// Example error states
const errorStates = {
  missingKey: "API key not configured for this platform",
  invalidKey: "API key is invalid or expired", 
  rateLimited: "Rate limit exceeded. Please wait before retrying",
  networkError: "Unable to connect to platform API",
  quotaExceeded: "API quota exceeded for this billing period"
};
```

### UI Error Handling
The interface should provide clear feedback when issues occur:
#### Disabled States
- Refresh buttons disabled when API keys are missing
- Platform pages show configuration prompts instead of data
- Query buttons show requirements (e.g., "Configure API key to enable")

#### Error Notifications
```typescript
interface ErrorNotification {
  type: 'warning' | 'error' | 'info';
  platform?: Platform;
  message: string;
  action?: {
    label: string;
    handler: () => void;
  };
}

// Example error notifications
const errorExamples = [
  {
    type: 'warning',
    platform: 'claude',
    message: 'Claude API key not configured',
    action: {
      label: 'Configure API Keys',
      handler: () => navigate('/settings/api-keys')
    }
  },
  {
    type: 'error', 
    platform: 'perplexity',
    message: 'Rate limit exceeded. Next retry available in 2 minutes',
    action: {
      label: 'Retry Now',
      handler: () => retryQuery()
    }
  }
];
```

#### Offline Mode Behavior
When network connectivity is lost or APIs are unavailable:
- Show cached data with timestamps indicating when data was last refreshed
- Display offline indicator in the header
- Queue failed requests for retry when connectivity returns
- Graceful degradation - show historical data even if real-time refresh fails

#### Error Recovery
```typescript
class ErrorRecoveryService {
  async handleApiError(error: ApiError) {
    switch (error.type) {
      case 'RATE_LIMITED':
        // Show countdown timer until next allowed request
        this.showRateLimitWarning(error.retryAfter);
        break;
        
      case 'INVALID_API_KEY':
        // Navigate to API key configuration
        this.showApiKeyError(error.platform);
        break;
        
      case 'NETWORK_ERROR':
        // Show offline mode
        this.enableOfflineMode();
        break;
        
      case 'QUOTA_EXCEEDED':
        // Show quota exhaustion warning
        this.showQuotaWarning(error.platform);
        break;
    }
  }
}
```

## Backup & Data Export
### Manual Backup Workflow
Since WordsRank uses local SQLite storage, backing up your data is straightforward:
#### Database Backup
```bash
# Simple file copy backup
cp data/wordsrank.sqlite backups/wordsrank-backup-$(date +%Y%m%d).sqlite

# Or create a backup script
#!/bin/bash
BACKUP_DIR="backups"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR

# Backup database
cp data/wordsrank.sqlite $BACKUP_DIR/wordsrank-$DATE.sqlite

# Backup configuration
cp config/settings.json $BACKUP_DIR/settings-$DATE.json
cp config/api-keys.json $BACKUP_DIR/api-keys-$DATE.json

echo "Backup completed: $BACKUP_DIR/wordsrank-$DATE.sqlite"
```

#### Export Data to Portable Formats
The application should provide export functionality for data portability:
```javascript
// Backend export service
class DataExportService {
  async exportAllData() {
    const data = {
      websites: await Website.findAll(),
      competitors: await Competitor.findAll(), 
      searchTerms: await SearchTerm.findAll(),
      mentions: await AIMention.findAll(),
      analytics: await AnalyticsSnapshot.findAll(),
      exportDate: new Date().toISOString()
    };
    
    return data;
  }
  
  async exportToCSV() {
    // Export data in CSV format for analysis in Excel/Sheets
    const mentions = await AIMention.findAll();
    return this.convertToCSV(mentions);
  }
  
  async exportToJSON() {
    // Complete data export in JSON format
    return this.exportAllData();
  }
}
```

#### Pre-Upgrade Backup Checklist
Before updating WordsRank to a new version:
1. Stop the application
2. Backup database file: cp data/wordsrank.sqlite backups/
3. Backup configuration: cp config/*.json backups/
4. Export data (optional): Use the export feature to create CSV/JSON exports
5. Test backup: Verify backup files are complete and not corrupted
6. Proceed with upgrade

#### Data Recovery
If you need to restore from backup:
```bash
# Stop WordsRank application first
# Restore database
cp backups/wordsrank-backup-20240315.sqlite data/wordsrank.sqlite

# Restore configuration if needed
cp backups/settings-20240315.json config/settings.json
cp backups/api-keys-20240315.json config/api-keys.json

# Restart application
npm start
```

#### Automated Backup (Optional)
For regular backups, you can set up a simple cron job:
```bash
# Add to crontab (crontab -e)
# Daily backup at 2 AM
0 2 * * * /path/to/wordsrank/scripts/backup.sh

# Weekly cleanup of old backups (keep last 30 days)
0 3 * * 0 find /path/to/wordsrank/backups -name "*.sqlite" -mtime +30 -delete
```

### Backend Testing
Focus on core functionality rather than infrastructure:
```javascript
// tests/monitoring.test.js
const MonitoringService = require('../src/services/monitoring.service');
const { SearchTerm, Website } = require('../src/models');

describe('Monitoring Service', () => {
  let monitoringService;

  beforeEach(() => {
    monitoringService = new MonitoringService();
  });

  describe('Platform Querying', () => {
    it('should query platform and store results', async () => {
      const searchTerm = {
        id: 1,
        term: 'best project management tools',
        website_id: 1
      };
      
      const brandTerms = ['Trello', 'Asana'];
      
      const result = await monitoringService.queryPlatform(
        searchTerm, 
        'claude', 
        brandTerms
      );

      expect(result).toHaveProperty('platform', 'claude');
      expect(result).toHaveProperty('position');
      expect(result).toHaveProperty('sentiment');
      expect(typeof result.apiCost).toBe('number');
    });
  });

  describe('Position Extraction', () => {
    it('should correctly identify brand position in response', () => {
      const content = 'Trello is excellent for project management. Asana is also good.';
      const brandTerms = ['Trello'];
      
      const servie = monitoringService.aiServiceFactory.getService('claude');
      const position = service.extractPosition(content, brandTerms);
      
      expect(position).toBe(1);
    });
  });
});

```

### Frontend Testing
```typescript
// tests/Dashboard.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { Dashboard } from '../src/pages/Dashboard';
import { vi } from 'vitest';

vi.mock('../src/services/api', () => ({
  getDashboardAnalytics: vi.fn(),
  refreshPlatformData: vi.fn()
}));

describe('Dashboard', () => {
  it('renders stats and refresh functionality', async () => {
    const mockStats = {
      totalMonitoredTerms: 25,
      brandVisibilityScore: 68.5,
      averagePosition: 3.2,
      top3Rankings: 8
    };

    vi.mocked(getDashboardAnalytics).mockResolvedValue({
      overview: mockStats,
      platformPerformance: {}
    });

    render(<Dashboard />);

    await waitFor(() => {
      expect(screen.getByText('25')).toBeInTheDocument();
      expect(screen.getByText('68.5%')).toBeInTheDocument();
    });

    const refreshButton = screen.getByText('Refresh Data');
    expect(refreshButton).toBeInTheDocument();
    
    fireEvent.click(refreshButton);
    // Test refresh functionality
  });
});
```

## Deployment & Setup
### Local Installation Package
Create a simple installer for end users:
```javascript
// scripts/install.js
const fs = require('fs');
const path = require('path');
const { execSync } = require('child_process');

class WordsRankInstaller {
  async install() {
    console.log('Installing WordsRank Local...');
    
    // Create data directory
    const dataDir = path.join(process.cwd(), 'data');
    if (!fs.existsSync(dataDir)) {
      fs.mkdirSync(dataDir, { recursive: true });
    }
    
    // Create exports directory
    const exportsDir = path.join(process.cwd(), 'exports');
    if (!fs.existsSync(exportsDir)) {
      fs.mkdirSync(exportsDir, { recursive: true });
    }
    
    // Copy configuration templates
    this.copyConfigTemplates();
    
    // Initialize database
    console.log('Initializing database...');
    execSync('npm run migrate', { stdio: 'inherit' });
    
    console.log('Installation complete!');
    console.log('');
    console.log('Next steps:');
    console.log('1. Edit config/api-keys.json with your API keys');
    console.log('2. Run "npm start" to launch WordsRank');
    console.log('3. Open http://localhost:3000 in your browser');
  }
  
  copyConfigTemplates() {
    const configDir = path.join(process.cwd(), 'config');
    
    const apiKeysTemplate = {
      anthropic: "",
      google: "",
      perplexity: ""
    };
    
    const settingsTemplate = {
      app: {
        name: "WordsRank",
        version: "1.0.0",
        data_retention_days: 365
      },
      monitoring: {
        default_platforms: ["perplexity", "gemini", "claude"],
        request_delay_ms: 2000,
        max_retries: 3
      },
      reporting: {
        default_format: "pdf",
        include_charts: true,
        include_competitor_analysis: true
      }
    };
    
    // Create example templates (committed to git)
    fs.writeFileSync(
      path.join(configDir, 'api-keys.example.json'),
      JSON.stringify(apiKeysTemplate, null, 2)
    );
    
    fs.writeFileSync(
      path.join(configDir, 'settings.example.json'),
      JSON.stringify(settingsTemplate, null, 2)
    );
    
    // Create actual config files (gitignored)
    fs.writeFileSync(
      path.join(configDir, 'api-keys.json'),
      JSON.stringify(apiKeysTemplate, null, 2)
    );
    
    fs.writeFileSync(
      path.join(configDir, 'settings.json'),
      JSON.stringify(settingsTemplate, null, 2)
    );
  }
}

const installer = new WordsRankInstaller();
installer.install();
```

### Startup Script
```javascript
// scripts/start.js
const path = require('path');
const { spawn } = require('child_process');

class WordsRankLauncher {
  async start() {
    console.log('Starting WordsRank...');
    
    // Check configuration
    this.checkConfiguration();
    
    // Start backend
    const backend = spawn('node', ['src/app.js'], {
      cwd: path.join(__dirname, '../backend'),
      stdio: 'inherit'
    });
    
    // Start frontend (in production, this would be served by backend)
    setTimeout(() => {
      const frontend = spawn('npm', ['run', 'preview'], {
        cwd: path.join(__dirname, '../frontend'),
        stdio: 'inherit'
      });
    }, 3000);
    
    console.log('WordsRank is running!');
    console.log('Open http://localhost:3000 in your browser');
  }
  
  checkConfiguration() {
    const apiKeysPath = path.join(__dirname, '../config/api-keys.json');
    const settingsPath = path.join(__dirname, '../config/settings.json');
    
    if (!require('fs').existsSync(apiKeysPath)) {
      console.error('API keys configuration not found. Run "npm run install" first.');
      process.exit(1);
    }
    
    if (!require('fs').existsSync(settingsPath)) {
      console.error('Settings configuration not found. Run "npm run install" first.');
      process.exit(1);
    }
    
    const config = require(apiKeysPath);
    const hasApiKey = Object.values(config).some(key => key && key.length > 0);
    
    if (!hasApiKey) {
      console.warn('Warning: No API keys configured. Add your API keys to config/api-keys.json');
    }
  }
}

const launcher = new WordsRankLauncher();
launcher.start();
```
