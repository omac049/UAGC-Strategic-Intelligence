# Section 2: Behavioral Tracking Implementation Guide

## Implementation Overview

This guide provides comprehensive technical implementation details for the UAGC behavioral tracking system that powers user journey personalization. The system captures, processes, and acts upon user behavioral data to deliver tailored educational experiences.

## Core Tracking Infrastructure

### Event Tracking Architecture

#### Universal Event Schema
```javascript
const universalEventSchema = {
  // Core Event Properties
  eventId: 'uuid-v4-generated',
  timestamp: 'ISO-8601-datetime',
  sessionId: 'session-identifier',
  userId: 'persistent-user-identifier',
  
  // Event Classification
  eventType: 'page_view|interaction|conversion|system',
  eventCategory: 'navigation|engagement|form|download',
  eventAction: 'click|view|submit|download|scroll',
  eventLabel: 'specific-element-or-content-identifier',
  
  // Context Data
  pageUrl: 'current-page-url',
  referrer: 'referrer-url',
  userAgent: 'browser-user-agent',
  screenResolution: 'width-x-height',
  viewportSize: 'width-x-height',
  
  // Behavioral Context
  timeOnPage: 'seconds-since-page-load',
  scrollDepth: 'percentage-of-page-scrolled',
  clickPosition: { x: 'pixel-x-coordinate', y: 'pixel-y-coordinate' },
  
  // Personalization Context
  userSegment: 'behavioral-segment-classification',
  journeyStage: 'current-journey-stage',
  personalizationVariant: 'active-personalization-version',
  
  // Attribution Data
  utmSource: 'traffic-source',
  utmMedium: 'traffic-medium',
  utmCampaign: 'campaign-identifier',
  utmContent: 'content-identifier',
  utmTerm: 'keyword-term'
};
```

#### Event Collection System
```javascript
class BehavioralTracker {
  constructor(config) {
    this.config = {
      endpoint: config.trackingEndpoint || '/api/events',
      batchSize: config.batchSize || 10,
      flushInterval: config.flushInterval || 5000,
      sessionTimeout: config.sessionTimeout || 1800000, // 30 minutes
      ...config
    };
    
    this.eventQueue = [];
    this.sessionData = this.initializeSession();
    this.userProfile = this.loadUserProfile();
    
    this.initializeTracking();
  }
  
  // Session Management
  initializeSession() {
    const existingSession = this.getStoredSession();
    if (existingSession && !this.isSessionExpired(existingSession)) {
      return existingSession;
    }
    
    return {
      sessionId: this.generateUUID(),
      startTime: Date.now(),
      lastActivity: Date.now(),
      pageViews: 0,
      interactions: 0,
      conversions: 0
    };
  }
  
  // Core Event Tracking
  track(eventType, eventData) {
    const event = {
      ...universalEventSchema,
      eventId: this.generateUUID(),
      timestamp: new Date().toISOString(),
      sessionId: this.sessionData.sessionId,
      userId: this.userProfile.userId,
      eventType: eventType,
      ...eventData,
      ...this.getContextualData()
    };
    
    this.eventQueue.push(event);
    this.updateSession(event);
    
    if (this.eventQueue.length >= this.config.batchSize) {
      this.flushEvents();
    }
  }
  
  // Contextual Data Collection
  getContextualData() {
    return {
      pageUrl: window.location.href,
      referrer: document.referrer,
      userAgent: navigator.userAgent,
      screenResolution: `${screen.width}x${screen.height}`,
      viewportSize: `${window.innerWidth}x${window.innerHeight}`,
      timeOnPage: Date.now() - this.pageLoadTime,
      scrollDepth: this.calculateScrollDepth(),
      userSegment: this.userProfile.segment,
      journeyStage: this.userProfile.journeyStage
    };
  }
}
```

### Page-Level Tracking Implementation

