# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

CrossCode is a full-stack collaborative code editor application with real-time synchronization capabilities. The project consists of:

- **Client** (`/client`): React frontend application using Create React App
- **Server** (`/server`): Node.js/Express backend with Socket.IO for real-time communication

## Development Commands

### Client (Frontend)
Navigate to `/client` directory for all client commands:
```bash
cd client
npm start          # Start development server (runs on http://localhost:3000)
npm run build      # Build for production
npm test          # Run tests
```

### Server (Backend)
Navigate to `/server` directory for all server commands:
```bash
cd server
node src/index.js  # Start the server (runs on port 5000)
```

## Architecture Overview

### Real-Time Collaboration System
- **Socket.IO Integration**: Real-time code synchronization, chat, and user presence
- **Room-based Architecture**: Users join rooms by room ID for collaborative sessions
- **Event-driven Communication**: 
  - `code-change` → `code-update`: Code synchronization
  - `language-change` → `language-update`: Language switching
  - `title-change` → `title-update`: File name changes
  - Chat messaging system with `sendMessage` → `receive-message`

### Authentication & Data Models
- **User Schema** (`/server/src/models/userSchema.js`): JWT-based auth with bcrypt password hashing
- **Room Management**: Dynamic room creation/cleanup, tracks joined users
- **Database**: MongoDB with Mongoose ODM

### Code Execution System
- **External API Integration**: JDoodle API for code compilation and execution
- **Supported Languages**: C++, Python, JavaScript, C, Java, Go
- **Configuration**: Requires JDOODLE_CLIENT_ID, JDOODLE_CLIENT_SECRET, and JDOODLE_URL in environment

### Frontend Components
- **Monaco Editor**: Code editor with syntax highlighting and themes
- **Material-UI**: UI components and theming
- **React Router**: Navigation between pages (Home, Rooms, Login/Signup)
- **Device Detection**: Desktop-only experience (mobile not supported)

## Environment Configuration

### Server Environment (`/server/src/config.env`)
Required environment variables:
- `DATABASE`: MongoDB Atlas connection string  
- `PORT`: Server port (default 5000)
- `SECRET_KEY`: JWT signing secret
- `JDOODLE_CLIENT_ID`: JDoodle API client ID
- `JDOODLE_CLIENT_SECRET`: JDoodle API client secret
- `JDOODLE_URL`: JDoodle API endpoint URL

### Client Configuration
- Proxy configured to `http://localhost:5000/` for API calls
- Socket.IO client connects to `http://localhost:5000`

## Key Technical Details

### Real-time Synchronization Flow
1. User joins room via Socket.IO with room ID and username
2. New users request initial state from existing users via `request-info` → `accept-info`
3. All code/language/title changes broadcast to room participants
4. Room cleanup occurs automatically when empty (60-minute intervals)

### Code Execution Flow
1. Client sends code + language + input to `/execute` endpoint
2. Server proxies request to JDoodle API with credentials
3. Execution results returned to client for display

### Authentication Flow
1. User registration/login via `/register` and `/login` endpoints
2. JWT tokens stored in HTTP-only cookies
3. Protected routes use authentication middleware
4. User session persistence across page refreshes

## File Upload/Download
- File upload: Reads local files into Monaco editor
- File download: Exports current code with appropriate file extension
- File naming: Dynamic title system with real-time updates across users