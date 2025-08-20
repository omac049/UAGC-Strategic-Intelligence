# Technical Architecture Overview

## Purpose
This document provides the master technical architecture overview for the UAGC Cookie Personalization system. It serves as the central reference point for all technical teams implementing the system.

## System Architecture

### High-Level Architecture
The UAGC Cookie Personalization system follows a **client-side first architecture** with cross-domain synchronization capabilities:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   uagc.edu      │    │ cloud.mail.     │    │ portal.uagc.edu │
│   (Marketing)   │◄──►│ uagc.edu        │◄──►│ (Application)   │
│                 │    │ (Stealth Apps)  │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 ▼
                    ┌─────────────────────────┐
                    │   Unified Cookie        │
                    │   Management Layer      │
                    │   (Cross-Domain Sync)   │
                    └─────────────────────────┘
                                 │
                    ┌─────────────────────────┐
                    │   Analytics & CRM       │
                    │   Integration Layer     │
                    │   (GA4, Salesforce)     │
                    └─────────────────────────┘
```

### Technology Stack

#### Frontend Framework
- **Bootstrap 5.3.2** - Responsive design framework with UAGC custom components
- **Vanilla JavaScript ES6+** - No external dependencies for core functionality
- **CSS Grid & Flexbox** - Modern layout systems for responsive design
- **PostMessage API** - Cross-domain communication and synchronization

#### Data Management Layer
- **First-Party Cookies** - Primary visitor tracking with 4 compliance categories
- **Local Storage API** - User preferences and persistent session data
- **Session Storage** - Temporary data and form state management
- **Cross-Domain Sync** - Real-time visitor status synchronization

#### Analytics & Integration
- **Google Analytics 4** - Enhanced conversions and event tracking
- **Custom dataLayer** - Structured event data for analytics platforms
- **Lead API Integration** - Workato → Salesforce pipeline
- **OneTrust Integration** - Enterprise cookie consent management

## Core Components

### 1. Cookie Management System
**File Reference:** [cookie-architecture.md](./cookie-architecture.md)

```javascript
// Core cookie categories (GDPR/CCPA compliant)
const COOKIE_CATEGORIES = {
    ESSENTIAL: 'essential',        // No consent required
    ANALYTICS: 'analytics',        // Journey tracking & performance
    PERSONALIZATION: 'personalization', // Content customization
    MARKETING: 'marketing'         // Campaign attribution
};
```

**Key Features:**
- Granular category-based consent management
- Cross-domain synchronization across all UAGC properties
- Automatic expiration and cleanup policies
- Real-time consent status updates

### 2. User Journey Tracking
**File Reference:** [event-tracking.md](./event-tracking.md)

```javascript
// Journey stage progression tracking
const JOURNEY_STAGES = {
    AWARENESS: 'awareness',        // Initial visit, content browsing
    CONSIDERATION: 'consideration', // Program exploration, return visits
    ENGAGEMENT: 'engagement',      // RFI submission, advisor contact
    APPLICATION: 'application'     // Application start/completion
};
```

**Implementation:**
- Behavioral trigger detection
- Automatic stage progression logic
- GA4 event integration
- CRM synchronization

### 3. Personalization Engine
**File Reference:** [code-examples.md](./code-examples.md)

```javascript
// Dynamic content personalization
class UAGCPersonalizationEngine {
    personalizeContent(visitorProfile, pageContext) {
        // Implements 6 visitor profile scenarios
        // Returns personalized content variations
    }
    
