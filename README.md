# CrossCode - Real-Time Collaborative Code Editor

A full-stack web application for real-time collaborative code editing with chat functionality and code execution capabilities.

## Technology Stack

**Frontend:**
- React.js
- Material-UI
- Monaco Editor
- Socket.IO Client
- Axios

**Backend:**
- Node.js
- Express.js
- Socket.IO
- Mongoose
- JWT Authentication
- bcrypt

**Database:**
- MongoDB

**External API:**
- JDoodle API for code execution

## Features

- Real-time collaborative code editing
- Integrated chat system
- Multiple programming language support (C++, Python, JavaScript, C, Java, Go)
- Code execution in browser
- User authentication and authorization
- File upload and download
- Light and dark theme support

## Installation and Setup

### Prerequisites
- Node.js (v14 or higher)
- MongoDB (local installation or MongoDB Atlas)
- npm package manager

### Setup Instructions

1. Clone the repository
```bash
git clone https://github.com/RUCHITARANI/crosscode.git
cd crosscode
```

2. Install server dependencies
```bash
cd server
npm install
```

3. Install client dependencies
```bash
cd ../client
npm install
```

4. Environment Configuration
Create `server/src/config.env` with the following variables:
```
DATABASE=mongodb://localhost:27017/crosscode
PORT=5000
SECRET_KEY=your_jwt_secret_key_here
JDOODLE_CLIENT_ID=your_jdoodle_client_id
JDOODLE_CLIENT_SECRET=your_jdoodle_client_secret
JDOODLE_URL=https://api.jdoodle.com/v1/execute
```

5. Start MongoDB service
```bash
sudo systemctl start mongod
```

6. Start the server
```bash
cd server
node src/index.js
```

7. Start the client (in a new terminal)
```bash
cd client
npm start
```

## Access Points
- Frontend: http://localhost:3000
- Backend API: http://localhost:5000

## Project Structure
```
crosscode/
├── client/          # React frontend application
├── server/          # Node.js backend application
├── CLAUDE.md        # Claude Code configuration
└── TECHNICAL_DOCUMENTATION.md
```

## API Endpoints

**Authentication:**
- POST /register - User registration
- POST /login - User login
- GET /logout - User logout

**Code Execution:**
- POST /execute - Execute code using JDoodle API

## Real-time Communication

The application uses Socket.IO for real-time features including:
- Live code synchronization
- Chat messaging
- Room management
- User presence tracking

## Deployment

The project includes configuration files for deployment on Railway and Vercel platforms, both of which offer free tiers suitable for hosting this application.