#### Automatic Page View Tracking
```javascript
class PageViewTracker extends BehavioralTracker {
  initializePageTracking() {
    // Track initial page load
    this.trackPageView();
    
    // Track SPA navigation
    this.setupSPATracking();
    
    // Track scroll behavior
    this.setupScrollTracking();
    
    // Track exit intent
    this.setupExitIntentTracking();
  }
  
  trackPageView(customData = {}) {
    const pageData = {
      eventCategory: 'navigation',
      eventAction: 'page_view',
      eventLabel: this.getPageIdentifier(),
      pageTitle: document.title,
      pageType: this.classifyPageType(),
      contentCategory: this.extractContentCategory(),
      ...customData
    };
    
    this.track('page_view', pageData);
    this.sessionData.pageViews++;
  }
  
  setupScrollTracking() {
    const scrollMilestones = [25, 50, 75, 100];
    let trackedMilestones = [];
    
    const trackScrollMilestone = () => {
      const scrollPercent = this.calculateScrollDepth();
      
      scrollMilestones.forEach(milestone => {
        if (scrollPercent >= milestone && !trackedMilestones.includes(milestone)) {
          trackedMilestones.push(milestone);
          
          this.track('interaction', {
            eventCategory: 'engagement',
            eventAction: 'scroll',
            eventLabel: `${milestone}%`,
            scrollDepth: milestone
          });
        }
      });
    };
    
    window.addEventListener('scroll', this.throttle(trackScrollMilestone, 250));
  }
  
  setupExitIntentTracking() {
    let exitIntentTriggered = false;
    
    document.addEventListener('mouseleave', (event) => {
      if (event.clientY <= 0 && !exitIntentTriggered) {
        exitIntentTriggered = true;
        
        this.track('interaction', {
          eventCategory: 'engagement',
          eventAction: 'exit_intent',
          eventLabel: 'mouse_leave',
          timeOnPage: Date.now() - this.pageLoadTime
        });
        
        // Trigger exit intent personalization
        this.triggerExitIntentPersonalization();
      }
    });
  }
}
```

### Interaction Tracking System

#### Click and Form Interaction Tracking
```javascript
class InteractionTracker extends BehavioralTracker {
  initializeInteractionTracking() {
    this.setupClickTracking();
    this.setupFormTracking();
    this.setupContentEngagementTracking();
  }
  
  setupClickTracking() {
    document.addEventListener('click', (event) => {
      const element = event.target;
      const clickData = this.analyzeClickElement(element);
      
      if (clickData.trackable) {
        this.track('interaction', {
          eventCategory: 'engagement',
          eventAction: 'click',
          eventLabel: clickData.label,
          elementType: clickData.elementType,
          elementText: clickData.text,
          elementId: clickData.id,
          elementClass: clickData.className,
          clickPosition: { x: event.clientX, y: event.clientY },
          ...clickData.customProperties
        });
        
        // Check for conversion events
        if (clickData.isConversion) {
          this.trackConversion(clickData.conversionType, clickData);
        }
      }
    });
  }
  
  analyzeClickElement(element) {
    const analysis = {
      trackable: true,
      elementType: element.tagName.toLowerCase(),
      text: element.textContent.trim(),
      id: element.id,
      className: element.className,
      customProperties: {}
    };
    
    // Analyze specific element types
    switch (analysis.elementType) {
      case 'a':
        analysis.label = `link_${element.href}`;
        analysis.customProperties.linkDestination = element.href;
        analysis.customProperties.linkType = this.classifyLinkType(element.href);
        break;
        
      case 'button':
        analysis.label = `button_${analysis.text}`;
        analysis.customProperties.buttonType = element.type;
        analysis.isConversion = this.isConversionButton(element);
        break;
        
      case 'input':
        if (element.type === 'submit') {
          analysis.label = `submit_${element.value}`;
          analysis.isConversion = true;
          analysis.conversionType = 'form_submission';
        }
        break;
        
      default:
        analysis.label = `${analysis.elementType}_${analysis.id || analysis.className}`;
    }
    
    return analysis;
  }
  
  setupFormTracking() {
    // Track form field interactions
    document.addEventListener('focus', (event) => {
      if (event.target.tagName === 'INPUT' || event.target.tagName === 'TEXTAREA' || event.target.tagName === 'SELECT') {
        this.trackFormFieldInteraction('focus', event.target);
      }
    }, true);
    
    document.addEventListener('blur', (event) => {
      if (event.target.tagName === 'INPUT' || event.target.tagName === 'TEXTAREA' || event.target.tagName === 'SELECT') {
        this.trackFormFieldInteraction('blur', event.target);
      }
    }, true);
    
    // Track form submissions
    document.addEventListener('submit', (event) => {
      this.trackFormSubmission(event);
    });
  }
  
  trackFormFieldInteraction(action, field) {
    const formData = this.analyzeFormField(field);
    
    this.track('interaction', {
      eventCategory: 'form',
      eventAction: action,
      eventLabel: `${formData.formType}_${formData.fieldName}`,
      formId: formData.formId,
      formType: formData.formType,
      fieldName: formData.fieldName,
      fieldType: formData.fieldType,
      fieldRequired: formData.required,
      fieldPosition: formData.position
    });
  }
  
  trackFormSubmission(event) {
    const form = event.target;
    const formData = this.analyzeForm(form);
    
    this.track('conversion', {
      eventCategory: 'form',
      eventAction: 'submit',
      eventLabel: formData.formType,
      formId: formData.formId,
      formType: formData.formType,
      fieldCount: formData.fieldCount,
      completionRate: formData.completionRate,
      timeToComplete: formData.timeToComplete
    });
    
    // Update user journey stage
    this.updateJourneyStage(formData.formType);
  }
}
```

