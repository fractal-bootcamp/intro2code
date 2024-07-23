# Tutorial: Building a Real-time Group Chat App with Express.js, Socket.IO, and SQLite


## Prep:

#### Programming Languages:
- [How does the computer actually do things?](https://www.youtube.com/watch?v=Z5JC9Ve1sfI&t=417s)
- [Computer science explained in 17 min](https://www.youtube.com/watch?v=CxGSnA-RTsA)

#### Databases:
- Databases: [What is SQL?](https://www.youtube.com/watch?v=zsjvFFKOm3c)
- 7 Database Paradigms: [Video](https://www.youtube.com/watch?v=W2Z7fbCLSTw)


## Step 1: Set up the project

1. Create a new directory for your project:
   ```
   mkdir chat-app
   cd chat-app
   ```

2. Initialize a new Node.js project:
   ```
   npm init -y
   ```

3. Install the required dependencies:
   ```
   npm install express socket.io sqlite3
   ```

## Step 2: Create the server-side code

1. Create a new file named `server.js` in the project root directory.

2. Copy and paste the following code into `server.js`:

   ```javascript
   const express = require('express');
   const app = express();
   const http = require('http').createServer(app);
   const io = require('socket.io')(http);
   const sqlite3 = require('sqlite3').verbose();

   // Connect to SQLite database
   const db = new sqlite3.Database('./chat.db', (err) => {
     if (err) {
       console.error('Error opening database', err);
     } else {
       console.log('Connected to the SQLite database.');
       db.run(`CREATE TABLE IF NOT EXISTS messages (
         id INTEGER PRIMARY KEY AUTOINCREMENT,
         user TEXT,
         message TEXT,
         timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
       )`);
     }
   });

   app.use(express.static('public'));

   app.get('/', (req, res) => {
     res.sendFile(__dirname + '/public/index.html');
   });

   io.on('connection', (socket) => {
     console.log('A user connected');

     // Load previous messages
     db.all('SELECT * FROM messages ORDER BY timestamp DESC LIMIT 10', (err, rows) => {
       if (err) {
         console.error('Error fetching messages', err);
       } else {
         socket.emit('load previous messages', rows.reverse());
       }
     });

     socket.on('chat message', (msg) => {
       const { user, message } = msg;
       
       // Save message to database
       db.run('INSERT INTO messages (user, message) VALUES (?, ?)', [user, message], (err) => {
         if (err) {
           console.error('Error saving message', err);
         } else {
           io.emit('chat message', msg);
         }
       });
     });

     socket.on('disconnect', () => {
       console.log('User disconnected');
     });
   });

   const PORT = process.env.PORT || 3000;
   http.listen(PORT, () => {
     console.log(`Server running on port ${PORT}`);
   });
   ```

   This code sets up an Express server with Socket.IO, connects to an SQLite database, and handles real-time message broadcasting and storage.

## Step 3: Create the client-side code

1. Create a new directory named `public` in the project root:
   ```
   mkdir public
   ```

2. Create a new file named `index.html` inside the `public` directory.

3. Copy and paste the following code into `index.html`:

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Real-time Group Chat</title>
       <style>
           body { font-family: Arial, sans-serif; margin: 0; padding: 20px; }
           #chat { height: 400px; overflow-y: scroll; border: 1px solid #ccc; padding: 10px; margin-bottom: 20px; }
           #message-form { display: flex; }
           #user-input { flex: 1; margin-right: 10px; }
           #m { flex: 3; }
       </style>
   </head>
   <body>
       <div id="chat"></div>
       <form id="message-form">
           <input id="user-input" type="text" placeholder="Your name" required>
           <input id="m" autocomplete="off" placeholder="Type a message..." required>
           <button>Send</button>
       </form>

       <script src="/socket.io/socket.io.js"></script>
       <script>
           const socket = io();
           const chatDiv = document.getElementById('chat');
           const messageForm = document.getElementById('message-form');
           const userInput = document.getElementById('user-input');
           const messageInput = document.getElementById('m');

           function appendMessage(msg) {
               const messageElement = document.createElement('p');
               messageElement.textContent = `${msg.user}: ${msg.message}`;
               chatDiv.appendChild(messageElement);
               chatDiv.scrollTop = chatDiv.scrollHeight;
           }

           socket.on('load previous messages', (messages) => {
               messages.forEach(appendMessage);
           });

           socket.on('chat message', appendMessage);

           messageForm.addEventListener('submit', (e) => {
               e.preventDefault();
               if (messageInput.value) {
                   const msg = {
                       user: userInput.value,
                       message: messageInput.value
                   };
                   socket.emit('chat message', msg);
                   messageInput.value = '';
               }
           });
       </script>
   </body>
   </html>
   ```

   This HTML file contains the structure, styling, and client-side JavaScript for the chat interface.

## Step 4: Run the application

1. Start the server by running:
   ```
   node server.js
   ```

2. Open a web browser and navigate to `http://localhost:3000`.

3. You should see the chat interface. Open multiple browser windows to simulate different users chatting.

## Step 5: Deploy the application to the World Wide Web
You can follow [this guide](https://docs.render.com/deploy-node-express-app) to deploy your app, or just ask!


## Next Steps

- Try to read and comment every single line of the code. Ask lots of questions!
- Write down 1-2 things you want to do to improve this application.
- Ask me how to build those things!
- Or ask ChatGPT to help you out!

## Explanation of key components:

1. **Server-side (server.js)**:
   - Sets up an Express server and Socket.IO for real-time communication.
   - Connects to an SQLite database to store chat messages.
   - Handles new connections, loads previous messages, and broadcasts new messages to all connected clients.

2. **Client-side (index.html)**:
   - Provides a simple chat interface with a message display area and input form.
   - Uses Socket.IO client to connect to the server and handle real-time events.
   - Displays incoming messages and allows users to send new messages.

3. **Database**:
   - Uses SQLite to persist chat messages.
   - Stores user names, messages, and timestamps.
   - Loads the 1000 most recent messages when a new user connects.

This application demonstrates the basics of creating a real-time chat system with message persistence. You can further enhance it by adding features like user authentication, private messaging, or message editing/deletion.