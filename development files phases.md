# WordsRank - Complete File Structure & Content Guide
#### Total Files: 102

## Backend Files (40 files)
## Controllers (8 files)
1. auth.controller.js - Handles user registration, login, logout, password reset, and profile management
2. dashboard.controller.js - Manages dashboard data aggregation, stats calculation, and overview analytics
3. websites.controller.js - CRUD operations for websites, status management, and monitoring configuration
4. competitors.controller.js - Competitor website management, addition, deletion, and monitoring setup
5. searchTerms.controller.js - Search term CRUD, bulk import, platform assignment, and status management
6. analytics.controller.js - Advanced analytics, competitive analysis, ranking trends, and reporting
7. platforms.controller.js - AI platform configuration, API key testing, and platform-specific analytics
8. team.controller.js - Team member management, invitations, role assignments, and permissions

## Services (12 files)
9. services/ai-platforms/base.service.js - Abstract base class defining common AI platform interface and shared methods
10. services/ai-platforms/openai.service.js - ChatGPT API integration, query processing, and response analysis
11. services/ai-platforms/anthropic.service.js - Claude API integration with conversation handling and analysis
12. services/ai-platforms/google.service.js - Gemini API integration focusing on mobile and voice search optimization
13. services/ai-platforms/perplexity.service.js - Perplexity API integration with citation tracking and authority scoring
14. services/ai-platforms/mistral.service.js - Mistral AI integration with European market focus
15. services/ai-platforms/deepseek.service.js - DeepSeek API integration for technical and research queries
16. services/ai-platforms/factory.service.js - Factory pattern for creating appropriate AI service instances
17. services/auth.service.js - Authentication logic, JWT token management, and user session handling
18. services/analytics.service.js - Data aggregation, trend calculation, competitive analysis, and insights generation
19. services/monitoring.service.js - Automated monitoring jobs, scheduling, and data collection orchestration
20. services/scheduler.service.js - Background job scheduling, cron job management, and task queuing

## Models (4 files)
21. models/User.js - User data model with authentication, profile, and plan management methods
22. models/Website.js - Website entity model with monitoring configuration and relationship management
23. models/SearchTerm.js - Search term model with platform assignments and tracking status
24. models/AIMention.js - AI platform mention model with position, sentiment, and performance data

## Middleware (3 files)
25. middleware/auth.middleware.js - JWT authentication, role-based authorization, and security validation
26. middleware/validation.middleware.js - Request validation using Joi schemas for all API endpoints
27. middleware/rateLimit.middleware.js - API rate limiting, abuse prevention, and quota management

## Routes (5 files)
28. routes/auth.routes.js - Authentication endpoints for login, registration, and profile management
29. routes/dashboard.routes.js - Dashboard data endpoints and overview analytics
30. routes/websites.routes.js - Website and competitor management endpoints
31. routes/analytics.routes.js - Advanced analytics, rankings, and reporting endpoints
32. routes/platforms.routes.js - AI platform configuration and platform-specific analytics

## Utils (4 files)
33. utils/database.js - Database connection management, query helpers, and transaction handling
34. utils/encryption.js - API key encryption/decryption for secure storage of sensitive credentials
35. utils/logger.js - Application logging configuration with multiple transport levels
36. utils/helpers.js - Common utility functions for data processing and formatting

## Core & Config (3 files)
37. src/app.js - Main Express application setup, middleware configuration, and route registration
38. config/database.config.js - PostgreSQL database configuration and connection settings
39. config/app.config.js - Application-wide configuration settings and environment variables

## Package Files (1 file)
40. package.json - Backend dependencies, scripts, and Node.js project configuration

## Frontend Files (49 files)
## Layout Components (3 files)
41. components/layout/Sidebar.tsx - Navigation sidebar with platform sections, badges, and active state management
42. components/layout/Header.tsx - Dynamic page title, user profile section, and dropdown menu
43. components/layout/Layout.tsx - Main layout wrapper combining sidebar, header, and content areas

