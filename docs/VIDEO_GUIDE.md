# Video Tutorial Integration Guide

## Overview
This guide explains how to integrate video tutorials for each app in the Infra Control Center.

## Video Sources

### Supported Platforms
1. **YouTube** (Recommended)
2. **Vimeo**
3. **Self-hosted videos**
4. **Loom**
5. **Wistia**

## Adding Video Tutorials

### Step 1: Prepare Your Videos

Create tutorial videos for each app covering:
- Overview and purpose
- How to access the tool
- Key features walkthrough
- Common use cases
- Troubleshooting tips

**Recommended specifications:**
- Duration: 3-5 minutes per app
- Resolution: 1080p (Full HD)
- Format: MP4 (for self-hosted)
- Audio: Clear voiceover with background music

### Step 2: Upload Videos

#### Option 1: YouTube (Recommended)

1. **Upload to YouTube:**
   - Go to YouTube Studio
   - Click "Create" → "Upload video"
   - Upload your tutorial video
   - Fill in title: "[App Name] Tutorial - Infra Control Center"
   - Add description with key points
   - Set visibility (Public/Unlisted)

2. **Get Embed Code:**
   - Go to video page
   - Click "Share"
   - Click "Embed"
   - Copy the iframe src URL (e.g., `https://www.youtube.com/embed/VIDEO_ID`)

3. **Update Code:**
   In `index.html`, find the `videoUrls` object and update:
   ```javascript
   const videoUrls = {
       'metrics': 'https://www.youtube.com/embed/YOUR_VIDEO_ID',
       'acl': 'https://www.youtube.com/embed/YOUR_VIDEO_ID',
       // ... rest of the apps
   };
   ```

#### Option 2: Vimeo

1. **Upload to Vimeo:**
   - Upload video to your Vimeo account
   - Configure privacy settings
   - Get video ID from URL

2. **Update Code:**
   ```javascript
   const videoUrls = {
       'metrics': 'https://player.vimeo.com/video/YOUR_VIDEO_ID',
       'acl': 'https://player.vimeo.com/video/YOUR_VIDEO_ID',
   };
   ```

#### Option 3: Self-Hosted

1. **Host Video Files:**
   - Upload MP4 files to your server
   - Or use cloud storage (AWS S3, Google Cloud Storage)

2. **Update Modal HTML:**
   Replace iframe with video tag in `index.html`:
   ```javascript
   function openVideo(appId, title) {
       const modal = document.getElementById('videoModal');
       const videoContainer = document.querySelector('.video-container');
       const videoTitle = document.getElementById('videoTitle');
       
       videoTitle.textContent = title;
       
       // Replace iframe with video tag
       videoContainer.innerHTML = `
           <video controls autoplay>
               <source src="${videoUrls[appId]}" type="video/mp4">
               Your browser does not support the video tag.
           </video>
       `;
       
       modal.classList.add('active');
   }
   ```

## Video Recording Tips

### Tools for Screen Recording

**Windows:**
- OBS Studio (Free)
- Camtasia (Paid)
- Screencast-O-Matic
- Windows Game Bar (Built-in: Win + G)

**Mac:**
- QuickTime Player (Built-in)
- OBS Studio (Free)
- ScreenFlow (Paid)
- Loom

**Cross-platform:**
- OBS Studio
- Loom
- Zoom (record meetings)

### Recording Best Practices

1. **Preparation:**
   - Write a script or outline
   - Test audio levels
   - Clean up desktop/interface
   - Close unnecessary applications
   - Use highest resolution possible

2. **During Recording:**
   - Speak clearly and at moderate pace
   - Pause between sections (easier to edit)
   - Use mouse clicks audibly
   - Highlight important areas
   - Keep cursor movement smooth

3. **Content Structure:**
   ```
   Introduction (15 seconds)
   - App name and purpose
   
   Navigation (30 seconds)
   - How to access the app
   - Main interface overview
   
   Features Demo (2-3 minutes)
   - Key feature #1
   - Key feature #2
   - Key feature #3
   
   Use Case Example (1 minute)
   - Real-world scenario
   - Step-by-step walkthrough
   
   Conclusion (15 seconds)
   - Quick recap
   - Call to action
   ```

4. **Post-Production:**
   - Trim silence and mistakes
   - Add intro/outro cards
   - Add captions/subtitles
   - Add background music (low volume)
   - Export in 1080p

## Example Video URLs Object

```javascript
// In index.html
const videoUrls = {
    // User-accessible apps
    'metrics': 'https://www.youtube.com/embed/ABC123XYZ',
    'acl': 'https://www.youtube.com/embed/DEF456ABC',
    'mac': 'https://www.youtube.com/embed/GHI789DEF',
    'config': 'https://www.youtube.com/embed/JKL012GHI',
    'topology': 'https://www.youtube.com/embed/MNO345JKL',
    'scripts': 'https://www.youtube.com/embed/PQR678MNO',
    
    // Admin-only apps
    'ksi': 'https://www.youtube.com/embed/STU901PQR',
    'cert': 'https://www.youtube.com/embed/VWX234STU',
    'eip': 'https://www.youtube.com/embed/YZA567VWX'
};
```

