# Changelog

All notable changes to the Infra Control Center project will be documented in this file.

## [2.0.0] - 2026-02-19

### ✨ Major New Features

#### User/Admin View Toggle
- **Two-Mode Interface**: Toggle between User and Admin views
- **Role-Based Access Control**: Admin-only apps are locked for users
- **Visual Indicators**: Lock icons and badges for restricted content
- **Seamless Switching**: Switch roles from header toggle or profile dropdown

#### Video Tutorial Integration
- **Tutorial Buttons**: Each app card now has a "Tutorial" button
- **Video Modal**: Beautiful modal popup for video playback
- **YouTube/Vimeo Support**: Easy integration with popular platforms
- **Self-Hosted Option**: Support for custom video hosting
- **Keyboard Controls**: ESC to close, native player controls

#### Admin-Only Applications
Added 3 new admin-only apps:
- **KSI Generation**: Keystore Infrastructure management
- **Certificate Management**: SSL/TLS certificate lifecycle
- **Elastic IP Management**: Cloud IP address management

#### Enhanced Navigation
- **Fixed Back Button**: "Back to Dashboard" now works correctly
- **Proper Linking**: All navigation between pages functional
- **Profile Dropdown**: User menu with role switching and logout

### 🎨 Design Improvements

#### User Interface
- **User Profile Section**: Avatar, name, and role display
- **View Toggle Buttons**: Prominent user/admin mode switcher
- **Lock Icons**: Visual feedback for restricted content
- **Badge System**: Different badge colors for user/admin apps

#### Animations & Effects
- **Card Filtering**: Smooth show/hide when switching views
- **Modal Transitions**: Elegant video modal entrance/exit
- **3D Card Effects**: Enhanced mouse-responsive animations
- **Scroll Reveals**: Improved timing and smoothness

### 📚 Documentation

#### New Guides
- **VIDEO_GUIDE.md**: Complete video integration documentation
  - Recording best practices
  - Upload instructions (YouTube/Vimeo/Self-hosted)
  - Script templates
  - Accessibility guidelines
  
#### Updated Documentation
- **README.md**: Updated feature list and app count
- **SETUP_GUIDE.md**: Enhanced with new features
- **API.md**: Added role-based access examples

### 🔧 Technical Changes

#### JavaScript Enhancements
- `switchView()`: Function to toggle user/admin modes
- `openVideo()`: Video modal management
- `closeVideo()`: Clean modal closure with iframe reset
- Role-based card filtering logic
- Dynamic app count updates

#### HTML Structure
- Video modal component
- User profile dropdown
- View toggle controls
- Admin-only card templates

#### CSS Additions
- Video modal styles
- Profile dropdown styling
- Admin-lock indicator styles
- View toggle button states

### 🐛 Bug Fixes

- **Fixed**: Back to Dashboard button not working (changed from `onclick` to proper `href`)
- **Fixed**: Navigation between index.html and ai-assistant.html
- **Fixed**: Card visibility logic when switching views
- **Fixed**: Modal close on outside click
- **Fixed**: Proper iframe cleanup when closing video modal

### 📊 Statistics

- **Total Apps**: 9 (6 user + 3 admin)
- **Video Tutorials**: 9 (one per app)
- **Documentation**: 5 guides (README, SETUP, API, CUSTOMIZATION, DEPLOYMENT, VIDEO)
- **Total Size**: ~37KB (compressed)
- **Code Lines**: ~700 lines HTML/CSS/JS

## [1.0.0] - 2026-02-19

### Initial Release

#### Core Features
- Modern glassmorphism design
- 6 network automation tools
- AI Assistant interface
- 3D card effects
- Scroll-reveal animations
- Responsive layout

#### Applications
1. Operational Metrics
2. ACL Live Checker
3. MAC Address Management
4. Config Validation
5. Network Topology
6. Automation Scripts

#### Documentation
- Complete README
- Setup guide
- API integration docs
- Customization guide
- Deployment guide

---

## Roadmap

### Version 2.1.0 (Planned)
- [ ] Real backend API integration
- [ ] User authentication system
- [ ] Live data visualization charts
- [ ] Notification system
- [ ] Search functionality
- [ ] Dark/Light theme toggle
- [ ] Multi-language support

### Version 2.2.0 (Planned)
- [ ] Mobile app (React Native)
- [ ] Desktop app (Electron)
- [ ] Real-time collaboration
- [ ] Advanced analytics dashboard
- [ ] Custom reporting tools
- [ ] Workflow automation builder

### Version 3.0.0 (Future)
- [ ] AI-powered network optimization
- [ ] Predictive maintenance
- [ ] Machine learning insights
- [ ] Integration marketplace
- [ ] White-label options
- [ ] Enterprise SSO

---

## Migration Guides

### Upgrading from 1.0.0 to 2.0.0

1. **Backup your current files**
2. **Replace HTML files** with new versions
3. **Update video URLs** in `index.html`:
   ```javascript
   const videoUrls = {
       'metrics': 'YOUR_VIDEO_URL',
       // ... add your URLs
   };
   ```
4. **Test role switching** functionality
5. **Verify navigation** between pages

### Breaking Changes
- None (fully backward compatible)

### New Dependencies
- None (still pure HTML/CSS/JS)

---

## Contributors

- Initial design and development: [@b-w-v](https://github.com/b-w-v)
- Design inspiration: Dribbble, Behance, 2026 UI trends

## License

MIT License - see [LICENSE](LICENSE) file for details

---

**Note**: This project follows [Semantic Versioning](https://semver.org/).

Format: `[MAJOR.MINOR.PATCH]`
- **MAJOR**: Incompatible API changes
- **MINOR**: Backward-compatible functionality additions
- **PATCH**: Backward-compatible bug fixes
