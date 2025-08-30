# CrossCode: Real-Time Collaborative Code Editor - Technical Documentation

## Project Overview
CrossCode is a full-stack web application that enables real-time collaborative code editing. Multiple users can join coding rooms, edit code simultaneously, chat, and execute code in various programming languages.

## Technology Stack
- **Frontend**: React.js, Material-UI, Monaco Editor, Socket.IO Client
- **Backend**: Node.js, Express.js, Socket.IO Server
- **Database**: MongoDB with Mongoose ODM
- **Authentication**: JWT (JSON Web Tokens) with bcrypt
- **Code Execution**: JDoodle API
- **Real-time Communication**: WebSockets via Socket.IO

---

## Setup and Installation

### Prerequisites
- Node.js (v14 or higher)
- MongoDB (local installation or MongoDB Atlas)
- npm package manager

### Installation Steps

#### 1. Clone and Navigate
```bash
git clone <repository-url>
cd CrossCode
```

#### 2. Server Setup
```bash
cd server
npm install
```

#### 3. Client Setup  
```bash
cd ../client
npm install
```

#### 4. Environment Configuration
Create `server/src/config.env`:
```env
DATABASE=mongodb://localhost:27017/crosscode
PORT=5000
SECRET_KEY=your_jwt_secret_key_here
JDOODLE_CLIENT_ID=your_jdoodle_client_id
JDOODLE_CLIENT_SECRET=your_jdoodle_client_secret
JDOODLE_URL=https://api.jdoodle.com/v1/execute
```

#### 5. Start Services
```bash
# Terminal 1 - Start MongoDB
sudo systemctl start mongod

# Terminal 2 - Start Server
cd server
node src/index.js

# Terminal 3 - Start Client
cd client
npm start
```

### Access Points
- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:5000

---

## Frontend Implementation (React Client)

### Architecture Overview
The client follows a component-based architecture with React Router for navigation and Socket.IO for real-time communication.

### Key Components Structure
```
src/
├── components/
│   ├── Editor/
│   │   └── myEditor.jsx          # Monaco Editor wrapper
│   ├── ChatFeature/
│   │   ├── Messages/             # Chat message display
│   │   └── Input/                # Chat input component
│   ├── Home/                     # Landing page
│   ├── Room/                     # Room management
│   ├── InsideRoom/               # Main collaboration interface
│   └── Login/SignUp/             # Authentication components
├── pages/                        # Route components
└── App.js                       # Main app router
```

### Real-Time Communication Implementation

#### Socket.IO Client Connection
```javascript
// App.js - Socket initialization
useEffect(() => {
    const s = io("http://localhost:5000");
    setSocket(s);
    return () => s.disconnect();
}, []);
```

#### Code Synchronization
```javascript
// myEditor.jsx - Real-time code sync
useEffect(() => {
    socket.emit("code-change", editorCode);
}, [editorCode]);

socket.on("code-update", (data) => {
    setValue(data);
});
```

### Monaco Editor Integration
- **Language Support**: C++, Python, JavaScript, C, Java, Go
- **Theme Support**: Light/Dark mode switching
- **Features**: Syntax highlighting, auto-completion, font size adjustment

### State Management
Uses React hooks for local state management:
- `useState` for component state
- `useEffect` for side effects and real-time listeners
- Props drilling for sharing data between components

### Key Features Implementation

#### 1. File Operations
```javascript
// File Download
const downloadCode = () => {
    fileDownload(editorCode, `${title}.${languageExtension}`);
};

// File Upload
const showFile = async (e) => {
    const reader = new FileReader();
    reader.onload = async (e) => {
        const text = e.target.result;
        setValue(text);
    };
    reader.readAsText(e.target.files[0]);
};
```

#### 2. Room Management
```javascript
// Join Room
socket.emit("join-room", { id, nameOfUser });

// Check Room Existence  
socket.emit("room-id", roomId);
socket.on("room-check", (exists) => {
    if (!exists) setValid(false);
});
```

---

## Backend Implementation (Node.js Server)

### Server Architecture
```
src/
├── index.js                 # Main server entry point
├── db/
│   └── connec.js           # MongoDB connection
├── models/
│   ├── userSchema.js       # User data model
│   ├── roomSchema.js       # Room data model
│   └── message.js          # Message data model
├── router/
│   └── auth.js             # Authentication routes
├── middleware/
│   └── authenticate.js     # JWT middleware
└── config.env              # Environment variables
```

