# Botcart SaaS Platform - Implementation Context

**IMPORTANT: Please read this entire document carefully to understand the full context of the Botcart SaaS platform before proceeding with implementation.**

## Introduction

This document outlines the complete specification for Botcart, an AI-powered platform that automates Facebook comment responses. As the AI assistant, it's crucial you understand all components and their interactions to provide optimal implementation guidance.

## Core Concept

Botcart is a SaaS platform where users can:
1. Connect their Facebook business pages
2. Select which posts to monitor for comments
3. Set monitoring intervals (10 mins - 1 hour)
4. Provide business knowledge to personalize AI responses
5. Automatically respond to new comments with AI-generated replies

## Detailed System Architecture

### 1. User Authentication & Token Management

- **User Registration/Login**:
  - Primary authentication via Supabase Auth (email/password, magic links)
  - Google OAuth integration through Supabase Auth
  - Facebook OAuth integration handled by backend for security
  - Required Facebook permissions: pages_read_engagement, pages_manage_posts, pages_manage_engagement

- **Authentication Flow**:
  - Frontend redirects users to backend endpoint (`/api/auth/facebook`) for Facebook OAuth
  - Backend handles OAuth code-to-token exchange with Facebook
  - Backend securely stores tokens in Supabase, associated with user ID
  - Each user gets a unique identifier upon registration for user-specific data management

- **Token Management**:
  - Backend handles all token refreshing and expiration monitoring
  - Tokens are encrypted before storage in Supabase
  - Implementation of proper token rotation and refresh mechanisms
  - Automatic token validation before API calls

- **Subscription Management**:
  - User tiers (Free, Premium) tracked in user profile
  - Usage limits enforced based on subscription tier
  - Infrastructure ready for future payment integration
  - API endpoints structured to validate subscription entitlements

- **Data Storage**:
  - User profile data in Supabase with subscription tier information
  - Facebook page access tokens securely stored and encrypted in Supabase
  - Page metadata (name, ID, avatar) stored for dashboard display

- **Implementation Notes**:
  - Never expose Facebook App Secret in frontend code
  - Implement robust token refresh and rotation mechanisms
  - Build subscription-aware services to enable future premium features
  - Store only necessary data to comply with data protection regulations
  - Associate all data with user IDs for proper multi-tenant isolation

### 2. Dashboard UI/UX

- **Main Components**:
  - Header with app logo, user profile, and notifications
  - Sidebar navigation with main sections
  - Main content area displaying current section
  - Subscription status/upgrade banner (for tier visibility)

- **Sidebar Navigation**:
  - Dashboard Home (overview statistics)
  - Connected Pages (manage Facebook pages)
  - Posts Management (select and monitor posts)
  - Knowledge Base (business information)
  - AI Conversations (view comment-reply history)
  - Settings
  - Subscription Management (for future implementation)

- **Connected Pages View**:
  - List of connected Facebook pages (retrieved from backend API)
  - Page metadata (name, followers, etc.)
  - Quick actions (view posts, edit settings)
  - Page connection limits based on subscription tier

- **Data Flow**:
  - Frontend requests data from Node.js backend API
  - Backend fetches data from Supabase and Facebook API as needed
  - Backend enforces subscription-based limits before returning data
  - Frontend displays data and handles user interactions
  - Any changes or selections are sent to backend for processing

### 3. Posts Management System

- **Features**:
  - Fetch and display posts from connected Facebook pages (retrieved from backend)
  - Multi-select functionality for posts
  - Moderation interval selector (10 mins, 15 mins, 30 mins, 1 hour)
  - Enable/disable monitoring for selected posts
  - "Start Monitoring" button to initiate backend processes
  - Post monitoring limits based on subscription tier

- **Data Flow**:
  - Frontend sends API request to backend to retrieve posts
  - Backend uses stored Facebook tokens to fetch posts from Facebook API
  - Backend enforces subscription limits before returning data
  - Frontend displays posts and allows selection
  - When posts are selected and "Start Monitoring" clicked, frontend sends selection data to backend
  - Backend stores selection in Supabase and initializes monitoring