### Conversion Tracking Framework

#### Micro and Macro Conversion Tracking
```javascript
class ConversionTracker extends BehavioralTracker {
  constructor(config) {
    super(config);
    this.conversionDefinitions = this.loadConversionDefinitions();
  }
  
  loadConversionDefinitions() {
    return {
      microConversions: {
        newsletter_signup: { value: 1, stage: 'awareness' },
        resource_download: { value: 2, stage: 'interest' },
        program_inquiry: { value: 3, stage: 'consideration' },
        advisor_request: { value: 5, stage: 'consideration' },
        campus_visit_request: { value: 4, stage: 'consideration' }
      },
      macroConversions: {
        rfi_submission: { value: 10, stage: 'consideration' },
        application_start: { value: 25, stage: 'intent' },
        application_submit: { value: 50, stage: 'application' },
        enrollment_complete: { value: 100, stage: 'enrollment' }
      }
    };
  }
  
  trackConversion(conversionType, conversionData = {}) {
    const conversionDef = this.getConversionDefinition(conversionType);
    
    if (!conversionDef) {
      console.warn(`Unknown conversion type: ${conversionType}`);
      return;
    }
    
    const conversionEvent = {
      eventCategory: 'conversion',
      eventAction: conversionType,
      eventLabel: conversionData.label || conversionType,
      conversionValue: conversionDef.value,
      conversionStage: conversionDef.stage,
      conversionId: this.generateUUID(),
      ...conversionData
    };
    
    this.track('conversion', conversionEvent);
    
    // Update user profile
    this.updateUserProfile({
      lastConversion: conversionType,
      lastConversionTime: Date.now(),
      totalConversionValue: this.userProfile.totalConversionValue + conversionDef.value,
      journeyStage: conversionDef.stage
    });
    
    // Trigger post-conversion personalization
    this.triggerPostConversionPersonalization(conversionType, conversionEvent);
  }
  
  // Enhanced Conversion Attribution
  trackEnhancedConversion(conversionType, enhancedData) {
    const baseConversion = this.trackConversion(conversionType, enhancedData);
    
    // GA4 Enhanced Conversions
    if (window.gtag) {
      gtag('event', 'conversion', {
        send_to: 'AW-CONVERSION_ID/CONVERSION_LABEL',
        email: enhancedData.email,
        phone_number: enhancedData.phone,
        first_name: enhancedData.firstName,
        last_name: enhancedData.lastName,
        street: enhancedData.address,
        city: enhancedData.city,
        region: enhancedData.state,
        postal_code: enhancedData.zip,
        country: enhancedData.country
      });
    }
    
    // Salesforce Lead API
    this.sendToSalesforce(conversionType, enhancedData);
  }
}
```