## Advanced Features

### Auto-play on Open
Videos already auto-play when modal opens. To disable:
```javascript
// Remove '&autoplay=1' from YouTube URL
'metrics': 'https://www.youtube.com/embed/ABC123XYZ', // no autoplay
```

### Add Timestamps
For longer videos, add chapters:

```javascript
function openVideo(appId, title) {
    const modal = document.getElementById('videoModal');
    const videoTitle = document.getElementById('videoTitle');
    const videoPlayer = document.getElementById('videoPlayer');
    
    videoTitle.textContent = title;
    
    // Add timestamp to URL for specific starting point
    const startTime = 0; // seconds
    videoPlayer.src = `${videoUrls[appId]}?start=${startTime}`;
    
    modal.classList.add('active');
}
```

### Video Playlist
Create a playlist for all tutorials:

```javascript
// Add playlist button to header
<button onclick="openPlaylist()">
    <i class="fas fa-list"></i> All Tutorials
</button>

// Function to open full playlist
function openPlaylist() {
    window.open('https://www.youtube.com/playlist?list=YOUR_PLAYLIST_ID', '_blank');
}
```

### Analytics Tracking

Track video views:

```javascript
function openVideo(appId, title) {
    // ... existing code ...
    
    // Track with Google Analytics
    if (typeof gtag !== 'undefined') {
        gtag('event', 'video_open', {
            'event_category': 'Tutorial',
            'event_label': appId,
            'value': title
        });
    }
    
    // Or use custom analytics
    fetch('/api/analytics/video-view', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            app: appId,
            timestamp: new Date().toISOString()
        })
    });
}
```

## Video Management Dashboard

Create a management interface for admins to update video URLs without touching code:

```javascript
// admin-videos.html (new page)
const videoManager = {
    async loadVideos() {
        // Load from backend
        const response = await fetch('/api/videos');
        const videos = await response.json();
        return videos;
    },
    
    async updateVideo(appId, newUrl) {
        // Update in backend
        await fetch('/api/videos', {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ appId, url: newUrl })
        });
    }
};
```

## Accessibility

Make videos accessible:

1. **Captions:**
   - Add closed captions to all videos
   - Use YouTube's auto-caption feature
   - Edit for accuracy

2. **Transcripts:**
   - Provide text transcripts
   - Add below video in modal

3. **Keyboard Navigation:**
   - Already supported (Escape to close)
   - Space to play/pause (native browser)

## Testing Checklist

Before going live, test each video:

- [ ] Video loads correctly
- [ ] Audio is clear
- [ ] Resolution is appropriate
- [ ] Modal opens/closes properly
- [ ] Works on desktop browsers
- [ ] Works on mobile devices
- [ ] Closed captions available
- [ ] No broken links
- [ ] Playback is smooth
- [ ] Controls are accessible

## Sample Video Script Template

```
[INTRO - 15 seconds]
"Hi! In this quick tutorial, I'll show you how to use the [App Name] 
in the Infra Control Center. This tool helps you [brief purpose]."

[ACCESS - 15 seconds]
"To access [App Name], simply click on the [App Name] card from the 
main dashboard. You'll see it under [category/section]."

[FEATURE 1 - 45 seconds]
"Let's start with [Feature 1]. Here's how it works...
[Demo with clear steps]
This is useful when you need to [use case]."

[FEATURE 2 - 45 seconds]
"Next, let's look at [Feature 2]...
[Demo with clear steps]
You can use this to [use case]."

[FEATURE 3 - 45 seconds]
"Finally, [Feature 3] allows you to...
[Demo with clear steps]
This comes in handy for [use case]."

[EXAMPLE - 60 seconds]
"Let's see a real-world example. Imagine you need to [scenario]...
[Step-by-step walkthrough]
And there you have it!"

[CONCLUSION - 15 seconds]
"That's it for [App Name]! If you have questions, check out our 
documentation or contact support. Thanks for watching!"
```

## Troubleshooting

### Video Won't Load
- Check URL is correct
- Verify embed permissions
- Test URL in browser directly
- Check CORS settings for self-hosted

### Audio Issues
- Re-record with better microphone
- Use audio editing software to enhance
- Normalize audio levels

### Low Quality
- Re-record at higher resolution
- Use better screen recording settings
- Compress without losing quality

## Alternative: Video Hosting Services

### Wistia
- Professional video hosting
- Advanced analytics
- Custom branding
- Email capture

### Vidyard
- Video hosting and analytics
- Personalized video
- Integration with CRM

### Loom
- Quick screen recordings
- Easy sharing
- Built-in editing
- Free tier available

---

For questions or issues, contact your development team or refer to the main [README.md](../README.md).
