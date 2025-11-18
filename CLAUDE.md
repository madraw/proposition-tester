# Proposition Tester - InterviewKickstart Non-Tech AI Business

**Last Updated:** November 19, 2025

## Project Overview

This is a landing page infrastructure for testing value propositions for InterviewKickstart's non-tech AI business (AI Marketing Accelerator program). The system enables rapid A/B testing of different value propositions, pricing models, and audience segments to find product-market fit before scaling marketing spend.

### Business Context

**Program:** AI Marketing Accelerator - 6-7 week program teaching non-technical professionals to build AI agents and workflows without coding.

**Target Audience:**
- Career transitioners (10+ years experience, laid off or pivoting)
- Momentum professionals (upwardly mobile, high-agency individuals)
- Solo entrepreneurs/freelancers building agencies
- Geographic focus: US (80% EST, 20% PST)

**Value Propositions Being Tested:**
1. "Become an AI Builder Without Coding" (primary - removes #1 barrier)
2. "Accelerate Your AI Journey" (compress learning into intensive program)
3. "Transform Your Career with AI" (career advancement positioning)
4. "Lead with AI Without Knowing How to Code" (for senior leaders)
5. "Build Skills to Get a Better Job" (direct career outcomes)

**Pricing Tests:**
- Current: $4,000-$5,000 (6-week program)
- Accelerator model: $999-$1,999 (3-4 day intensive)
- Formats: Full-time (3-4 days) / Part-time (2 weekends) / Semi part-time (1 month)

**Conversion Goal:** Book calendar appointment with program advisor on Google Calendar

## User Journey

```
Ad Impression (LinkedIn/Meta/Google/Reddit)
    ↓
User Clicks Ad (UTM tracked)
    ↓
Landing Page (Dynamic content based on UTM parameters)
    ↓
User Fills Form & Books Calendar Appointment
    ↓
Embedded Google Calendar Selection
    ↓
Confirmation & Analytics Event Tracking
```

## Technical Architecture

### Stack Overview

**Frontend:**
- Static HTML/CSS/JavaScript (no framework needed for speed)
- Responsive design (mobile-first)
- BEM CSS methodology for maintainability

**Backend:**
- Node.js/Express server
- PostgreSQL database for content management
- UTM parameter parsing and content routing

**Hosting:**
- VPS deployment (separate from main IK infrastructure)
- Multiple domains for different test variants
- SSL/TLS encryption

**Analytics:**
- Google Tag Manager (GTM) as container
- Google Analytics 4 (GA4)
- Meta Pixel (Facebook/Instagram)
- LinkedIn Insight Tag
- Reddit Pixel (if needed)

**Calendar Integration:**
- Google Calendar API
- Embedded calendar booking widget
- Automated confirmation emails

### Database Schema

```sql
-- UTM Campaign Content Mapping
CREATE TABLE landing_page_content (
    id SERIAL PRIMARY KEY,

    -- UTM Parameters (composite key for content lookup)
    utm_campaign VARCHAR(255) NOT NULL,
    utm_source VARCHAR(100),
    utm_medium VARCHAR(100),
    utm_content VARCHAR(255),
    utm_term VARCHAR(255),

    -- Proposition Details
    proposition_name VARCHAR(255) NOT NULL,
    proposition_variant VARCHAR(100), -- e.g., "v1", "v2" for A/B testing

    -- Pricing & Format
    price_point INTEGER, -- e.g., 999, 1499, 1999
    program_format VARCHAR(100), -- "full-time", "part-time", "semi-part-time"
    program_duration VARCHAR(100), -- "3-days", "2-weekends", "1-month"

    -- Page Content (JSON for flexibility)
    hero_section JSONB NOT NULL,
    process_section JSONB,
    trust_section JSONB,
    faq_section JSONB,
    cta_section JSONB,
    footer_section JSONB,

    -- Styling Configuration
    styling JSONB,

    -- Meta Data
    page_title VARCHAR(255),
    meta_description TEXT,

    -- Status & Version Control
    is_active BOOLEAN DEFAULT true,
    version INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    -- Indexes for fast lookup
    CONSTRAINT unique_utm_combo UNIQUE (utm_campaign, utm_source, utm_medium, utm_content, utm_term)
);

CREATE INDEX idx_utm_campaign ON landing_page_content(utm_campaign);
CREATE INDEX idx_active_content ON landing_page_content(is_active) WHERE is_active = true;

-- Form Submissions & Calendar Bookings
CREATE TABLE form_submissions (
    id SERIAL PRIMARY KEY,

    -- User Information
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL,
    phone VARCHAR(50),

    -- Professional Context
    current_role VARCHAR(255),
    company VARCHAR(255),
    years_experience INTEGER,
    linkedin_url VARCHAR(500),

    -- UTM Tracking (what brought them here)
    utm_campaign VARCHAR(255),
    utm_source VARCHAR(100),
    utm_medium VARCHAR(100),
    utm_content VARCHAR(255),
    utm_term VARCHAR(255),

    -- Landing Page Context
    landing_page_id INTEGER REFERENCES landing_page_content(id),
    proposition_shown VARCHAR(255),
    price_shown INTEGER,

    -- Calendar Booking
    calendar_event_id VARCHAR(255), -- Google Calendar event ID
    appointment_time TIMESTAMP,
    advisor_email VARCHAR(255),
    booking_confirmed BOOLEAN DEFAULT false,

    -- Session Data
    session_id VARCHAR(255),
    ip_address VARCHAR(45),
    user_agent TEXT,
    referrer TEXT,

    -- Timing Metrics
    time_on_page INTEGER, -- seconds
    form_start_time TIMESTAMP,
    form_submit_time TIMESTAMP,

    -- Status
    status VARCHAR(50) DEFAULT 'submitted', -- submitted, confirmed, attended, no-show
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_email ON form_submissions(email);
CREATE INDEX idx_utm_campaign_sub ON form_submissions(utm_campaign);
CREATE INDEX idx_created_at ON form_submissions(created_at);
CREATE INDEX idx_appointment_time ON form_submissions(appointment_time);

-- Analytics Events
CREATE TABLE analytics_events (
    id SERIAL PRIMARY KEY,

    -- Session Context
    session_id VARCHAR(255) NOT NULL,
    user_id VARCHAR(255), -- if identified

    -- Event Data
    event_name VARCHAR(100) NOT NULL,
    event_category VARCHAR(100),
    event_label VARCHAR(255),
    event_value NUMERIC,

    -- Page Context
    page_url TEXT,
    page_title VARCHAR(255),
    landing_page_id INTEGER REFERENCES landing_page_content(id),

    -- UTM Parameters
    utm_campaign VARCHAR(255),
    utm_source VARCHAR(100),
    utm_medium VARCHAR(100),
    utm_content VARCHAR(255),
    utm_term VARCHAR(255),

    -- Additional Data
    event_data JSONB,

    -- Metadata
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address VARCHAR(45),
    user_agent TEXT
);

CREATE INDEX idx_event_name ON analytics_events(event_name);
CREATE INDEX idx_session_id ON analytics_events(session_id);
CREATE INDEX idx_timestamp ON analytics_events(timestamp);

-- A/B Test Performance Tracking
CREATE TABLE variant_performance (
    id SERIAL PRIMARY KEY,

    -- Test Configuration
    utm_campaign VARCHAR(255) NOT NULL,
    proposition_variant VARCHAR(100) NOT NULL,

    -- Daily Metrics
    date DATE NOT NULL,

    -- Traffic Metrics
    impressions INTEGER DEFAULT 0,
    clicks INTEGER DEFAULT 0,
    landing_page_views INTEGER DEFAULT 0,

    -- Engagement Metrics
    avg_time_on_page NUMERIC,
    bounce_rate NUMERIC,
    scroll_depth_avg NUMERIC,

    -- Conversion Metrics
    form_starts INTEGER DEFAULT 0,
    form_submissions INTEGER DEFAULT 0,
    calendar_bookings INTEGER DEFAULT 0,

    -- Ad Spend
    ad_spend NUMERIC DEFAULT 0,

    -- Calculated Rates
    ctr NUMERIC GENERATED ALWAYS AS (
        CASE WHEN impressions > 0 THEN (clicks::NUMERIC / impressions::NUMERIC) * 100 ELSE 0 END
    ) STORED,
    conversion_rate NUMERIC GENERATED ALWAYS AS (
        CASE WHEN landing_page_views > 0 THEN (calendar_bookings::NUMERIC / landing_page_views::NUMERIC) * 100 ELSE 0 END
    ) STORED,
    cost_per_booking NUMERIC GENERATED ALWAYS AS (
        CASE WHEN calendar_bookings > 0 THEN ad_spend / calendar_bookings ELSE 0 END
    ) STORED,

    -- Metadata
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT unique_variant_date UNIQUE (utm_campaign, proposition_variant, date)
);

CREATE INDEX idx_campaign_date ON variant_performance(utm_campaign, date);
```

### Content Configuration Structure (JSONB)

```javascript
// Example landing_page_content.hero_section JSONB structure
{
  "headline": "Become an AI Builder Without Coding",
  "subheadline": "Transform from zero AI knowledge to building AI agents in 6 weeks. No technical background required.",
  "benefits": [
    "Build AI agents without writing a single line of code",
    "Go from beginner to deploying real marketing automation",
    "Join 6-week live cohort with hands-on projects",
    "Get portfolio-ready projects for job interviews"
  ],
  "cta": {
    "text": "Book Free Strategy Call",
    "style": {
      "backgroundColor": "#1aaa55",
      "textColor": "#ffffff",
      "borderRadius": "8px",
      "fontSize": "1.125rem",
      "padding": "0.875rem 2rem"
    }
  },
  "styling": {
    "background": {
      "type": "gradient",
      "direction": "135deg",
      "colors": ["#f8f9fa", "#ffffff"]
    },
    "textColor": "#2c3e50",
    "headlineSize": {
      "desktop": "3rem",
      "tablet": "2.5rem",
      "mobile": "2rem"
    }
  },
  "trustSignal": {
    "enabled": true,
    "text": "Join 100+ professionals who've transformed their careers",
    "type": "social-proof"
  }
}

// Example landing_page_content.process_section JSONB structure
{
  "sectionTitle": "Your Journey to AI Mastery",
  "subtitle": "A proven path from complete beginner to confident AI builder",
  "steps": [
    {
      "number": 1,
      "title": "Book Strategy Call",
      "description": "Discuss your goals with our program advisor and see if you're a fit",
      "icon": { "type": "emoji", "value": "📅" },
      "color": "#1aaa55"
    },
    {
      "number": 2,
      "title": "Join Live Cohort",
      "description": "6-week intensive program with hands-on projects from day 1",
      "icon": { "type": "emoji", "value": "🚀" },
      "color": "#4fc3f7"
    },
    {
      "number": 3,
      "title": "Build AI Agents",
      "description": "Create real marketing automation and AI workflows for your portfolio",
      "icon": { "type": "emoji", "value": "🤖" },
      "color": "#ff6900"
    },
    {
      "number": 4,
      "title": "Transform Your Career",
      "description": "Get promoted, switch careers, or start your AI consulting practice",
      "icon": { "type": "emoji", "value": "✨" },
      "color": "#1aaa55"
    }
  ],
  "guarantee": {
    "title": "100% Money-Back Guarantee",
    "text": "If you attend all sessions and complete the projects but don't feel you've gained valuable AI skills, we'll refund your investment in full.",
    "style": {
      "backgroundColor": "#f1f3f4",
      "borderColor": "#1aaa55",
      "textColor": "#2c3e50"
    }
  },
  "cta": {
    "text": "Start Your AI Journey",
    "style": {
      "backgroundColor": "#1aaa55",
      "textColor": "#ffffff",
      "borderRadius": "8px"
    }
  }
}

// Example landing_page_content.pricing (within hero or dedicated section)
{
  "pricePoint": 1499,
  "currency": "USD",
  "displayFormat": "$1,499",
  "frequency": "one-time",
  "strikethroughPrice": 4999,
  "savingsMessage": "Save $3,500 - Limited Time Accelerator Pricing",
  "paymentOptions": [
    "Pay in full",
    "3-month payment plan available"
  ],
  "valueProps": [
    "6 weeks of live instruction",
    "Lifetime access to course materials",
    "Private community access",
    "Career mentoring included"
  ]
}
```

## Key Features

### 1. UTM-Based Dynamic Content Routing

**Flow:**
1. User clicks ad with UTM parameters (e.g., `?utm_campaign=ai-builder-no-code&utm_source=linkedin&utm_medium=cpc&utm_content=variant-a`)
2. Server parses UTM parameters
3. Database lookup to find matching content configuration
4. Dynamically render landing page with appropriate content
5. Track all UTM parameters through conversion funnel

**Example URL:**
```
https://ai-accelerator-test1.com/?utm_campaign=no-code-builder&utm_source=meta&utm_medium=cpc&utm_content=headline-v2&utm_term=ai-marketing
```

### 2. A/B Testing Infrastructure

**Testing Dimensions:**
- Value propositions (5+ variants)
- Headlines and copy
- Pricing ($999/$1,499/$1,999)
- Program format (full-time/part-time/semi-part-time)
- CTA button text and design
- Hero images/videos
- Social proof elements

**Testing Process:**
1. Create multiple content entries in database with different UTM parameter combinations
2. Set up separate ad campaigns for each variant
3. Track performance metrics in `variant_performance` table
4. Weekly review: Eliminate bottom 2 performers, add 2 new tests
5. Iterate until finding winning combination

### 3. Analytics Integration

**Google Tag Manager Setup:**
```javascript
// GTM Container with all tracking pixels
window.dataLayer = window.dataLayer || [];

// Page View Event (with UTM parameters)
dataLayer.push({
  'event': 'page_view',
  'page_title': document.title,
  'page_path': window.location.pathname + window.location.search,
  'utm_campaign': getUTMParam('utm_campaign'),
  'utm_source': getUTMParam('utm_source'),
  'utm_medium': getUTMParam('utm_medium'),
  'utm_content': getUTMParam('utm_content'),
  'utm_term': getUTMParam('utm_term'),
  'proposition_variant': '<%= propositionVariant %>',
  'price_shown': <%= priceShown %>
});

// Form Start Event
dataLayer.push({
  'event': 'form_start',
  'form_name': 'calendar_booking',
  'utm_campaign': getUTMParam('utm_campaign')
});

// Form Submit Event
dataLayer.push({
  'event': 'form_submit',
  'form_name': 'calendar_booking',
  'conversion_value': <%= priceShown %>
});

// Calendar Booking Confirmed
dataLayer.push({
  'event': 'booking_confirmed',
  'appointment_time': appointmentTime,
  'conversion_value': <%= priceShown %>
});
```

**GA4 Custom Events:**
- `page_view` (with UTM dimensions)
- `form_start`
- `form_field_completed` (track field-by-field)
- `form_submit`
- `calendar_opened`
- `calendar_date_selected`
- `booking_confirmed`
- `scroll_depth` (25%, 50%, 75%, 100%)
- `cta_click` (track all CTA button clicks)

**Meta Pixel Events:**
```javascript
fbq('track', 'PageView');
fbq('track', 'Lead'); // on form submit
fbq('track', 'Schedule'); // on calendar booking
fbq('trackCustom', 'FormStart');
```

**LinkedIn Insight Tag:**
```javascript
window._linkedin_data_partner_ids = window._linkedin_data_partner_ids || [];
window._linkedin_data_partner_ids.push(_linkedin_partner_id);

// Track conversion
lintrk('track', { conversion_id: CONVERSION_ID });
```

### 4. Calendar Integration

**Google Calendar API Setup:**
1. Service account for server-side calendar management
2. Shared calendar for program advisors
3. Automated slot availability checking
4. Conflict prevention

**Booking Flow:**
```javascript
// 1. Fetch available time slots
GET /api/calendar/available-slots
Response: {
  slots: [
    { start: "2025-11-20T10:00:00Z", end: "2025-11-20T10:30:00Z" },
    { start: "2025-11-20T14:00:00Z", end: "2025-11-20T14:30:00Z" }
  ]
}

// 2. User selects time and submits form
POST /api/booking/create
Body: {
  firstName: "John",
  lastName: "Doe",
  email: "john@example.com",
  phone: "+1234567890",
  selectedSlot: "2025-11-20T10:00:00Z",
  utmParams: {...},
  currentRole: "Marketing Manager",
  yearsExperience: 12
}

// 3. Server creates calendar event
Response: {
  success: true,
  eventId: "google-calendar-event-id",
  confirmationUrl: "/confirmation?booking=xxx"
}

// 4. Send confirmation email (via SendGrid/AWS SES)
// 5. Add to CRM (if integrated)
```

### 5. Security & Performance

**Security Measures:**
- SSL/TLS encryption (Let's Encrypt)
- Environment variables for sensitive data
- Input validation and sanitization
- SQL injection prevention (parameterized queries)
- Rate limiting on form submissions
- CAPTCHA on form (optional, monitor spam first)
- CORS configuration
- CSP headers

**Performance Optimization:**
- CDN for static assets (Cloudflare)
- Image optimization (WebP with fallbacks)
- Lazy loading for below-fold content
- Minified CSS/JS
- Gzip compression
- Database query optimization (indexed lookups)
- Caching strategy (Redis for frequent queries)
- Lighthouse score target: 90+ on mobile

### 6. Content Management

**Approach 1: Database-Driven (Recommended)**
- All content in PostgreSQL JSONB columns
- Easy A/B testing via database updates
- No code changes needed for content updates
- Admin UI for non-technical content updates (future phase)

**Approach 2: Configuration Files (Alternative)**
- Content in JSON/YAML files
- Version controlled via Git
- Requires developer for changes
- Better for initial setup, harder for rapid iteration

**Content Update Workflow:**
```sql
-- Insert new proposition variant
INSERT INTO landing_page_content (
  utm_campaign,
  utm_source,
  utm_medium,
  utm_content,
  proposition_name,
  proposition_variant,
  price_point,
  hero_section,
  process_section,
  page_title,
  meta_description
) VALUES (
  'ai-builder-career-transform',
  'linkedin',
  'cpc',
  'headline-variant-b',
  'Career Transformation',
  'v2',
  1499,
  '{"headline": "Transform Your Career...", ...}'::jsonb,
  '{"sectionTitle": "Your Path...", ...}'::jsonb,
  'Transform Your Career with AI - No Coding Required',
  'Learn to build AI agents in 6 weeks. Join live cohort. No technical background needed.'
);

-- Update existing variant (A/B test iteration)
UPDATE landing_page_content
SET hero_section = jsonb_set(
  hero_section,
  '{headline}',
  '"NEW HEADLINE HERE"'
)
WHERE utm_campaign = 'ai-builder-career-transform'
  AND utm_content = 'headline-variant-b';
```

## Project Structure

```
proposition-tester/
├── client/                      # Frontend code
│   ├── css/
│   │   ├── base.css            # Reset, typography, variables
│   │   ├── components/         # BEM components
│   │   │   ├── header.css
│   │   │   ├── hero.css
│   │   │   ├── process.css
│   │   │   ├── form.css
│   │   │   ├── calendar.css
│   │   │   └── footer.css
│   │   └── main.css            # Import all styles
│   ├── js/
│   │   ├── analytics.js        # GTM, GA4, Meta Pixel setup
│   │   ├── form-handler.js     # Form validation and submission
│   │   ├── calendar.js         # Calendar widget integration
│   │   ├── utm-tracker.js      # UTM parameter parsing and storage
│   │   └── main.js             # App initialization
│   ├── images/                 # Optimized images
│   └── index.html              # Template (server-rendered)
│
├── server/                      # Backend code
│   ├── config/
│   │   ├── database.js         # PostgreSQL connection
│   │   ├── google-calendar.js  # Google Calendar API setup
│   │   └── environment.js      # Environment variables
│   ├── models/
│   │   ├── landing-page.js     # Landing page content model
│   │   ├── form-submission.js  # Form submission model
│   │   └── analytics.js        # Analytics events model
│   ├── controllers/
│   │   ├── page-controller.js  # Landing page rendering
│   │   ├── booking-controller.js # Calendar booking logic
│   │   └── analytics-controller.js # Event tracking
│   ├── routes/
│   │   ├── pages.js            # Page routes
│   │   ├── api.js              # API endpoints
│   │   └── webhooks.js         # Calendar webhooks
│   ├── middleware/
│   │   ├── utm-parser.js       # Parse and validate UTM params
│   │   ├── rate-limiter.js     # Rate limiting
│   │   └── error-handler.js    # Error handling
│   ├── services/
│   │   ├── content-service.js  # Content lookup from DB
│   │   ├── calendar-service.js # Google Calendar integration
│   │   ├── email-service.js    # Confirmation emails
│   │   └── analytics-service.js # Analytics event logging
│   ├── utils/
│   │   ├── validators.js       # Input validation
│   │   └── formatters.js       # Data formatting
│   └── server.js               # Express app setup
│
├── database/
│   ├── migrations/             # Database migrations
│   │   ├── 001_initial_schema.sql
│   │   ├── 002_add_analytics_tables.sql
│   │   └── 003_add_variant_performance.sql
│   ├── seeds/                  # Seed data for testing
│   │   └── sample_content.sql
│   └── schema.sql              # Full schema definition
│
├── scripts/
│   ├── deploy.sh               # VPS deployment script
│   ├── backup-db.sh            # Database backup
│   └── seed-content.sh         # Content seeding
│
├── docs/
│   ├── API.md                  # API documentation
│   ├── DEPLOYMENT.md           # Deployment guide
│   └── TESTING.md              # A/B testing guide
│
├── knowledge/                   # Project context
│   ├── reference/
│   │   ├── landing-page-checklist.md
│   │   ├── 251118-ik-non-tech-ai-business-overview.md
│   │   └── 251118-ryan-meeting-summary.md
│   └── design/                 # Design specifications (TBD)
│
├── .env.example                # Environment variables template
├── .gitignore
├── package.json
├── README.md
└── CLAUDE.md                   # This file
```

## Environment Variables

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/proposition_tester
DATABASE_POOL_SIZE=10

# Server
NODE_ENV=production
PORT=3000
DOMAIN=https://ai-accelerator-test1.com

# Google Calendar API
GOOGLE_CALENDAR_CLIENT_EMAIL=service-account@project.iam.gserviceaccount.com
GOOGLE_CALENDAR_PRIVATE_KEY=-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----
GOOGLE_CALENDAR_ID=calendar-id@group.calendar.google.com

# Email Service (SendGrid or AWS SES)
SENDGRID_API_KEY=SG.xxxxx
FROM_EMAIL=advisor@interviewkickstart.com
FROM_NAME=IK Program Advisor

# Analytics
GTM_CONTAINER_ID=GTM-XXXXXX
GA4_MEASUREMENT_ID=G-XXXXXXXXXX
META_PIXEL_ID=123456789012345
LINKEDIN_PARTNER_ID=1234567

# Security
SESSION_SECRET=random-secure-string-here
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# Optional
RECAPTCHA_SITE_KEY=6Lxxxxxx
RECAPTCHA_SECRET_KEY=6Lxxxxxx
SENTRY_DSN=https://xxx@sentry.io/xxx
```

## API Endpoints

### Public Endpoints

```
GET  /
  - Landing page (UTM parameters in query string)
  - Returns: Rendered HTML with dynamic content

GET  /api/calendar/available-slots
  - Query params: ?timezone=America/New_York&days=7
  - Returns: Available time slots for booking

POST /api/booking/create
  - Body: { firstName, lastName, email, phone, selectedSlot, utmParams, ... }
  - Returns: { success, eventId, confirmationUrl }

GET  /confirmation
  - Query params: ?booking=xxx
  - Returns: Confirmation page

POST /api/analytics/track
  - Body: { event, eventData, sessionId, utmParams }
  - Returns: { success }
```

### Admin Endpoints (Future Phase)

```
GET  /admin/dashboard
  - Analytics dashboard

GET  /admin/content
  - Content management UI

POST /admin/content/create
  - Create new variant

PUT  /admin/content/:id
  - Update existing variant

GET  /admin/performance
  - A/B test performance metrics
```

## Deployment Strategy

### VPS Setup (e.g., DigitalOcean, AWS EC2, Linode)

**Infrastructure:**
- 2 GB RAM minimum (4 GB recommended)
- Ubuntu 22.04 LTS
- PostgreSQL 15
- Node.js 20 LTS
- Nginx as reverse proxy
- SSL via Let's Encrypt

**Deployment Process:**
1. Provision VPS
2. Install dependencies (Node.js, PostgreSQL, Nginx)
3. Clone repository
4. Set up environment variables
5. Run database migrations
6. Seed initial content
7. Build frontend assets
8. Configure Nginx
9. Set up SSL certificates
10. Start Node.js app (PM2 process manager)
11. Configure monitoring (optional: Sentry, LogRocket)

**Multiple Domains for Testing:**
- `ai-accelerator-test1.com` - Proposition A
- `ai-accelerator-test2.com` - Proposition B
- `ai-accelerator-test3.com` - Proposition C
- Each domain points to same VPS
- Nginx routes based on domain + UTM parameters

## Success Metrics

### Primary KPIs
1. **Landing Page CTR**: 5-20% registration rate target
2. **Cost Per Booking**: Track across all variants
3. **Calendar Show Rate**: 80% target (4/5 bookings show up)
4. **Time to Conversion**: Speed from click to booking

### Testing Metrics
- Impressions per variant
- Clicks per variant
- Bounce rate
- Average time on page
- Scroll depth
- Form start rate
- Form completion rate
- Calendar booking rate

### Weekly Review Process
1. Pull performance data from `variant_performance` table
2. Calculate statistical significance
3. Eliminate bottom 2 performers
4. Create 2 new test variants
5. Update UTM campaign mapping
6. Launch new ad campaigns

## Development Phases

### Phase 1: MVP (Week 1-2)
- [ ] Database schema setup
- [ ] Basic server with UTM parsing
- [ ] Single landing page template
- [ ] Form submission (no calendar yet)
- [ ] GTM + GA4 integration
- [ ] VPS deployment
- [ ] Content seeding for 5 initial propositions

### Phase 2: Calendar Integration (Week 3)
- [ ] Google Calendar API setup
- [ ] Available slots fetching
- [ ] Booking creation
- [ ] Confirmation emails
- [ ] Webhook for calendar updates

### Phase 3: Analytics Enhancement (Week 4)
- [ ] Meta Pixel integration
- [ ] LinkedIn Insight Tag
- [ ] Event tracking refinement
- [ ] Performance dashboard queries
- [ ] A/B test reporting

### Phase 4: Optimization (Week 5-6)
- [ ] Performance tuning
- [ ] Security hardening
- [ ] Admin UI for content updates
- [ ] Automated testing
- [ ] Monitoring and alerting

### Phase 5: Scale (Ongoing)
- [ ] Multi-domain setup
- [ ] Advanced A/B testing
- [ ] CRM integration
- [ ] Automated reporting
- [ ] Continuous iteration

## Design Principles (from spec.md reference)

### Basecamp-Inspired Aesthetic
- Clean, friendly, approachable design
- Rounded corners (8px border radius)
- Soft shadows (subtle depth)
- Warm color accents
- Generous padding and whitespace

### Typography
- **Font**: Inter from Google Fonts (400, 500, 600 weights)
- **Fallback**: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif
- **Responsive sizing**: Desktop/Tablet/Mobile variants

### Color Palette (Configurable per variant)
- **Primary**: Warm green (#1aaa55) for CTAs
- **Secondary**: Soft blue (#4fc3f7) for accents
- **Highlight**: Warm orange (#ff6900) for urgency
- **Neutrals**: Soft grays (#f1f3f4, #6f7782)
- **Background**: Cream/off-white (#fefefe, #f8f9fa)

### CSS Methodology: BEM
```css
/* Block */
.hero { }

/* Element */
.hero__headline { }
.hero__subheadline { }
.hero__cta-button { }

/* Modifier */
.hero--dark-theme { }
.hero__cta-button--primary { }
.hero__cta-button--secondary { }
```

## Key Learnings from Reference Project

### What to Copy from Newsletter Deals Project

1. **Configuration-Driven Content**
   - All content in database/config files
   - No hardcoded copy in HTML
   - Easy A/B testing via config changes

2. **Responsive Design System**
   - Mobile-first approach
   - CSS custom properties for theming
   - Smooth transitions across breakpoints

3. **Analytics Foundation**
   - GTM as single integration point
   - Comprehensive event tracking
   - UTM parameter preservation

4. **Form Best Practices**
   - Progressive disclosure (multi-step if needed)
   - Inline validation
   - Clear error messages
   - Auto-complete enabled

5. **Performance Patterns**
   - Lazy loading
   - Image optimization
   - Minimal dependencies
   - Fast initial load

6. **Security Baseline**
   - Input sanitization
   - Rate limiting
   - Environment variable management
   - HTTPS enforcement

### What NOT to Copy
- Newsletter-specific business logic
- Payment processing (not needed here)
- Newsletter icon scrolling (different social proof approach)

## Critical Constraints

### Must Avoid
1. **Cannibalization**: This infrastructure is separate from main IK business
2. **Dependencies**: No reliance on main product team
3. **Complexity**: Keep it simple for rapid iteration
4. **Over-engineering**: Start with database-driven content, add admin UI only if needed

### Must Include
1. **UTM Tracking**: Every click must be traceable to source
2. **Fast Iteration**: Content changes should take minutes, not days
3. **Clear Analytics**: Know exactly which proposition is winning
4. **Scalability**: Support 5-20 simultaneous test variants

## Next Steps

1. **Review this specification** with stakeholders
2. **Provide design specification** (design/spec.md file)
3. **Set up development environment**
4. **Create initial database schema**
5. **Build MVP landing page**
6. **Seed 5 initial propositions**
7. **Launch first test campaigns**
8. **Weekly review and iteration**

## Questions for Stakeholder

1. Google Calendar details:
   - Which calendar should be used?
   - Appointment duration? (30 min suggested)
   - Timezone handling preferences?
   - Multiple advisors or single calendar?

2. Domain strategy:
   - Purchase separate domains or use subdomains?
   - Preferred naming convention?

3. CRM integration:
   - Need to sync with existing CRM?
   - HubSpot/Salesforce/other?

4. Email service:
   - SendGrid vs AWS SES preference?
   - Existing email templates?

5. Initial test variants:
   - Which 5 propositions to start with?
   - Price points for each?
   - Ad budget per variant?

---

**Last Updated**: November 19, 2025
**Status**: Specification Complete - Awaiting Design Spec
**Next Milestone**: MVP Development Start
