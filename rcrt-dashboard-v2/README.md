# RCRT Dashboard v2 🚀

**A modern React frontend that transforms the RCRT experience with self-configuration, real-time visualization, and smooth 2D/3D interactions.**

## 🎯 Vision

Dashboard v2 is a **frontend-only rewrite** that uses the existing RCRT server APIs to create a completely new user experience:

- **Everything is a breadcrumb** - Configuration, layout, preferences stored as breadcrumbs
- **Real-time visualization** - Live chat bubbles, agent thinking, tool responses  
- **Self-configuring** - Reads its own settings from RCRT breadcrumbs
- **Dynamic & extensible** - New schemas automatically get visualized
- **Smooth UX** - Physics-based interactions, seamless 2D/3D transitions

## 🏗️ Architecture

### Why Frontend-Only?

The **existing RCRT server is perfect!** It already provides:

✅ **Complete API** - All breadcrumb, agent, secret operations  
✅ **Real-time SSE** - Live event streaming at `/events/stream`  
✅ **Vector Search** - Semantic search capabilities  
✅ **Subscriptions** - Advanced selector-based subscriptions  
✅ **Authentication** - JWT and ACL systems  
✅ **Performance** - High-performance Rust backend  

### New Frontend Stack

- **React 18** with concurrent features for smooth UX
- **TypeScript** for type safety across the entire app
- **Zustand + Immer** for predictable state management
- **React Three Fiber** for declarative 3D visualization
- **Framer Motion** for physics-based smooth interactions
- **TanStack Query** for intelligent data fetching + SSE integration
- **Tailwind CSS** with custom RCRT theme

## 🌟 Key Features

### Self-Configuration via RCRT Breadcrumbs
```json
{
  "schema_name": "dashboard.config.v1",
  "title": "Dashboard Configuration",
  "tags": ["dashboard:config", "workspace:system"],
  "context": {
    "default_view": "2d",
    "real_time_updates": true,
    "node_styles": {
      "chat.message.v1": {"icon": "💬", "color": "#00f5ff"},
      "agent.response.v1": {"icon": "🤖", "color": "#8a2be2"}
    }
  }
}
```

### Real-Time Visualization
- **Live Chat**: Messages appear as animated bubbles
- **Agent Thinking**: Pulsing nodes show agent reasoning
- **Tool Responses**: Connection lines animate data flow
- **Dynamic Updates**: SSE integration for instant updates

### Smooth Interactions
- **Physics Drag & Drop**: Realistic spring animations
- **Seamless 2D/3D**: Smooth transitions between views
- **Smart Filtering**: Real-time search and tag filtering
- **Responsive Design**: Works on desktop, tablet, mobile

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- Running RCRT server (existing)

### Development Setup

1. **Ensure RCRT is running**:
```bash
# In the repo root
docker-compose up rcrt db nats -d
# RCRT API available at http://localhost:8081 (docker-compose maps host 8081 -> container 8080)
```

2. **Install and start frontend**:
```bash
cd rcrt-dashboard-v2/frontend
npm install
npm run dev
# Frontend serves on http://localhost:5173
# Automatically proxies API calls to RCRT at localhost:8081
```

3. **Visit the dashboard**:
```
http://localhost:5173
```

### Production Build

```bash
cd frontend
npm run build
# Creates optimized build in dist/

# Serve with any static server
npx serve dist
```

## 📁 Project Structure

```
rcrt-dashboard-v2/
├── frontend/                   # React frontend (only directory needed!)
│   ├── src/
│   │   ├── components/         # React components
│   │   │   ├── nodes/         # 2D & 3D node components
│   │   │   ├── canvas/        # Canvas and 3D scene
│   │   │   ├── panels/        # Left/right panel components
│   │   │   └── ui/            # Reusable UI components
│   │   ├── stores/            # Zustand state stores
│   │   ├── hooks/             # Custom React hooks
│   │   ├── types/             # TypeScript type definitions
│   │   ├── utils/             # Utility functions
│   │   └── App.tsx            # Main application
│   ├── package.json
│   ├── vite.config.ts         # Vite config with RCRT API proxy
│   └── tailwind.config.js     # RCRT-themed styling
└── README.md
```

## 🔗 RCRT API Integration

### Direct API Usage
```typescript
// No backend wrapper needed - direct RCRT API calls
const breadcrumbs = await fetch('/api/breadcrumbs').then(r => r.json());
const agents = await fetch('/api/agents').then(r => r.json());

// Real-time updates via SSE
const eventSource = new EventSource('/api/events/stream');
eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  // Handle real-time breadcrumb updates
};
```

### Vite Proxy Configuration
```typescript
// vite.config.ts automatically proxies to RCRT
export default defineConfig({
  server: {
    proxy: {
      '/api': 'http://localhost:8081',  // RCRT server (host-mapped port; container listens on 8080)
      '/events': 'http://localhost:8081'
    }
  }
})
```

## 🎨 Technology Choices

### Why This Stack?

- **React Three Fiber**: Declarative 3D with React patterns
- **Framer Motion**: Physics-based animations feel natural
- **TanStack Query**: Perfect for RCRT's REST APIs + SSE
- **Zustand**: Simple state management without Redux complexity
- **Tailwind**: Rapid development with design consistency

### Performance Optimizations

- **Code Splitting**: 3D components loaded on demand
- **Virtual Scrolling**: Handle thousands of breadcrumbs
- **Optimistic Updates**: Instant UI feedback
- **Smart Caching**: TanStack Query handles data efficiently

## 🔮 Roadmap

### Phase 1: Core Foundation ✅
- [x] Project setup and build system
- [x] TypeScript definitions for RCRT types
- [x] Tailwind theme with RCRT colors
- [ ] Basic breadcrumb visualization
- [ ] Real-time SSE integration

### Phase 2: Self-Configuration
- [ ] Configuration breadcrumb schemas
- [ ] Dynamic node factory
- [ ] Workspace system
- [ ] User preferences

### Phase 3: Real-Time Features
- [ ] Chat message visualization  
- [ ] Agent thinking animations
- [ ] Tool response flows
- [ ] Connection line animations

### Phase 4: 3D & Advanced UX
- [ ] React Three Fiber 3D scene
- [ ] Physics-based drag & drop
- [ ] Smooth 2D/3D transitions
- [ ] Advanced filtering system

### Phase 5: Polish & Extensions
- [ ] Plugin architecture
- [ ] Custom themes via breadcrumbs
- [ ] Performance optimization
- [ ] Mobile responsiveness

## 🤝 Contributing

The frontend is designed to be highly extensible:

- **New Node Types**: Add schema renderers via breadcrumb config
- **Custom Workspaces**: Define themes and layouts as breadcrumbs
- **Interaction Plugins**: Extend drag/drop and selection behavior
- **Real-time Features**: Add SSE event handlers for new schemas

## 🎉 Why This Approach Works

1. **🚀 Faster Development**: No backend changes needed
2. **🔧 Leverage Existing**: RCRT server is already excellent
3. **🎯 Focus**: Concentrate on UX improvements
4. **🔄 Incremental**: Can replace existing dashboard gradually
5. **🛡️ Reliable**: Built on proven RCRT APIs

**The existing RCRT server provides everything we need - let's build an amazing frontend experience on top of it!** 

## 📄 License

MIT License - Build amazing things with RCRT!