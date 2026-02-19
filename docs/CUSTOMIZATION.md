# Customization Guide

## Table of Contents
- [Color Customization](#color-customization)
- [Typography](#typography)
- [Adding Components](#adding-components)
- [Animations](#animations)
- [Icons](#icons)
- [Layout Modifications](#layout-modifications)

## Color Customization

### Primary Color Scheme
All colors are defined in CSS custom properties at the top of `index.html`:

```css
:root {
    --primary-color: #4F46E5;      /* Indigo - Main brand color */
    --secondary-color: #7C3AED;    /* Purple - Secondary actions */
    --accent-color: #06B6D4;       /* Cyan - Highlights and links */
    --dark-bg: #0F172A;            /* Dark blue - Main background */
    --card-bg: rgba(30, 41, 59, 0.7);  /* Card backgrounds */
    --text-primary: #F1F5F9;       /* Primary text color */
    --text-secondary: #94A3B8;     /* Secondary text color */
    --success: #10B981;            /* Success states */
    --warning: #F59E0B;            /* Warning states */
    --danger: #EF4444;             /* Error states */
    --glass-bg: rgba(255, 255, 255, 0.05);      /* Glassmorphism base */
    --glass-border: rgba(255, 255, 255, 0.1);   /* Glassmorphism borders */
}
```

### Creating Custom Color Themes

**Blue Ocean Theme:**
```css
:root {
    --primary-color: #0EA5E9;
    --secondary-color: #0284C7;
    --accent-color: #06B6D4;
}
```

**Green Forest Theme:**
```css
:root {
    --primary-color: #10B981;
    --secondary-color: #059669;
    --accent-color: #34D399;
}
```

**Red Fire Theme:**
```css
:root {
    --primary-color: #EF4444;
    --secondary-color: #DC2626;
    --accent-color: #F87171;
}
```

## Typography

### Changing Fonts

**Using Google Fonts:**
```html
<head>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700;800&display=swap" rel="stylesheet">
</head>

<style>
    body {
        font-family: 'Poppins', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    }
</style>
```

### Font Size Adjustments
```css
/* Larger text for accessibility */
body {
    font-size: 18px; /* Default is 16px */
}

.hero h1 {
    font-size: 4rem; /* Default is 3.5rem */
}

.card h3 {
    font-size: 1.75rem; /* Default is 1.5rem */
}
```

## Adding Components

### New Dashboard Card

```html
<div class="card scroll-reveal" style="animation-delay: 0.6s">
    <div class="card-icon">
        <i class="fas fa-database"></i>
    </div>
    <div class="card-header">
        <div>
            <h3>Database Monitor</h3>
            <span class="card-badge">NEW</span>
        </div>
    </div>
    <p>Monitor database performance and query analytics in real-time</p>
    <button class="card-button" onclick="openApp('database')">
        Open Monitor <i class="fas fa-arrow-right"></i>
    </button>
</div>
```

### New AI Prompt Card

In `ai-assistant.html`, add to prompts grid:

```html
<div class="prompt-card security" onclick="selectPrompt('security-audit')">
    <div class="prompt-icon">
        <i class="fas fa-shield-alt"></i>
    </div>
    <h3>Security Audit</h3>
    <p>Run a comprehensive security audit across all network devices and generate a report.</p>
</div>
```

Add to JavaScript prompts object:
```javascript
const prompts = {
    // ... existing prompts
    'security-audit': 'Run a comprehensive security audit across all network devices including firewall rules, open ports, and vulnerability scanning.'
};
```

Add custom color styling:
```css
.prompt-card.security .prompt-icon {
    background: linear-gradient(135deg, #EF4444, #DC2626);
}
```

### Status Indicators

Add live status badges:
```html
<div class="status-indicator">
    <span class="status-dot online"></span>
    <span>All Systems Operational</span>
</div>

<style>
.status-indicator {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem 1rem;
    background: var(--glass-bg);
    border-radius: 50px;
}

.status-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    animation: pulse-dot 2s infinite;
}

.status-dot.online { background: var(--success); }
.status-dot.warning { background: var(--warning); }
.status-dot.error { background: var(--danger); }

@keyframes pulse-dot {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.5; }
}
</style>
```

## Animations

### Adjusting Animation Speed

```css
/* Make animations faster */
.card {
    transition: all 0.2s ease; /* Default is 0.4s */
}

/* Make scroll reveals slower */
.scroll-reveal {
    transition: all 1.2s ease; /* Default is 0.8s */
}
```

### Disabling Animations
```css
/* Disable all animations (for accessibility) */
*, *::before, *::after {
    animation-duration: 0s !important;
    transition-duration: 0s !important;
}
```

### Custom Entrance Animation

```css
@keyframes slideInRight {
    from {
        opacity: 0;
        transform: translateX(100px);
    }
    to {
        opacity: 1;
        transform: translateX(0);
    }
}

.custom-animation {
    animation: slideInRight 0.8s ease;
}
```

## Icons

### Using Different Icon Libraries

**Material Icons:**
```html
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">

<div class="card-icon">
    <span class="material-icons">network_check</span>
</div>
```

**Bootstrap Icons:**
```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.0/font/bootstrap-icons.css">

<div class="card-icon">
    <i class="bi bi-router"></i>
</div>
```

### Custom SVG Icons
```html
<div class="card-icon">
    <svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor">
        <path d="M12 2L2 7v10c0 5.55 3.84 10.74 9 12 5.16-1.26 9-6.45 9-12V7l-10-5z"/>
    </svg>
</div>
```

## Layout Modifications

### Changing Grid Columns

```css
/* 2 columns on desktop */
.cards-grid {
    grid-template-columns: repeat(2, 1fr);
}

/* 4 columns on large screens */
@media (min-width: 1400px) {
    .cards-grid {
        grid-template-columns: repeat(4, 1fr);
    }
}

/* Single column on mobile */
@media (max-width: 768px) {
    .cards-grid {
        grid-template-columns: 1fr;
    }
}
```

### Sidebar Layout

Add a sidebar:
```html
<div class="layout-with-sidebar">
    <aside class="sidebar">
        <!-- Sidebar content -->
    </aside>
    <main class="main-content">
        <!-- Existing content -->
    </main>
</div>

<style>
.layout-with-sidebar {
    display: grid;
    grid-template-columns: 250px 1fr;
    gap: 2rem;
}

.sidebar {
    background: var(--glass-bg);
    backdrop-filter: blur(20px);
    border-radius: 24px;
    padding: 2rem;
    height: fit-content;
    position: sticky;
    top: 100px;
}
</style>
```

### Card Sizes

```css
/* Large featured card */
.card.large {
    grid-column: span 2;
}

/* Small card */
.card.small {
    padding: 1.5rem;
}

.card.small .card-icon {
    width: 40px;
    height: 40px;
    font-size: 1rem;
}
```

## Advanced Customization

### Adding Background Patterns

```css
.bg-animation::after {
    content: '';
    position: absolute;
    width: 100%;
    height: 100%;
    background-image: 
        radial-gradient(circle, rgba(255,255,255,0.05) 1px, transparent 1px);
    background-size: 50px 50px;
}
```

### Custom Scrollbar

```css
::-webkit-scrollbar {
    width: 12px;
}

::-webkit-scrollbar-track {
    background: var(--dark-bg);
    border-radius: 10px;
}

::-webkit-scrollbar-thumb {
    background: linear-gradient(135deg, var(--primary-color), var(--accent-color));
    border-radius: 10px;
}
```

### Hover Glow Effect

```css
.card:hover {
    box-shadow: 
        0 30px 60px rgba(0, 0, 0, 0.5),
        0 0 80px rgba(79, 70, 229, 0.3);
}
```

## Performance Tips

1. **Reduce animation complexity** on mobile:
```css
@media (max-width: 768px) {
    .card:hover {
        transform: translateY(-5px); /* Less complex than 3D */
    }
}
```

2. **Lazy load images:**
```html
<img src="placeholder.jpg" data-src="actual-image.jpg" loading="lazy" alt="Description">
```

3. **Minimize backdrop-filter usage** on low-end devices:
```css
@media (prefers-reduced-motion: reduce) {
    .card {
        backdrop-filter: none;
        background: var(--card-bg);
    }
}
```

---

For more customization options, see the main README.md file.