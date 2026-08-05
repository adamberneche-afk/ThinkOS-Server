# RCRT Chat Browser Extension

Your AI assistant powered by Right Context Right Time, accessible directly from your browser.

## Features

- 🤖 **AI-Powered Chat**: Interact with intelligent agents using natural language
- 🛠️ **Tool Integration**: Execute tools like file storage, web search, and more
- 📡 **Real-time Updates**: Live connection via Server-Sent Events
- 🔐 **Secure**: JWT authentication with the RCRT system
- 🎨 **Modern UI**: Clean, dark theme interface with smooth animations
- ⚡ **Fast & Responsive**: Built with React and TypeScript

## Quick Start

1. **Ensure RCRT is running**:
   ```bash
   docker compose up -d
   ```

2. **Add your OpenRouter API key**:
   ```bash
   export OPENROUTER_API_KEY=sk-or-v1-xxx
   node add-openrouter-key.js
   docker compose restart tools-runner
   ```

3. **Build and install**:
   ```bash
   npm install
   npm run build
   ```

4. **Load in Chrome/Edge**:
   - Go to `chrome://extensions/`
   - Enable "Developer mode"
   - Click "Load unpacked"
   - Select the `dist` folder

5. **Start chatting!** Click the extension icon

## Development

### Build for Development
```bash
npm run dev    # Watch mode with hot reload
npm run build  # Production build
```

### Project Structure
```
extension/
├── src/
│   ├── popup/          # Main chat interface
│   │   ├── RCRTChat.tsx    # Primary chat component
│   │   └── Popup.tsx       # Root component
│   ├── lib/            # RCRT client library
│   └── background/     # Service worker
├── manifest.json       # Extension manifest
└── dist/              # Built extension
```

### Key Components

- **RCRTChat**: Main chat interface with RCRT integration
- **RCRTExtensionClient**: Handles authentication and API calls
- **SSE Integration**: Real-time event streaming

## Architecture

The extension connects to your local RCRT instance and:
1. Creates `chat.message.v1` breadcrumbs for user input
2. Subscribes to `agent.response.v1` events via SSE
3. Shows tool invocations and responses in real-time
4. Maintains session context for conversations

## UI Design

The extension uses a modern dark theme:
- **Background**: Gradient from `gray-900` to `gray-800`
- **Primary**: Teal accents (`teal-400`, `teal-600`)
- **Messages**: User (teal), Assistant (gray), System (gray/red), Tools (purple)
- **Typography**: Clean, readable text with proper contrast

## Troubleshooting

### Common Issues

- **Connection Failed**: Ensure RCRT is running on `localhost:8081`
- **No Response**: Check if agent-runner is active
- **Tool Errors**: Verify tools-runner has the API keys

## License

Part of the RCRT ecosystem.