## Dashboard Components (4 files)
44. components/dashboard/DashboardOverview.tsx - Main dashboard container with performance cards and grid layout
45. components/dashboard/StatsGrid.tsx - Key performance stats with metrics like visibility score and platform coverage
46. components/dashboard/PerformanceChart.tsx - Interactive time-series charts for visibility trends across platforms
47. components/dashboard/RecentMentions.tsx - Recent activity feed with latest mentions and sentiment indicators

## Analytics Components (3 files)
48. components/analytics/AdvancedAnalytics.tsx - Executive summary, competitive intelligence, and business-focused insights
49. components/analytics/RankingsPositions.tsx - Comprehensive rankings table with position tracking across platforms
50. components/analytics/CompetitiveAnalysis.tsx - Competitor comparison tools and market position analysis

## Platform Components (7 files)
51. components/platforms/PlatformPage.tsx - Base platform page template with common functionality
52. components/platforms/ChatGPTPage.tsx - ChatGPT-specific performance analysis and optimization recommendations
53. components/platforms/PerplexityPage.tsx - Perplexity platform with citation analysis and authority tracking
54. components/platforms/GeminiPage.tsx - Gemini platform focusing on mobile and voice search optimization
55. components/platforms/ClaudePage.tsx - Claude platform with comprehensive response analysis
56. components/platforms/MistralPage.tsx - Mistral AI platform with European market focus
57. components/platforms/DeepSeekPage.tsx - DeepSeek platform for technical and research-focused queries

## Management Components (3 files)
58. components/management/WebsiteManagement.tsx - Simple website addition with URL, name, industry, and platform selection
59. components/management/CompetitorManagement.tsx - Competitor website management with monitoring configuration
60. components/management/SearchTermsManagement.tsx - Search term tracking with bulk import and platform assignment

## Account Components (3 files)
61. components/account/ProfileTeam.tsx - Two-column layout for profile editing and team member management
62. components/account/APIKeys.tsx - AI platform API key configuration with connection testing
63. components/account/Settings.tsx - Account settings, monitoring preferences, and notification configuration

## Common Components (3 files)
64. components/common/Button.tsx - Reusable button component with variants and accessibility features
65. components/common/Card.tsx - Card container component for consistent styling across the application
66. components/common/Table.tsx - Data table component with sorting, filtering, and responsive design

## Page Components (6 files)
67. pages/Dashboard.tsx - Main dashboard page combining overview stats and performance insights
68. pages/Analytics.tsx - Advanced analytics page with executive summaries and competitive intelligence
69. pages/Rankings.tsx - Rankings and positions page with comprehensive platform comparison
70. pages/Platform.tsx - Individual platform analysis page with platform-specific insights
71. pages/Login.tsx - User authentication page with login and registration forms
72. pages/Setup.tsx - Initial setup page for new users and website configuration

## Hooks (3 files)
73. hooks/useAuth.ts - Authentication state management and user session handling
74. hooks/useApi.ts - API request handling with loading states and error management
75. hooks/useAnalytics.ts - Analytics data fetching and caching for dashboard components

## Stores (3 files)
76. stores/authStore.ts - Zustand store for authentication state and user management
77. stores/websiteStore.ts - Global state management for websites and monitoring configuration
78. stores/analyticsStore.ts - Analytics data caching and state management for performance metrics

## Services (3 files)
79. services/api.ts - Centralized API client with axios configuration and interceptors
80. services/auth.ts - Frontend authentication service with token management
81. services/analytics.ts - Analytics data processing and transformation for UI components

## Types (3 files)
82. types/auth.types.ts - TypeScript interfaces for authentication, user profiles, and team members
83. types/website.types.ts - Type definitions for websites, competitors, and search terms
84. types/analytics.types.ts - Analytics data structures, performance metrics, and dashboard interfaces