## Personalization Trigger System

### Real-Time Personalization Engine
```javascript
class PersonalizationEngine extends BehavioralTracker {
  constructor(config) {
    super(config);
    this.personalizationRules = this.loadPersonalizationRules();
    this.activePersonalizations = new Map();
  }
  
  loadPersonalizationRules() {
    return {
      // Content Personalization Rules
      programRecommendations: {
        trigger: 'program_page_view',
        conditions: [
          { field: 'timeOnPage', operator: '>', value: 30 },
          { field: 'scrollDepth', operator: '>', value: 50 }
        ],
        action: 'show_related_programs',
        priority: 1
      },
      
      exitIntentOffer: {
        trigger: 'exit_intent',
        conditions: [
          { field: 'pageViews', operator: '>', value: 1 },
          { field: 'timeOnSite', operator: '>', value: 120 }
        ],
        action: 'show_exit_intent_modal',
        priority: 2
      },
      
      advisorChatOffer: {
        trigger: 'form_field_focus',
        conditions: [
          { field: 'formType', operator: '==', value: 'rfi' },
          { field: 'sessionPageViews', operator: '>', value: 3 }
        ],
        action: 'offer_advisor_chat',
        priority: 3
      },
      
      // Journey Stage Personalization
      considerationStageNurture: {
        trigger: 'journey_stage_update',
        conditions: [
          { field: 'journeyStage', operator: '==', value: 'consideration' },
          { field: 'daysSinceFirstVisit', operator: '>', value: 3 }
        ],
        action: 'trigger_nurture_sequence',
        priority: 1
      }
    };
  }
  
  evaluatePersonalizationTriggers(event) {
    Object.entries(this.personalizationRules).forEach(([ruleName, rule]) => {
      if (this.shouldTriggerRule(event, rule)) {
        this.executePersonalizationAction(ruleName, rule, event);
      }
    });
  }
  
  shouldTriggerRule(event, rule) {
    // Check if event matches trigger
    if (event.eventAction !== rule.trigger) {
      return false;
    }
    
    // Evaluate conditions
    return rule.conditions.every(condition => {
      const fieldValue = this.getFieldValue(condition.field, event);
      return this.evaluateCondition(fieldValue, condition.operator, condition.value);
    });
  }
  
  executePersonalizationAction(ruleName, rule, event) {
    const personalizationId = `${ruleName}_${Date.now()}`;
    
    switch (rule.action) {
      case 'show_related_programs':
        this.showRelatedPrograms(event, personalizationId);
        break;
        
      case 'show_exit_intent_modal':
        this.showExitIntentModal(event, personalizationId);
        break;
        
      case 'offer_advisor_chat':
        this.offerAdvisorChat(event, personalizationId);
        break;
        
      case 'trigger_nurture_sequence':
        this.triggerNurtureSequence(event, personalizationId);
        break;
    }
    
    // Track personalization execution
    this.track('personalization', {
      eventCategory: 'personalization',
      eventAction: 'executed',
      eventLabel: ruleName,
      personalizationId: personalizationId,
      rulePriority: rule.priority
    });
  }
}
```

## Advanced Analytics Implementation