### Express.js Server Setup
```javascript
// index.js - Server initialization
const express = require("express");
const http = require("http");
const socketIo = require("socket.io");

const app = express();
const httpServer = new http.Server(app);

// CORS configuration
app.use(cors({
    origin: "http://localhost:3000",
    credentials: true
}));

// Middleware
app.use(express.json());
app.use(cookieParser());

// Socket.IO setup
const io = socketIo(httpServer, {
    cors: { origin: "http://localhost:3000" }
});
```

### Socket.IO Real-Time Events

#### Room Management
```javascript
// User joins room
socket.on("join-room", (msg) => {
    socket.room = msg.id;
    socket.join(msg.id);
    
    // Notify existing users
    socket.broadcast.to(socket.room).emit("receive-message", {
        sender: "admin",
        text: `${msg.nameOfUser} has joined!`
    });
    
    // Send user count
    let room = io.sockets.adapter.rooms.get(socket.room);
    io.sockets.in(socket.room).emit("joined-users", room.size);
});
```

#### Code Synchronization
```javascript
// Broadcast code changes
socket.on("code-change", (msg) => {
    socket.broadcast.to(socket.room).emit("code-update", msg);
});

// Handle language changes
socket.on("language-change", (msg) => {
    io.sockets.in(socket.room).emit("language-update", msg);
});
```

#### Room Cleanup System
```javascript
// Automatic room cleanup
function removingRooms() {
    if (removeRooms.length != 0) {
        for (let i = 0; i < removeRooms.length; i++) {
            if (io.sockets.adapter.rooms[removeRooms[i]] === undefined) {
                rooms = rooms.filter(item => item !== removeRooms[i]);
            }
        }
    }
    removeRooms.splice(0, removeRooms.length);
    setTimeout(removingRooms, 60 * 60 * 1000); // Every hour
}
```

### Code Execution System
```javascript
// JDoodle API integration
app.post("/execute", async (req, res) => {
    const { script, language, stdin, versionIndex } = req.body;
    
    const response = await axios({
        method: "POST",
        url: process.env.JDOODLE_URL,
        data: {
            script: script,
            stdin: stdin,
            language: language,
            versionIndex: versionIndex,
            clientId: process.env.JDOODLE_CLIENT_ID,
            clientSecret: process.env.JDOODLE_CLIENT_SECRET,
        }
    });
    
    res.json(response.data);
});
```

---

## Database Implementation (MongoDB)

### Connection Setup
```javascript
// db/connec.js - MongoDB connection
const mongoose = require("mongoose");

const DB = process.env.DATABASE;

mongoose.connect(DB, {
    useNewUrlParser: true,
    useCreateIndex: true,
    useUnifiedTopology: true,
    useFindAndModify: false,
}).then(() => {
    console.log("Connection Successful");
}).catch((err) => console.log(err));
```

### Data Models

#### User Schema
```javascript
// models/userSchema.js
const userSchema = new mongoose.Schema({
    userName: {
        type: String,
        required: true
    },
    email: {
        type: String,
        required: true
    },
    password: {
        type: String,
        required: true
    },
    roomsJoinedByMe: [{
        type: mongoose.Schema.Types.ObjectId,
        ref: "ROOM"
    }],
    roomsCreatedByMe: [{
        type: mongoose.Schema.Types.ObjectId,
        ref: "ROOM"
    }],
    tokens: [{
        token: {
            type: String,
            required: true
        }
    }]
});

// Password hashing middleware
userSchema.pre('save', async function (next) {
    if (this.isModified('password')) {
        this.password = await bcrypt.hash(this.password, 12);
    }
    next();
});

// JWT token generation
userSchema.methods.generateAuthToken = async function() {
    try {
        let currToken = jwt.sign({ _id: this._id}, process.env.SECRET_KEY);
        this.tokens = this.tokens.concat({ token: currToken });
        await this.save();
        return currToken;
    } catch (error) {
        console.log(error);
    }
}
```

### Authentication Implementation

#### User Registration
```javascript
// router/auth.js - Registration endpoint
router.post('/register', async (req, res) => {
    const { userName, email, password } = req.body;
    
    if (!userName || !email || !password) {
        return res.status(422).json({ error: "Please fill all required fields" });
    }
    
    try {
        const userExists = await User.findOne({ email: email });
        
        if (userExists) {
            return res.status(422).json({ error: "User already exists" });
        }
        
        const user = new User({ userName, email, password });
        const userRegistered = await user.save();
        
        if (userRegistered) {
            res.status(201).json({ message: "User registered successfully" });
        }
    } catch (error) {
        console.log(error);
    }
});
```

