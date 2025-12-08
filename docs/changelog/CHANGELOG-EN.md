# 📝 XPay Theme Changelog

All notable changes to this project will be documented in this file.

---

## [2.1.0] - 2025-12-07

### ✨ Added

#### 🗺️ Source Maps for Webpack
- **Full Source Maps Support Implementation**
  - Configured webpack to generate source maps
  - Automatic creation of versioned files with source maps
  - Fixed PageSpeed error "Missing source maps for large JavaScript"
  - Automated scripts for building versioned files:
    - `create-versioned-sourcemaps.js` (Node.js)
    - `create-versioned-sourcemaps.ps1` (PowerShell)
  - Enhanced Browser Cache Management to regenerate source maps

#### 🔔 Najva Notifications Optimization
- **Fixed Permission Request Without Context**
  - Load Najva based on user interaction (scroll, click, touch)
  - Smart notification permission state checking
  - Immediate load for users who already granted permission
  - Ability to disable via `.env` (`ENABLE_NAJVA`)
  - Automatic disable in debug mode
  - Fixed PageSpeed error: "Requests notification permission on page load"

### 🔧 Improved

#### Performance
- **Added Page Loader to Prevent FOUC**
  - Display loading screen until page is fully ready
  - Prevents Flash of Unstyled Content
  - Smooth fade out animation
  - Automatic fallback after 3 seconds
- **Optimized Forced Reflow in custom-coins.js**
  - Added `batchReadWrite()` to separate DOM reads/writes
  - Added `batchMutate()` to batch DOM mutations
  - Optimized Mobile Menu operations
  - Separated computed style reads from DOM mutations
  - Reduced reflow time from 70ms to under 10ms
- Enhanced load time with delayed Najva script
- Used `passive: true` in event listeners
- Used `once: true` for automatic listener removal
- Reduced main thread blocking

#### Security
- Implemented strong Content Security Policy (CSP)
- Added CSP via HTTP headers instead of meta tag (more secure)
- Protection against XSS attacks
- Defined script-src and object-src directives
- Blocked execution of unsafe scripts
- Set object-src to 'none' to prevent injection
- Restricted executable resources to specific domains
- **Updated CSP to Fix Console Errors**
  - Added `api.xpay.co` to connect-src
  - Added `s1.mediaad.org` to script-src
  - Fixed "Refused to connect" error for API requests
  - Fixed "Refused to load script" error for mediaad retargeting

#### Images & Assets
- **Fixed Aspect Ratio issues in images**
  - Corrected header-qr.webp dimensions from 200x200 to 200x71
  - Maintained actual aspect ratio of 2.80:1 (1278x456)
  - Fixed PageSpeed Insights error about incorrect aspect ratio
- **Automatic width and height addition to content images**
  - Automatic filter for `the_content`
  - Read dimensions from image metadata
  - Support for wp-image-{id} class
  - Improved CLS (Cumulative Layout Shift)
  - Fixed PageSpeed error: "Image elements do not have explicit width and height"

### 🐛 Fixed

#### Error Handling
- **Improved Error Handling in Page Loader**
  - Added try-catch for removeChild operation
  - Check body existence before classList operations
  - Double check loader existence before removal
  - Fixed TypeError: "Cannot read properties of null"

### 🔄 CI/CD

- **Enhanced GitHub Actions Deploy Workflow**
  - Added Node.js setup to workflow
  - Automatic NPM dependencies installation
  - Automatic `npm run build:production` execution
  - Automatic versioned assets build before deploy
  - Upload versioned files to server
  - Removed build files from exclude list

#### Developer Experience
- Added new npm script: `build:production`
- Complete Source Maps documentation in `docs/SOURCE_MAPS_README.md`
- Complete Najva documentation in `docs/NAJVA_OPTIMIZATION.md`
- Step-by-step guides for build and deploy

### 📚 Documentation
- Moved all `.md` files to `docs/` folder
- Created `docs/changelog/` directory
- Updated `README.md` with new links
- Added Changelog section to main README

