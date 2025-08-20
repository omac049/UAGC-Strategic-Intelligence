# Personalization Demo Troubleshooting Guide

## Overview

This troubleshooting guide provides solutions for common issues encountered with the UAGC Interactive Personalization Demo. Use this guide to quickly diagnose and resolve problems during demo setup, execution, and stakeholder presentations.

## Quick Diagnostic Tools

### Demo Health Check
```javascript
// Run comprehensive demo diagnostics
function runDemoHealthCheck() {
  const results = {
    timestamp: new Date().toISOString(),
    personalizationEngine: checkPersonalizationEngine(),
    analytics: checkAnalytics(),
    scenarios: checkScenarios(),
    performance: checkPerformance(),
    connectivity: checkConnectivity()
  };
  
  console.log('Demo Health Check Results:', results);
  return results;
}

// Individual system checks
function checkPersonalizationEngine() {
  try {
    const engine = new PersonalizationDemo();
    return {
      status: 'operational',
      scenarios_loaded: engine.scenarios.size,
      active_visitors: engine.activeVisitors.size
    };
  } catch (error) {
    return {
      status: 'error',
      error: error.message
    };
  }
}
```

### Demo Reset Command
```javascript
// Emergency demo reset
function emergencyDemoReset() {
  try {
    // Clear all storage
    localStorage.clear();
    sessionStorage.clear();
    
    // Reset analytics
    if (window.DemoAnalytics) {
      window.DemoAnalytics.reset();
    }
    
    // Reset personalization state
    if (window.PersonalizationEngine) {
      window.PersonalizationEngine.reset();
    }
    
    // Reload page
    location.reload();
    
    return { status: 'success', message: 'Demo reset completed' };
  } catch (error) {
    return { status: 'error', message: error.message };
  }
}
```

## Common Issues and Solutions

### 1. Personalization Not Working

#### Issue: Scenarios Not Applying
**Symptoms:**
- Content doesn't change when selecting scenarios
- Visitor profile not updating
- No personalization indicators visible

**Diagnostic Steps:**
```javascript
// Check personalization engine status
console.log('Personalization Engine:', window.PersonalizationEngine);
console.log('Current Scenario:', window.PersonalizationEngine?.currentScenario);
console.log('Active Rules:', window.PersonalizationEngine?.activeRules);

// Verify scenario data
const scenarios = window.PersonalizationEngine?.scenarios;
console.log('Available Scenarios:', Array.from(scenarios?.keys() || []));
```

**Solutions:**

1. **Verify Demo Mode**
   ```javascript
   // Check if demo mode is enabled
   if (!window.DEMO_MODE) {
     console.error('Demo mode not enabled');
     window.DEMO_MODE = true;
   }
   ```

2. **Reload Scenarios**
   ```javascript
   // Force reload scenarios
   const engine = window.PersonalizationEngine;
   if (engine) {
     engine.initializeScenarios();
     console.log('Scenarios reloaded');
   }
   ```

3. **Check JavaScript Errors**
   ```javascript
   // Monitor for errors
   window.addEventListener('error', (event) => {
     console.error('Demo Error:', event.error);
   });
   ```

#### Issue: Profile Updates Not Persisting
**Symptoms:**
- Visitor profile resets unexpectedly
- Personalization rules not triggering
- Inconsistent demo behavior

**Solutions:**

1. **Fix Profile Storage**
   ```javascript
   // Ensure proper profile persistence
   class ProfileStorage {
     static save(profile) {
       try {
         localStorage.setItem('demo_profile', JSON.stringify(profile));
         sessionStorage.setItem('demo_profile_backup', JSON.stringify(profile));
       } catch (error) {
         console.error('Profile storage failed:', error);
       }
     }
     
     static load() {
       try {
         const profile = localStorage.getItem('demo_profile');
         return profile ? JSON.parse(profile) : null;
       } catch (error) {
         console.error('Profile load failed:', error);
         return null;
       }
     }
   }
   ```

2. **Validate Profile Data**
   ```javascript
   function validateProfile(profile) {
     const required = ['journey_stage', 'lead_score', 'visit_count'];
     const missing = required.filter(field => !(field in profile));
     
     if (missing.length > 0) {
       console.error('Missing profile fields:', missing);
       return false;
     }
     
     return true;
   }
   ```

### 2. Analytics Dashboard Issues

#### Issue: Dashboard Not Updating
**Symptoms:**
- Metrics stuck at zero
- Charts not refreshing
- Real-time data not flowing

**Diagnostic Steps:**
```javascript
// Check analytics system
console.log('Analytics Instance:', window.DemoAnalytics);
console.log('Event Queue:', window.DemoAnalytics?.events?.length);
console.log('Metrics:', window.DemoAnalytics?.metrics);

// Test event tracking
window.DemoAnalytics?.trackEvent('test_event', { test: true });
```

**Solutions:**