- **Data Structure**:
  - Post ID
  - Page ID
  - User ID (for multi-tenant isolation)
  - Monitoring status (active/inactive)
  - Check interval (in minutes)
  - Last checked timestamp
  - Creation timestamp
  - Subscription tier requirements

- **Implementation Notes**:
  - Post selection and configuration happens entirely in frontend
  - Backend provides API endpoints for post data retrieval
  - Backend enforces subscription-based limits on monitored posts
  - When "Start Monitoring" button is clicked, send data to Node.js backend to begin monitoring process

### 4. Comment Monitoring System

- **Backend Implementation**:
  - Built entirely on Node.js
  - Express.js server with scheduled tasks to check for new comments
  - Triggered based on monitoring intervals set in frontend
  - Only processes comments created after post was added to monitoring
  - Monitoring frequency and volume tied to subscription tier

- **Data Flow**:
  - Backend retrieves monitored posts from Supabase based on user ID
  - Backend uses stored Facebook tokens to check for new comments via Facebook API
  - When new comments are found, they're stored in Supabase with user ID association
  - Backend sends notification/update to frontend as needed

- **Process Flow**:
  1. For each active monitored post, check for new comments using Node.js backend
  2. Store all existing comments in database when post is first connected (for baseline comparison)
  3. On subsequent checks, identify truly new comments by comparing with stored comments
  4. Store new comments in database
  5. For each new comment, trigger AI response generation via Node.js service
  6. Track usage metrics against subscription limits

- **Data Structure**:
  - Comment ID
  - Post ID
  - User ID (for multi-tenant isolation)
  - Comment text
  - Commenter name/ID
  - Creation timestamp
  - Response status (pending/completed/failed)
  - IsNew flag (to distinguish between baseline and new comments)

- **Implementation Notes**:
  - Important: Store all existing comments when a post is first connected to establish baseline
  - Only trigger responses for comments that appear after post connection
  - Use timestamp comparison for additional verification
  - All comment fetching and processing handled by Node.js backend
  - Backend provides API endpoints for frontend to check monitoring status and view comments
  - Implement usage tracking for subscription tier enforcement

### 5. Knowledge Base Management

- **User Interface**:
  - Form for entering business details (implemented in frontend)
  - Rich text editor for detailed information
  - Categories for different types of knowledge
  - Knowledge base size limits based on subscription tier

- **Data Flow**:
  - Frontend collects knowledge base information from user
  - Frontend sends data to backend via API
  - Backend validates and stores data in Supabase with user ID association
  - Backend makes knowledge base available to AI response system

- **Knowledge Categories**:
  - Business overview (description, history)
  - Products/services offered
  - Pricing information
  - Common FAQs
  - Response tone and style preferences
  - Comment language preferences

- **Data Structure**:
  - User ID (for multi-tenant isolation)
  - Knowledge category
  - Content
  - Last updated timestamp
  - Size metrics for subscription enforcement
  
- **Implementation Notes**:
  - This system must be implemented before the AI response generation
  - Knowledge base inputs should be structured for easy incorporation into AI prompts
  - Validate and sanitize inputs in both frontend and backend
  - Knowledge base data entry happens in frontend
  - Data is stored in Supabase via Node.js backend and accessed by Node.js backend when generating responses
  - Implement size limits based on subscription tier

### 6. AI Response Generation

- **Backend Implementation**:
  - Fully implemented in Node.js
  - Set up API client with proper authentication for Gemini API
  - Create system prompt template
  - Implement response generation function
  - Track API usage per user for subscription tier enforcement

- **Prompt Engineering**:
  - Base system prompt describing the task
  - Incorporation of user's knowledge base (pulled from Supabase by Node.js)
  - Context about the post and comment
  - Instructions for response tone and format