#### User Login with JWT
```javascript
// Login endpoint
router.post('/login', async (req, res) => {
    const { userName, password } = req.body;
    
    try {
        const userExists = await User.findOne({ userName: userName });
        
        if (userExists) {
            const isMatch = await bcrypt.compare(password, userExists.password);
            
            if (isMatch) {
                const token = await userExists.generateAuthToken();
                
                res.cookie("jwtToken", token, { 
                    expires: new Date(Date.now() + 25892000000),
                    httpOnly: true
                });
                
                res.json({ message: "Logged In successfully" });
            } else {
                res.status(400).json({ error: "Invalid credentials" });   
            }
        } else {
            res.status(400).json({ error: "Invalid credentials" });
        }
    } catch (error) {
        console.log(error);
    }
});
```

#### JWT Middleware
```javascript
// middleware/authenticate.js
const jwt = require('jsonwebtoken');
const User = require('../models/userSchema');

const authenticate = async (req, res, next) => {
    try {
        const token = req.cookies.jwtToken;
        const verifyToken = jwt.verify(token, process.env.SECRET_KEY);
        
        const rootUser = await User.findOne({
            _id: verifyToken._id,
            "tokens.token": token
        });
        
        if (!rootUser) {
            throw new Error('User not found');
        }
        
        req.token = token;
        req.rootUser = rootUser;
        req.userID = rootUser._id;
        
        next();
    } catch (error) {
        res.status(401).send('Unauthorized: No token provided');
    }
}
```

---

## Database Operations and Access Patterns

### Creating Documents
```javascript
// Create new user
const user = new User({ userName, email, password });
await user.save();

// Create with validation
const newRoom = new Room({
    roomName: "Code Session",
    createdBy: userId,
    participants: [userId]
});
await newRoom.save();
```

### Reading Documents
```javascript
// Find single document
const user = await User.findOne({ email: userEmail });

// Find with population (JOIN-like operation)
const user = await User.findById(userId)
    .populate('roomsJoinedByMe')
    .populate('roomsCreatedByMe');

// Find with conditions
const activeRooms = await Room.find({ 
    isActive: true,
    createdAt: { $gte: new Date(Date.now() - 24*60*60*1000) }
});
```

### Updating Documents
```javascript
// Update single field
await User.findByIdAndUpdate(userId, { 
    $push: { roomsJoinedByMe: roomId }
});

// Update multiple fields
await User.updateOne(
    { _id: userId },
    { 
        $set: { lastActive: new Date() },
        $inc: { sessionCount: 1 }
    }
);
```

### Deleting Documents
```javascript
// Remove document
await User.findByIdAndDelete(userId);

// Remove with conditions
await Room.deleteMany({ 
    isActive: false,
    createdAt: { $lt: new Date(Date.now() - 7*24*60*60*1000) }
});
```

### Complex Queries
```javascript
// Aggregation pipeline
const userStats = await User.aggregate([
    { $match: { isActive: true } },
    { $group: {
        _id: null,
        totalUsers: { $sum: 1 },
        avgRoomsPerUser: { $avg: { $size: "$roomsJoinedByMe" }}
    }}
]);

// Text search
const searchResults = await Room.find({
    $text: { $search: searchTerm }
}).sort({ score: { $meta: 'textScore' }});
```

---

## Key Interview Topics Covered

### 1. **Full-Stack Development**
- React.js frontend with real-time updates
- Node.js/Express backend with RESTful APIs
- WebSocket communication via Socket.IO

### 2. **Database Design**
- MongoDB schema design with Mongoose ODM
- Relationships using ObjectId references
- Authentication with JWT tokens

### 3. **Real-Time Systems**
- WebSocket implementation for live collaboration
- Event-driven architecture
- State synchronization across multiple clients

### 4. **Security**
- Password hashing with bcrypt
- JWT-based authentication
- CORS configuration
- Input validation

### 5. **System Architecture**
- Separation of concerns (client/server/database)
- Environment configuration management
- Error handling and logging

### 6. **Development Operations**
- Local development setup
- Package management with npm
- Process management and monitoring

This technical documentation provides comprehensive coverage of the CrossCode implementation, making you well-prepared to discuss any aspect of the project in technical interviews.