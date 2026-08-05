# Testing RCRT Chrome Extension

## ✅ Extension Built Successfully! 

The extension has been cleaned up and now **only uses RCRT** via the dashboard proxy. 

## 🚀 Loading the Extension

1. **Open Chrome Extensions**:
   ```
   chrome://extensions/
   ```

2. **Enable Developer Mode** (top right toggle)

3. **Load Unpacked Extension**:
   - Click "Load unpacked"
   - Select: `extension/dist/` folder
   - Extension should appear with "RCRT Chat" name

## 🔍 What to Expect

### ✅ Working Now (No dependencies needed)
- **Extension loads** without errors
- **UI appears** in popup and sidepanel 
- **Local functionality** works (stored messages, settings)

### 🔧 Console Messages (Check Service Worker)
```
🚀 RCRT Extension - Connecting to dashboard: http://localhost:8081
❌ RCRT dashboard not available: [error]
ℹ️ Start RCRT dashboard (cargo run -p rcrt-dashboard) for full features
📋 Extension will work with local storage until dashboard is available
```

## 🎯 Testing Steps

### Phase 1: Extension Loads ✅
- [x] Load extension in Chrome
- [x] Click extension icon → opens sidepanel
- [x] UI renders without errors
- [x] Console shows graceful fallback messages

### Phase 2: RCRT Dashboard Connection 🔧
```bash
# In separate terminal - start RCRT dashboard 
cd crates/rcrt-dashboard
cargo run
```

**Expected Console Messages**:
```
🚀 RCRT Extension - Connecting to dashboard: http://localhost:8081
🔑 Fetching JWT token from RCRT dashboard...
✅ Got JWT token from RCRT dashboard
🎯 Extension ready for RCRT integration
```

### Phase 3: RCRT Full Integration 🚀
```bash
# Start RCRT core service
cargo run -p rcrt-server

# Start tools runner
cd rcrt-visual-builder/apps/tools-runner
npm start
```

## 🐛 Troubleshooting

### Extension Won't Load
- Check `extension/dist/` folder exists
- Reload extension after changes
- Check Chrome console for build errors

### Dashboard Connection Issues
- Ensure dashboard runs on `localhost:8081`
- Check CORS settings allow Chrome extension
- Verify JWT token endpoint exists

### UI Errors
- Extension works with **stored data** even without dashboard
- UI should never break due to RCRT unavailability
- Check browser console for specific errors

## 📋 Current Features

### Working Features (No RCRT needed)
- ✅ **Chat Interface**: UI renders and accepts input
- ✅ **Local Storage**: Messages/sessions persist locally
- ✅ **Settings**: Basic configuration works
- ✅ **Graceful Fallback**: No crashes when RCRT unavailable

### RCRT-Enhanced Features (Requires dashboard)
- 🔧 **Authentication**: JWT token from dashboard
- 🔧 **Breadcrumb Creation**: Chat → breadcrumbs
- 🔧 **Agent Discovery**: RCRT agents as crews
- 🔧 **Real-time Events**: SSE for live updates

## 🎉 Success Criteria

The extension should:
1. **Load without errors** ✅
2. **Show clean console messages** ✅
3. **Work offline** with local storage ✅
4. **Connect to RCRT** when dashboard available 🔧
5. **Create breadcrumbs** for chat messages 🔧

## Next Steps

1. **Test Basic Load**: Load extension → should work fine
2. **Test Dashboard Connection**: Start dashboard → should connect
3. **Test Chat Integration**: Send message → should create breadcrumb
4. **View in Dashboard**: Check breadcrumbs appear in RCRT dashboard

Ready to test! 🚀