### Behavioral Segmentation Engine
```javascript
class BehavioralSegmentation extends BehavioralTracker {
  segmentUser(userProfile, sessionData, behavioralHistory) {
    const segments = [];
    
    // High-Intent Segment
    if (this.isHighIntentUser(userProfile, behavioralHistory)) {
      segments.push('high_intent');
    }
    
    // Research-Heavy Segment
    if (this.isResearchHeavyUser(sessionData, behavioralHistory)) {
      segments.push('research_heavy');
    }
    
    // Mobile-Primary Segment
    if (this.isMobilePrimaryUser(behavioralHistory)) {
      segments.push('mobile_primary');
    }
    
    // Price-Sensitive Segment
    if (this.isPriceSensitiveUser(behavioralHistory)) {
      segments.push('price_sensitive');
    }
    
    // Career-Focused Segment
    if (this.isCareerFocusedUser(behavioralHistory)) {
      segments.push('career_focused');
    }
    
    return segments;
  }
  
  isHighIntentUser(userProfile, behavioralHistory) {
    const indicators = [
      behavioralHistory.formInteractions > 2,
      behavioralHistory.advisorContactRequests > 0,
      behavioralHistory.applicationPageViews > 1,
      behavioralHistory.programPageDepth > 5,
      behavioralHistory.returnVisits > 3
    ];
    
    return indicators.filter(Boolean).length >= 3;
  }
  
  calculateEngagementScore(behavioralHistory) {
    const weights = {
      pageViews: 1,
      timeOnSite: 0.01, // per second
      formInteractions: 5,
      contentDownloads: 3,
      videoPlays: 4,
      socialShares: 2,
      returnVisits: 3
    };
    
    let score = 0;
    Object.entries(weights).forEach(([metric, weight]) => {
      score += (behavioralHistory[metric] || 0) * weight;
    });
    
    return Math.min(score, 100); // Cap at 100
  }
}
```

## Performance Optimization

### Efficient Data Collection
```javascript
class OptimizedTracker extends BehavioralTracker {
  constructor(config) {
    super(config);
    this.setupPerformanceOptimizations();
  }
  
  setupPerformanceOptimizations() {
    // Implement event batching
    this.setupEventBatching();
    
    // Use web workers for heavy processing
    this.setupWebWorkerProcessing();
    
    // Implement intelligent sampling
    this.setupIntelligentSampling();
    
    // Setup offline capability
    this.setupOfflineCapability();
  }
  
  setupEventBatching() {
    setInterval(() => {
      if (this.eventQueue.length > 0) {
        this.flushEvents();
      }
    }, this.config.flushInterval);
    
    // Flush on page unload
    window.addEventListener('beforeunload', () => {
      this.flushEvents(true); // Synchronous flush
    });
  }
  
  setupIntelligentSampling() {
    // Sample based on user segment and session value
    this.samplingRate = this.calculateSamplingRate();
    
    this.originalTrack = this.track;
    this.track = (eventType, eventData) => {
      if (Math.random() <= this.samplingRate) {
        this.originalTrack(eventType, eventData);
      }
    };
  }
  
  calculateSamplingRate() {
    const userSegment = this.userProfile.segment;
    const engagementScore = this.userProfile.engagementScore;
    
    // Higher sampling for high-value users
    if (userSegment === 'high_intent' || engagementScore > 75) {
      return 1.0; // 100% sampling
    } else if (engagementScore > 50) {
      return 0.8; // 80% sampling
    } else {
      return 0.5; // 50% sampling
    }
  }
}
```

## Data Privacy and Compliance

### GDPR/CCPA Compliant Tracking
```javascript
class CompliantTracker extends BehavioralTracker {
  constructor(config) {
    super(config);
    this.consentManager = new ConsentManager();
    this.setupComplianceFeatures();
  }
  
  setupComplianceFeatures() {
    // Check consent before tracking
    this.originalTrack = this.track;
    this.track = (eventType, eventData) => {
      if (this.consentManager.hasConsent('analytics')) {
        this.originalTrack(eventType, eventData);
      }
    };
    
    // Implement data retention policies
    this.setupDataRetention();
    
    // Provide data export/deletion capabilities
    this.setupDataRights();
  }
  
  anonymizeUserData(userData) {
    const anonymized = { ...userData };
    
    // Remove PII
    delete anonymized.email;
    delete anonymized.phone;
    delete anonymized.name;
    delete anonymized.address;
    
    // Hash IP address
    if (anonymized.ipAddress) {
      anonymized.ipAddress = this.hashIP(anonymized.ipAddress);
    }
    
    return anonymized;
  }
  
  handleDataDeletionRequest(userId) {
    // Remove user data from tracking systems
    this.deleteUserEvents(userId);
    this.deleteUserProfile(userId);
    this.notifyDownstreamSystems(userId, 'delete');
  }
}
```

This comprehensive behavioral tracking implementation provides the technical foundation for understanding user behavior, delivering personalized experiences, and optimizing the user journey while maintaining compliance with privacy regulations. 