## Utils (3 files)
85. utils/constants.ts - Application constants, API endpoints, and configuration values
86. utils/formatters.ts - Data formatting utilities for numbers, dates, and metrics display
87. utils/helpers.ts - Common utility functions for data manipulation and UI helpers

## Package Files (1 file)
88. package.json - Frontend dependencies, build scripts, and React project configuration

## Charts Directory (1 file)
89. components/common/Charts/index.ts - Chart component exports and Recharts integration

## Configuration & Infrastructure Files (13 files)
## Environment & Setup (4 files)
90. .env.example - Environment variables template with database, API keys, and application settings
91. docker-compose.dev.yml - Development Docker configuration with PostgreSQL, backend, and frontend
92. docker-compose.yml - Production Docker setup with nginx, SSL, and network configuration
93. nginx/nginx.conf - Nginx reverse proxy configuration with SSL, rate limiting, and security headers

## Database & Scripts (2 files)
94. scripts/migrate.js - Database migration execution script with file tracking and rollback support
95. migrations/001_initial_schema.sql - Complete database schema with all tables, indexes, and constraints

## Testing Configuration (2 files)
96. backend/jest.config.js - Backend testing configuration with coverage thresholds and setup
97. frontend/vitest.config.ts - Frontend testing setup with React Testing Library and coverage

## Sample Test Files (4 files)
98. tests/auth.test.js - Backend authentication testing examples with user registration and login
99. tests/ai-platforms.test.js - AI platform service testing with query processing and analysis
100. tests/Dashboard.test.tsx - Frontend dashboard component testing with mock data
101. tests/components/StatsGrid.test.tsx - Component testing for stats grid with various data states

## CI/CD & Deployment (1 file)
102. .github/workflows/deploy.yml - GitHub Actions workflow for testing, building, and deployment

## File Organization Summary:
- Backend Logic: 40 files for API, services, models, and data processing
- Frontend UI: 49 files for React components, pages, and user interface
- Infrastructure: 13 files for deployment, testing, and configuration

Each file serves a specific purpose in creating the complete WordsRank AI platform monitoring tool as detailed in the specifications and UI requirements.


# WordsRank Development Phases - 15-Phase Implementation Plan

## Phase 1: Project Foundation & Infrastructure Setup
### Files to Create (8 files):
- package.json (root) - Master project configuration
- package.json (backend) - Backend dependencies and scripts
- package.json (frontend) - Frontend dependencies and scripts
- .env.example - Environment variables template
- docker-compose.dev.yml - Development environment setup
- config/database.config.js - Database connection configuration
- config/app.config.js - Application settings
- utils/database.js - Database connection utilities

### Content & Integration:
- Establish monorepo structure with backend/frontend separation
- Configure Node.js, Express, React, and TypeScript environments
- Set up PostgreSQL database connection
- Establish development Docker environment
- Configure environment variable management

### Testing Phase 1:
- Verify project structure creation
- Test database connection establishment
- Validate Docker environment startup
- Confirm package installations and dependencies
- Test environment variable loading

## Phase 2: Database Schema & Core Models
### Files to Create (6 files):
- migrations/001_initial_schema.sql - Complete database schema
- scripts/migrate.js - Migration execution script
- models/User.js - User data model
- models/Website.js - Website entity model
- models/SearchTerm.js - Search term model
- models/AIMention.js - AI mention data model

### Content & Integration:
- Create all database tables with proper relationships
- Implement data models with CRUD operations
- Establish foreign key constraints and indexes
- Set up migration system for schema versioning
- Create model relationships and validation rules

### Testing Phase 2:
- Run database migrations successfully
- Test all model CRUD operations
- Verify foreign key constraints
- Validate data model relationships
- Test migration rollback capabilities

## Phase 3: Authentication System
### Files to Create (6 files):
- middleware/auth.middleware.js - JWT authentication middleware
- services/auth.service.js - Authentication business logic
- controllers/auth.controller.js - Authentication endpoints
- routes/auth.routes.js - Authentication routing
- utils/encryption.js - Encryption utilities for API keys
- src/app.js - Basic Express application setup