1. **Restart Analytics**
   ```javascript
   // Reinitialize analytics system
   function restartAnalytics() {
     if (window.DemoAnalytics) {
       window.DemoAnalytics.stop();
     }
     
     window.DemoAnalytics = new DemoAnalytics();
     window.DemoAnalytics.start();
     
     console.log('Analytics restarted');
   }
   ```

2. **Fix WebSocket Connection**
   ```javascript
   // Reconnect real-time updates
   function reconnectWebSocket() {
     const analytics = window.DemoAnalytics;
     if (analytics && analytics.websocket) {
       analytics.websocket.close();
       analytics.setupWebSocket();
     }
   }
   ```

3. **Manual Metric Update**
   ```javascript
   // Force dashboard refresh
   function updateDashboard() {
     const dashboard = window.DemoDashboard;
     if (dashboard) {
       dashboard.updateRealTime(window.DemoAnalytics.metrics);
     }
   }
   ```

#### Issue: Charts Not Rendering
**Symptoms:**
- Empty chart containers
- Chart.js errors in console
- Missing chart data

**Solutions:**

1. **Verify Chart.js Loading**
   ```javascript
   // Check if Chart.js is loaded
   if (typeof Chart === 'undefined') {
     console.error('Chart.js not loaded');
     // Load Chart.js dynamically
     const script = document.createElement('script');
     script.src = 'https://cdn.jsdelivr.net/npm/chart.js';
     document.head.appendChild(script);
   }
   ```

2. **Reinitialize Charts**
   ```javascript
   function reinitializeCharts() {
     const dashboard = window.DemoDashboard;
     if (dashboard) {
       // Destroy existing charts
       Object.values(dashboard.charts).forEach(chart => {
         if (chart) chart.destroy();
       });
       
       // Recreate charts
       dashboard.initializeCharts();
     }
   }
   ```

### 3. Demo Control Panel Issues

#### Issue: Scenario Selector Not Working
**Symptoms:**
- Dropdown not populating
- Apply button not responding
- Scenario changes not taking effect

**Solutions:**

1. **Fix Dropdown Population**
   ```javascript
   function populateScenarioDropdown() {
     const select = document.getElementById('scenario-select');
     const scenarios = window.PersonalizationEngine?.scenarios;
     
     if (!select || !scenarios) return;
     
     // Clear existing options
     select.innerHTML = '<option value="">Select Scenario</option>';
     
     // Add scenario options
     scenarios.forEach((scenario, id) => {
       const option = document.createElement('option');
       option.value = id;
       option.textContent = scenario.name || id;
       select.appendChild(option);
     });
   }
   ```

2. **Fix Event Listeners**
   ```javascript
   function attachControlEventListeners() {
     const applyButton = document.getElementById('apply-scenario');
     const scenarioSelect = document.getElementById('scenario-select');
     
     if (applyButton) {
       applyButton.addEventListener('click', () => {
         const selectedScenario = scenarioSelect.value;
         if (selectedScenario) {
           window.PersonalizationEngine.applyScenario(selectedScenario);
         }
       });
     }
   }
   ```

### 4. Performance Issues

#### Issue: Slow Personalization Response
**Symptoms:**
- Delayed content updates
- Laggy interface interactions
- Timeout errors

**Diagnostic Steps:**
```javascript
// Measure personalization performance
function measurePersonalizationPerformance() {
  const startTime = performance.now();
  
  window.PersonalizationEngine.applyScenario('test-scenario');
  
  const endTime = performance.now();
  const duration = endTime - startTime;
  
  console.log(`Personalization took ${duration} milliseconds`);
  
  if (duration > 200) {
    console.warn('Personalization is slow');
  }
}
```

**Solutions:**

1. **Enable Performance Mode**
   ```javascript
   // Optimize for demo performance
   function enablePerformanceMode() {
     // Reduce animation delays
     document.documentElement.style.setProperty('--animation-duration', '0.1s');
     
     // Disable heavy features
     window.DEMO_PERFORMANCE_MODE = true;
     
     // Clear caches
     if (window.PersonalizationCache) {
       window.PersonalizationCache.clear();
     }
   }
   ```

2. **Preload Demo Data**
   ```javascript
   // Preload all demo scenarios
   function preloadDemoData() {
     const engine = window.PersonalizationEngine;
     if (!engine) return;
     
     // Preload all scenarios
     engine.scenarios.forEach((scenario, id) => {
       engine.preloadScenario(id);
     });
     
     console.log('Demo data preloaded');
   }
   ```

### 5. Browser Compatibility Issues

#### Issue: Demo Not Working in Specific Browsers
**Symptoms:**
- JavaScript errors in older browsers
- Missing features or broken layout
- Console errors about unsupported APIs

**Solutions:**

1. **Browser Detection and Fallbacks**
   ```javascript
   function checkBrowserCompatibility() {
     const unsupported = [];
     
     // Check for required features
     if (!window.localStorage) unsupported.push('localStorage');
     if (!window.fetch) unsupported.push('fetch API');
     if (!window.Promise) unsupported.push('Promises');
     if (!window.Map) unsupported.push('Map');
     
     if (unsupported.length > 0) {
       console.warn('Unsupported features:', unsupported);
       return false;
     }
     
     return true;
   }
   ```