- **Response Formatting**:
  - Start with @mention of original commenter
  - Include AI-generated response
  - Ensure compliance with Facebook comment limitations

- **Implementation Notes**:
  - Include rate limiting and error handling
  - Implement retry mechanism for failed API calls
  - Store generated responses in database with user ID association
  - All AI response generation handled by Node.js services
  - Response queue system to manage high volume
  - Track API usage for subscription tier limitations

### 7. Facebook Comment Posting

- **Graph API Integration**:
  - Implemented entirely in Node.js backend
  - Authenticate with page access token
  - Use `/{comment-id}/comments` endpoint for replying
  - Implement proper error handling
  - Track API usage against subscription limits

- **Process Flow**:
  1. Generate AI response via Node.js service
  2. Format with @mention
  3. Post to Facebook using Graph API through Node.js
  4. Update database with response status
  5. Log conversation for analytics
  6. Track usage metrics against subscription tier

- **Implementation Notes**:
  - Ensure knowledge base is consulted before generating responses
  - Implement failsafes to prevent duplicate replies
  - Monitor API rate limits
  - All comment posting handled by Node.js backend
  - Implement logging system for debugging and monitoring
  - Enforce subscription tier limits

### 8. AI Conversations Analytics

- **Features**:
  - Display history of comments and AI responses
  - Filter by date range, Facebook page, or post
  - Search functionality
  - Performance metrics (response time, engagement)
  - Analytics depth based on subscription tier

- **Data Visualization**:
  - Timeline of AI interactions
  - Comment volume charts
  - Response sentiment analysis
  - Common topics/questions
  - Advanced analytics for premium users

- **Implementation Notes**:
  - Implement real-time updates when possible
  - Allow manual override of AI responses
  - Provide feedback mechanism to improve AI
  - Frontend visualization with data provided by Node.js backend APIs
  - Historical data stored and retrieved from Supabase
  - Tier-based access to advanced analytics features

### 9. Database Schema (Supabase)

- **Tables**:
  - users (with subscription_tier field)
  - subscriptions (linked to users)
  - facebook_pages
  - monitored_posts
  - comments
  - ai_responses
  - knowledge_base
  - usage_metrics

- **Relationships**:
  - User has many Facebook Pages
  - User has one Subscription
  - Facebook Page has many Monitored Posts
  - Monitored Post has many Comments
  - Comment has one AI Response
  - User has many Knowledge Base entries
  - User has many Usage Metrics entries

- **Implementation Notes**:
  - All database interactions from backend handled by Node.js services
  - Use Supabase JavaScript SDK for database operations
  - Implement proper indexing for performance optimization
  - Ensure all tables have user_id foreign key for multi-tenant isolation
  - Include subscription-related fields for tier enforcement

### 10. Backend Architecture

- **Core Components**:
  - Express.js API server for handling frontend requests
  - Node.js worker services for comment monitoring
  - Scheduled jobs for regular post checking (using node-cron or similar)
  - Queue system for handling AI response generation (Bull or similar)
  - Subscription validation middleware

- **API-Driven Architecture**:
  - Frontend makes all requests to backend API endpoints
  - Backend handles all data processing and external API calls
  - Frontend is responsible only for UI rendering and user interaction
  - Backend provides data to frontend as needed
  - Backend enforces subscription limits on all operations

- **Service Organization**:
  - Authentication Service: Handles full OAuth flow and token management
  - Subscription Service: Manages user tiers and enforces limits
  - Facebook Service: Manages Facebook API interactions
  - Monitoring Service: Handles post monitoring and comment detection
  - AI Service: Manages Gemini API integration and response generation
  - Response Service: Handles posting comments back to Facebook
  - Analytics Service: Processes and provides usage insights