    trackEngagement(interactionData) {
        // Records user interactions for optimization
    }
}
```

**Capabilities:**
- 6 distinct visitor profile scenarios
- Dynamic CTA optimization
- Content variation testing
- Real-time A/B testing

### 4. Cross-Domain Synchronization
**File Reference:** [cross-domain-sync.md](./cross-domain-sync.md)

```javascript
// Multi-domain visitor state sync
window.addEventListener('message', (event) => {
    if (event.origin === 'https://uagc.edu' || 
        event.origin === 'https://cloud.mail.uagc.edu' ||
        event.origin === 'https://portal.uagc.edu') {
        // Process cross-domain visitor data
        syncVisitorState(event.data);
    }
});
```

**Features:**
- Real-time visitor status synchronization
- Secure cross-origin communication
- Persistent journey state across domains
- Automatic failover and recovery

## Performance Specifications

### Loading Performance Targets
- **First Contentful Paint:** < 1.5s
- **Largest Contentful Paint:** < 2.5s  
- **Cumulative Layout Shift:** < 0.1
- **First Input Delay:** < 100ms

### Runtime Performance
- **Smooth Scrolling:** 60fps navigation experience
- **Interactive Response:** < 50ms interaction delays
- **Memory Usage:** Optimized DOM manipulation
- **Network Efficiency:** Minimal HTTP requests

### Optimization Techniques
- Debounced scroll events for navigation highlighting
- Lazy loading for content sections
- Efficient DOM manipulation with minimal reflows
- Event delegation for optimized event handling

## Security & Compliance Framework

### Privacy Compliance
- **GDPR Article 6 Compliance** - Legal basis documentation and user rights
- **CCPA Compliance** - California Consumer Privacy Act adherence  
- **FERPA Compliance** - Educational records protection
- **COPPA Safe Harbor** - Under-13 data protection protocols

### Security Measures
- **XSS Protection** - Input sanitization and output encoding
- **CSRF Prevention** - Token-based request validation
- **Content Security Policy** - Restricted resource loading
- **Data Encryption** - Sensitive data protection for PII

### Cookie Security
```javascript
// Secure cookie configuration
const cookieConfig = {
    secure: true,          // HTTPS only
    sameSite: 'Strict',    // CSRF protection
    httpOnly: false,       // Client-side access needed
    domain: '.uagc.edu',   // Cross-subdomain sharing
    path: '/'              // Site-wide availability
};
```

## Browser Compatibility Matrix

| Feature | Chrome 90+ | Firefox 88+ | Safari 14+ | Edge 90+ |
|---------|------------|-------------|------------|----------|
| Cookie Management | ✅ | ✅ | ✅ | ✅ |
| Cross-Domain Sync | ✅ | ✅ | ✅ | ✅ |
| Local Storage | ✅ | ✅ | ✅ | ✅ |
| PostMessage API | ✅ | ✅ | ✅ | ✅ |
| ES6+ Features | ✅ | ✅ | ✅ | ✅ |

## Integration Points

### External Systems
1. **Google Analytics 4** - Enhanced conversions and event tracking
2. **Salesforce CRM** - Lead management and scoring integration
3. **Workato** - Data pipeline and automation platform
4. **OneTrust** - Enterprise cookie consent management
5. **Facebook Pixel** - Campaign attribution and retargeting

### Internal Systems
1. **UAGC Marketing Site** (uagc.edu) - Primary lead generation
2. **Application Portal** (portal.uagc.edu) - Student application system
3. **Stealth Application** (cloud.mail.uagc.edu) - Seamless app initiation
4. **Student Portal** - Post-enrollment systems integration

## Deployment Architecture

### Server Requirements
- **Web Server:** Apache/Nginx with HTTPS support
- **SSL Certificate:** Required for secure cookie handling
- **CDN Support:** Bootstrap and icon fonts from CDN
- **Static Content:** No server-side processing required

### Configuration Files
- **.htaccess** - Apache server configuration for security headers
- **robots.txt** - Search engine directives
- **sitemap.xml** - SEO optimization
- **manifest.json** - Progressive Web App support

## Monitoring & Maintenance

### Performance Monitoring
- **Core Web Vitals** - Continuous performance tracking
- **User Experience Metrics** - Real-time interaction monitoring
- **Error Tracking** - JavaScript error logging and alerts
- **Analytics Validation** - Data quality assurance

### Maintenance Procedures
1. **Weekly Performance Audits** - Core Web Vitals assessment
2. **Monthly Security Scans** - Vulnerability assessment
3. **Quarterly Compliance Reviews** - Privacy regulation updates
4. **Annual Architecture Reviews** - Technology stack evaluation

## Related Documentation

- **[Cookie Architecture](./cookie-architecture.md)** - Detailed cookie implementation
- **[API Reference](./api-reference.md)** - Complete API documentation
- **[Event Tracking](./event-tracking.md)** - GA4 and analytics setup
- **[Cross-Domain Sync](./cross-domain-sync.md)** - Multi-domain implementation
- **[Security Specifications](./security-specifications.md)** - Security protocols
- **[Troubleshooting](./troubleshooting.md)** - Common issues and solutions

---

*Last Updated: January 27, 2025*  
*Next: [Cookie Architecture](./cookie-architecture.md)* 