### Content & Integration:
- Implement JWT token generation and validation
- Create user registration and login functionality
- Set up password hashing with bcrypt
- Establish role-based access control
- Integrate authentication middleware with routing

### Testing Phase 3:
- Test user registration flow
- Verify login/logout functionality
- Validate JWT token generation and verification
- Test password hashing and comparison
- Verify role-based access restrictions

## Phase 4: Core API Infrastructure
### Files to Create (4 files):
- middleware/validation.middleware.js - Request validation
- middleware/rateLimit.middleware.js - Rate limiting protection
- utils/logger.js - Application logging system
- utils/helpers.js - Common utility functions

### Content & Integration:
- Implement request validation using Joi schemas
- Set up rate limiting for API protection
- Configure logging system with multiple levels
- Create reusable utility functions
- Integrate middleware stack with Express app

### Testing Phase 4:
- Test request validation with valid/invalid data
- Verify rate limiting functionality
- Validate logging output at different levels
- Test utility function operations
- Confirm middleware integration order

## Phase 5: Website & Search Term Management
### Files to Create (6 files):
- controllers/websites.controller.js - Website CRUD operations
- controllers/competitors.controller.js - Competitor management
- controllers/searchTerms.controller.js - Search term management
- routes/websites.routes.js - Website routing
- backend/jest.config.js - Testing configuration
- tests/auth.test.js - Authentication tests

### Content & Integration:
- Implement website addition and management
- Create competitor tracking functionality
- Build search term CRUD operations
- Set up bulk import capabilities for search terms
- Establish relationships between websites and search terms

### Testing Phase 5:
- Test website creation and management
- Verify competitor addition and deletion
- Test search term CRUD operations
- Validate bulk import functionality
- Test data relationships and constraints

## Phase 6: AI Platform Service Foundation
### Files to Create (4 files):
- services/ai-platforms/base.service.js - Abstract AI service interface
- services/ai-platforms/factory.service.js - Service factory pattern
- services/monitoring.service.js - Monitoring orchestration
- services/scheduler.service.js - Background job scheduling

### Content & Integration:
- Define common AI platform interface
- Implement factory pattern for platform services
- Create monitoring job orchestration
- Set up background task scheduling
- Establish position extraction and sentiment analysis framework

### Testing Phase 6:
- Test base service interface implementation
- Verify factory pattern service creation
- Test monitoring job scheduling
- Validate background task execution
- Test position extraction and sentiment analysis algorithms

## Phase 7: OpenAI/ChatGPT Integration
### Files to Create (4 files):
- services/ai-platforms/openai.service.js - ChatGPT API integration
- controllers/platforms.controller.js - Platform management
- routes/platforms.routes.js - Platform routing
- tests/ai-platforms.test.js - AI platform testing

### Content & Integration:
- Implement OpenAI API client and query processing
- Create response analysis and position detection
- Set up cost calculation and usage tracking
- Integrate with monitoring service
- Establish error handling and retry logic

### Testing Phase 7:
- Test OpenAI API connectivity and queries
- Verify response analysis and position extraction
- Test cost calculation accuracy
- Validate error handling and retry mechanisms
- Test integration with monitoring service

## Phase 8: Anthropic & Google AI Integration
### Files to Create (2 files):
- services/ai-platforms/anthropic.service.js - Claude API integration
- services/ai-platforms/google.service.js - Gemini API integration

### Content & Integration:
- Implement Anthropic Claude API client
- Create Google Gemini API integration
- Adapt base service methods for each platform
- Configure platform-specific analysis logic
- Integrate with factory service for dynamic instantiation

### Testing Phase 8:
- Test Claude API connectivity and responses
- Verify Gemini API integration and mobile optimization
- Test platform-specific analysis logic
- Validate factory service platform creation
- Test multi-platform monitoring capabilities