- **API Endpoints**:
  - `/api/auth/facebook` - Handle Facebook OAuth flow
  - `/api/auth/callback` - Process OAuth callback and store tokens
  - `/api/subscription/status` - Check current subscription tier and limits
  - `/api/pages/list` - Get connected Facebook pages
  - `/api/posts/list` - Get posts from a connected page
  - `/api/posts/monitor` - Start monitoring selected posts
  - `/api/monitor/status` - Check monitoring status
  - `/api/knowledge/save` - Save knowledge base data
  - `/api/conversations/list` - Retrieve conversation history

- **Data Flow**:
  1. Frontend redirects to backend for Facebook authentication
  2. Backend handles entire OAuth flow and securely stores tokens
  3. Frontend requests data from backend as needed
  4. Backend validates subscription entitlements before processing
  5. Backend fetches data from Facebook and other APIs
  6. Backend processes data and sends results to frontend
  7. Backend handles all ongoing monitoring and responses

- **Implementation Notes**:
  - Deploy as serverless functions or containerized application
  - Implement proper error handling and logging
  - Design for horizontal scaling
  - Use environment variables for configuration
  - Build with subscription tier enforcement throughout

### 11. Security Considerations

- **Authentication**:
  - JWT-based auth for API requests
  - OAuth flows handled entirely by backend
  - Secure storage of Facebook access tokens with encryption
  - Implementation of token refresh and rotation

- **Data Protection**:
  - Encryption of sensitive data
  - Compliance with data protection regulations
  - Regular security audits
  - Multi-tenant isolation via user ID validation

- **API Security**:
  - Rate limiting
  - Input validation
  - CORS configuration
  - Subscription tier validation

### 12. Subscription System

- **Tier Structure** (Not implemented initially, but architecture ready):
  - Free tier: Limited monitored posts, basic analytics
  - Premium tier: More posts, faster monitoring, advanced analytics

- **Usage Tracking**:
  - Monitor number of posts being tracked
  - Track number of AI responses generated
  - Measure knowledge base size
  - Log API call frequency

- **Enforcement Points**:
  - Post selection (limit by tier)
  - Monitoring frequency (limit by tier)
  - AI response generation (limit by tier)
  - Analytics features (limit by tier)

- **Implementation Notes**:
  - Build with subscription awareness from the beginning
  - Create middleware for subscription validation
  - Include subscription fields in database schema
  - Design UI with subscription-tier visibility

## Implementation Timeline

### Phase 1: Foundation (Weeks 1-2)
- User authentication system with Supabase Auth
- Backend-managed Facebook OAuth integration
- Basic dashboard structure
- Database schema setup with subscription readiness

### Phase 2: Core Functionality (Weeks 3-4)
- Posts management implementation
- Comment monitoring system
- Basic AI response generation
- Facebook comment posting

### Phase 3: Enhancement (Weeks 5-6)
- Knowledge base management
- Improved AI prompting
- Conversation analytics
- UI/UX refinements
- Basic subscription structure

### Phase 4: Testing & Optimization (Weeks 7-8)
- End-to-end testing
- Performance optimization
- Security review
- Bug fixes
- Final subscription-tier validations

## Technical Stack

- **Frontend**:
  - React.js
  - Tailwind CSS or Material UI
  - Context API or Redux for state management

- **Backend**:
  - Node.js/Express.js
  - Serverless functions (Vercel, AWS Lambda)
  - Cron jobs for scheduled tasks

- **Database**:
  - Supabase (PostgreSQL)

- **Authentication**:
  - Supabase Auth (email, magic links, Google OAuth)
  - Custom backend-managed Facebook OAuth

- **External APIs**:
  - Facebook Graph API
  - Gemini API

## Success Criteria

- Secure authentication including backend-managed Facebook OAuth
- Successful Facebook page connection with secure token management
- Accurate monitoring of selected posts
- Timely detection of new comments
- Natural-sounding AI responses
- Proper @mention formatting
- Reliable posting of responses via Graph API
- Comprehensive analytics for users
- Architecture ready for future subscription tiers

As you work on implementing Botcart, please refer back to this document frequently to ensure all components are developed according to specification. Feel free to suggest optimizations or alternative approaches where beneficial.