2. **Polyfill Loading**
   ```javascript
   function loadPolyfills() {
     // Load polyfills for older browsers
     if (!window.fetch) {
       const script = document.createElement('script');
       script.src = 'https://polyfill.io/v3/polyfill.min.js?features=fetch';
       document.head.appendChild(script);
     }
   }
   ```

## Emergency Procedures

### Demo Day Emergency Protocol

#### 1. Complete System Failure
```javascript
// Emergency recovery steps
const EmergencyRecovery = {
  step1_basicReset() {
    localStorage.clear();
    sessionStorage.clear();
    location.reload();
  },
  
  step2_fallbackMode() {
    // Switch to static demo mode
    window.DEMO_FALLBACK_MODE = true;
    this.loadStaticDemo();
  },
  
  step3_manualDemo() {
    // Prepare for manual demonstration
    this.showManualDemoInstructions();
  },
  
  loadStaticDemo() {
    // Load pre-recorded demo scenarios
    const staticScenarios = {
      'scenario1': { /* static data */ },
      'scenario2': { /* static data */ },
      'scenario3': { /* static data */ }
    };
    
    window.StaticDemo = new StaticDemoPlayer(staticScenarios);
  }
};
```

#### 2. Network Connectivity Issues
```javascript
// Offline demo mode
function enableOfflineMode() {
  // Use cached data
  const cachedData = localStorage.getItem('demo_cache');
  if (cachedData) {
    window.DemoData = JSON.parse(cachedData);
    console.log('Offline mode enabled with cached data');
  }
  
  // Disable real-time features
  window.OFFLINE_MODE = true;
}
```

### Stakeholder Demo Checklist

#### Pre-Demo (15 minutes before)
```bash
# Run pre-demo checklist
function preDemoChecklist() {
  const checks = [
    () => runDemoHealthCheck(),
    () => testAllScenarios(),
    () => verifyAnalyticsDashboard(),
    () => checkPerformance(),
    () => prepareBackupOptions()
  ];
  
  const results = checks.map(check => {
    try {
      return { status: 'pass', result: check() };
    } catch (error) {
      return { status: 'fail', error: error.message };
    }
  });
  
  console.log('Pre-demo check results:', results);
  return results;
}
```

#### During Demo
- Keep browser console open for monitoring
- Have emergency reset button ready
- Monitor performance metrics
- Be prepared to switch to fallback mode

#### Post-Demo
- Collect any error logs
- Document issues for improvement
- Reset demo for next use

## Monitoring and Alerts

### Real-Time Monitoring
```javascript
// Demo monitoring system
class DemoMonitor {
  constructor() {
    this.alerts = [];
    this.monitoring = false;
  }
  
  startMonitoring() {
    this.monitoring = true;
    
    // Monitor key metrics
    setInterval(() => {
      this.checkSystemHealth();
    }, 5000); // Every 5 seconds
  }
  
  checkSystemHealth() {
    const health = {
      personalization: this.checkPersonalizationEngine(),
      analytics: this.checkAnalytics(),
      performance: this.checkPerformance()
    };
    
    // Alert on issues
    Object.entries(health).forEach(([system, status]) => {
      if (status.status === 'error') {
        this.alert(`${system} system error: ${status.message}`);
      }
    });
  }
  
  alert(message) {
    console.warn('Demo Alert:', message);
    this.alerts.push({
      timestamp: Date.now(),
      message: message
    });
    
    // Show visual alert if needed
    this.showVisualAlert(message);
  }
}
```

## Support Resources

### Quick Reference Commands
```javascript
// Essential demo commands
const DemoCommands = {
  // Reset everything
  reset: () => emergencyDemoReset(),
  
  // Check system status
  status: () => runDemoHealthCheck(),
  
  // Apply specific scenario
  scenario: (id) => window.PersonalizationEngine?.applyScenario(id),
  
  // Enable performance mode
  performance: () => enablePerformanceMode(),
  
  // Export demo data
  export: () => window.DemoAnalytics?.generateReport()
};

// Make commands globally available
window.demo = DemoCommands;
```

### Contact Information
- **Technical Support**: demo-support@uagc.edu
- **Emergency Contact**: +1-555-DEMO-911
- **Backup Presenter**: backup-demo@uagc.edu

### External Resources
- [Demo Documentation](./overview.md)
- [Implementation Guide](./implementation-guide.md)
- [Browser Compatibility Matrix](https://caniuse.com)
- [Chart.js Documentation](https://www.chartjs.org/docs/)

## Related Documentation

- [Personalization Demo Overview](./overview.md)
- [Implementation Guide](./implementation-guide.md)
- [Technical Implementation](../03-technical-implementation/overview.md)
- [Success Metrics](../08-success-metrics/overview.md) 