## Phase 9: Remaining AI Platforms
### Files to Create (3 files):
- services/ai-platforms/perplexity.service.js - Perplexity API integration
- services/ai-platforms/mistral.service.js - Mistral AI integration
- services/ai-platforms/deepseek.service.js - DeepSeek integration

### Content & Integration:
- Implement Perplexity custom HTTP client
- Create Mistral AI integration with European focus
- Build DeepSeek integration for technical queries
- Complete factory service with all platforms
- Test full multi-platform monitoring system

### Testing Phase 9:
- Test all remaining platform integrations
- Verify custom HTTP client implementations
- Test platform-specific features and optimizations
- Validate complete factory service functionality
- Test end-to-end multi-platform monitoring

## Phase 10: Analytics & Dashboard Backend
### Files to Create (5 files):
- services/analytics.service.js - Analytics data processing
- controllers/analytics.controller.js - Analytics endpoints
- controllers/dashboard.controller.js - Dashboard data aggregation
- routes/analytics.routes.js - Analytics routing
- routes/dashboard.routes.js - Dashboard routing

### Content & Integration:
- Implement analytics data aggregation and calculation
- Create competitive analysis algorithms
- Build dashboard statistics generation
- Set up trend analysis and forecasting
- Integrate with all AI platform services for data collection

### Testing Phase 10:
- Test analytics data aggregation accuracy
- Verify competitive analysis calculations
- Test dashboard statistics generation
- Validate trend analysis algorithms
- Test integration with platform services

## Phase 11: Frontend Foundation & Authentication
### Files to Create (10 files):
- components/layout/Layout.tsx - Main layout wrapper
- components/layout/Header.tsx - Dynamic header component
- components/layout/Sidebar.tsx - Navigation sidebar
- pages/Login.tsx - Authentication page
- services/api.ts - API client configuration
- services/auth.ts - Frontend auth service
- hooks/useAuth.ts - Authentication hook
- stores/authStore.ts - Authentication state
- types/auth.types.ts - Authentication types
- frontend/vitest.config.ts - Frontend testing config

### Content & Integration:
- Set up React application structure with TypeScript
- Implement authentication state management
- Create responsive layout with sidebar and header
- Build login/registration forms
- Establish API client with interceptors

### Testing Phase 11:
- Test React application startup
- Verify authentication state management
- Test login/logout functionality
- Validate responsive layout behavior
- Test API client authentication integration

## Phase 12: Dashboard & Core Components
### Files to Create (9 files):
- pages/Dashboard.tsx - Main dashboard page
- components/dashboard/DashboardOverview.tsx - Dashboard container
- components/dashboard/StatsGrid.tsx - Performance statistics
- components/dashboard/PerformanceChart.tsx - Trend visualizations
- components/dashboard/RecentMentions.tsx - Activity feed
- components/common/Button.tsx - Reusable button component
- components/common/Card.tsx - Card container
- components/common/Table.tsx - Data table component
- hooks/useApi.ts - API request management

### Content & Integration:
- Build main dashboard with performance overview
- Create interactive charts using Recharts
- Implement stats grid with key metrics
- Build recent activity feed
- Establish common UI component library

### Testing Phase 12:
- Test dashboard data loading and display
- Verify chart interactions and responsiveness
- Test stats grid calculations
- Validate activity feed updates
- Test common component reusability

## Phase 13: Analytics & Platform Pages
### Files to Create (12 files):
- pages/Analytics.tsx - Advanced analytics page
- pages/Rankings.tsx - Rankings and positions
- pages/Platform.tsx - Individual platform analysis
- components/analytics/AdvancedAnalytics.tsx - Business intelligence
- components/analytics/RankingsPositions.tsx - Rankings table
- components/analytics/CompetitiveAnalysis.tsx - Competitor comparison
- components/platforms/PlatformPage.tsx - Base platform template
- components/platforms/ChatGPTPage.tsx - ChatGPT analysis
- components/platforms/PerplexityPage.tsx - Perplexity insights
- components/platforms/GeminiPage.tsx - Gemini mobile focus
- components/platforms/ClaudePage.tsx - Claude comprehensive analysis
- hooks/useAnalytics.ts - Analytics data management