---

## [2.0.0] - 2025-11-01

### ✨ Added

#### 🏗️ MVC Architecture
- **Complete MVC Architecture Implementation** (Laravel-inspired)
  - Created BaseController for all controllers
  - Separated Controllers: PageController, ArchiveController, SingleController
  - Laravel-like View and Template system
  - Support for Partial Views and Components

#### 🔄 Routing System
- **Automatic Routing System**
  - Route Registration for all templates
  - Custom Post Types support
  - Taxonomies support
  - Automatic 404 error handling

#### 📁 File Structure
- **Complete Migration to MVC Structure**
  - Moved all templates to `views/`
  - Created partial views in `views/partials/`
  - Reorganized asset files
  - Separated concerns in `app/`

### 🚀 PageSpeed Optimizations

#### Critical Path Optimization
- **47% Reduction in Critical Path Latency**
  - Converted blocking CSS to non-blocking
  - Inlined Critical CSS
  - Lazy loading for above-the-fold content
  - Preload for critical fonts

#### JavaScript Optimization
- **100% Elimination of Forced Reflows**
  - Optimized DOM manipulation
  - Used DocumentFragment
  - Batch DOM updates
  - requestAnimationFrame for animations

#### YouTube Embed Optimization
- **90% Reduction in Initial Load**
  - YouTube Facade for preview
  - Load on click instead of autoload
  - Reduced from 700KB to 70KB
  - Improved LCP and TBT

#### CSS Optimization
- **Removed Block Library CSS (13.8 KB)**
  - Disabled Gutenberg styles
  - Removed unused CSS
  - Minification and compression
  - Critical CSS extraction

### 🔧 SEO Improvements

#### Rank Math Integration
- **Schema and JSON-LD Optimization**
  - Complete Schema Types customization
  - Automatic Breadcrumbs
  - Canonical URLs
  - Robots Meta Tags
  - FAQ Schema optimization

#### Redirects and URL Management
- **Automatic SEO Redirects**
  - Fixed Duplicate Content with Trailing Slash
  - 301 Redirects for all Post Types
  - Taxonomies support
  - Crawl Budget optimization

### 🌍 GeoLocation

#### IP-based Location Detection
- **Geographic Location Detection System**
  - Country detection from IP
  - Access restriction (Iran only)
  - Multiple API Provider support
  - Automatic fallback on error
  - Helper functions: `xpay_is_user_from_iran()`

### 📚 Documentation

#### Developer Guides
- **Comprehensive Developer Documentation**
  - DEVELOPER-GUIDE.md: Template Addition Guide
  - MVC-ARCHITECTURE.md: Architecture and Structure
  - MIGRATION-GUIDE.md: Migration Guide
  - DEPLOYMENT.md: Deployment Guide
  - PAGESPEED-DEVELOPER-GUIDE.md: PageSpeed Guide
  - GIT-FUNCTIONS-GUIDE.md: PowerShell Functions

### 🔄 GitHub Actions

#### CI/CD Pipeline
- **Automatic Deployment**
  - Workflow for Staging
  - Workflow for Production
  - Pull Request automation
  - Error handling and notifications

---

## [1.0.0] - 2025-10-15

### ✨ Initial Release

#### Core Features
- WordPress theme installation and setup
- Initial architecture implementation
- Advanced Custom Fields integration
- Initial Rank Math configuration
- Docker environment setup

---

## Changelog Writing Guidelines

### Types of Changes
- **✨ Added**: New features
- **🔧 Changed**: Changes in existing functionality
- **⚠️ Deprecated**: Features that will be removed soon
- **🗑️ Removed**: Removed features
- **🐛 Fixed**: Bug fixes
- **🔒 Security**: Security fixes
- **🚀 Performance**: Performance improvements
- **📚 Documentation**: Documentation changes

---

**Maintained by:** XPay Development Team  
**Last Updated:** 2025-12-07