### Content & Integration:
- Build advanced analytics with executive summaries
- Create comprehensive rankings table with sorting
- Implement platform-specific analysis pages
- Build competitive intelligence dashboards
- Integrate with analytics backend services

### Testing Phase 13:
- Test analytics data visualization
- Verify rankings table functionality
- Test platform-specific insights
- Validate competitive analysis accuracy
- Test data integration and updates

## Phase 14: Management & Settings Pages
### Files to Create (10 files):
- pages/Setup.tsx - Initial setup workflow
- components/management/WebsiteManagement.tsx - Website CRUD
- components/management/CompetitorManagement.tsx - Competitor tracking
- components/management/SearchTermsManagement.tsx - Term management
- components/account/ProfileTeam.tsx - Profile and team settings
- components/account/APIKeys.tsx - AI platform configuration
- components/account/Settings.tsx - Account preferences
- components/platforms/MistralPage.tsx - Mistral AI analysis
- components/platforms/DeepSeekPage.tsx - DeepSeek technical focus
- stores/websiteStore.ts - Website state management

### Content & Integration:
- Build website and competitor management interfaces
- Create search term bulk import functionality
- Implement team member management
- Build API key configuration and testing
- Create user settings and preferences

### Testing Phase 14:
- Test website and competitor CRUD operations
- Verify search term management and bulk import
- Test team management functionality
- Validate API key configuration and testing
- Test settings persistence and updates

## Phase 15: Final Integration & Deployment
### Files to Create (15 files):
- controllers/team.controller.js - Team management backend
- stores/analyticsStore.ts - Analytics state management
- types/website.types.ts - Website and competitor types
- types/analytics.types.ts - Analytics data types
- utils/constants.ts - Application constants
- utils/formatters.ts - Data formatting utilities
- utils/helpers.ts - Frontend utility functions
- components/common/Charts/index.ts - Chart exports
- tests/Dashboard.test.tsx - Dashboard component tests
- tests/components/StatsGrid.test.tsx - Component testing
- docker-compose.yml - Production Docker setup
- nginx/nginx.conf - Production nginx configuration
- .github/workflows/deploy.yml - CI/CD pipeline
- README.md - Project documentation
- CHANGELOG.md - Version history

### Content & Integration:
- Complete team management functionality
- Finalize all state management stores
- Complete type definitions and utilities
- Implement comprehensive testing suite
- Set up production deployment configuration
- Configure CI/CD pipeline

### Testing Phase 15:
- Run complete end-to-end testing
- Verify all feature integrations
- Test production deployment process
- Validate CI/CD pipeline functionality
- Perform security and performance testing
- Conduct user acceptance testing

## Phase Dependencies & Integration Points:
### Critical Dependencies:
- Phase 1-2: Foundation for all subsequent phases
- Phase 3: Required for all authenticated features
- Phase 6-9: AI platform services must be completed before analytics
- Phase 10: Backend analytics required for frontend dashboard
- Phase 11: Frontend foundation required for all UI phases

### Integration Testing Strategy:
- Each phase includes specific testing requirements that validate both individual functionality and integration with previous phases. Testing focuses on:
- Unit Testing: Individual component/service functionality
- Integration Testing: Cross-component communication
- API Testing: Backend endpoint validation
- UI Testing: Frontend component behavior
- End-to-End Testing: Complete user workflows

### Deployment Readiness:
- By Phase 15, the application will have:
- Complete backend API with all AI platform integrations
- Full frontend interface with all specified pages
- Comprehensive testing coverage
- Production-ready deployment configuration
- CI/CD pipeline for automated deployments
- Documentation and maintenance procedures

This phased approach ensures each development stage is tested and validated before proceeding, reducing integration issues and maintaining code quality throughout the development process.
