# Class 3 - Building a Real-Time Collaborative Whiteboard App

In this tutorial, we'll create a simple web application that allows any number of users to draw on a shared whiteboard in real-time over the 🌈 INTERNET 🌈.

## Explanation

Some technologies being covered:

1. **HTML5 Canvas**: For drawing on the web page
2. **JavaScript**: To handle user interactions and communicate with the server
3. **Socket.IO**: For real-time continuous communication between the client and server
4. **Express**: A minimal web server to serve our files and handle Socket.IO integration

The application works by sending draw events from each client to the server, which then **broadcasts** these events to all other connected clients. This allows for real-time collaboration.


## Step 1: Set Up Your Project

First, create a new directory for your project and navigate into it:

```bash
mkdir collaborative-whiteboard
cd collaborative-whiteboard
```

Now, let's set up our project structure:

```bash
mkdir public
touch server.js
touch public/index.html
touch public/styles.css
touch public/main.js
```

## Step 2: Initialize Your Project and Install Dependencies

Initialize your project and install the necessary dependencies:

```bash
npm init -y
npm install express socket.io
```

## Step 3: Create the Server

Open `server.js` in your text editor and paste the following code:

```javascript
import express from 'express';
import { createServer } from 'http';
import { Server } from 'socket.io';
import { fileURLToPath } from 'url';
import { dirname, join } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

const app = express();
const server = createServer(app);
const io = new Server(server);

// Store drawing history
let drawingHistory = [];

// Serve static files from the 'public' directory
app.use(express.static(join(__dirname, 'public')));

io.on('connection', (socket) => {
  console.log('A user connected');

  // Send drawing history to the newly connected client
  socket.emit('drawing history', drawingHistory);

  socket.on('draw', (data) => {
    drawingHistory.push(data);
    socket.broadcast.emit('draw', data);
  });

  socket.on('clear', () => {
    drawingHistory = [];
    socket.broadcast.emit('clear');
  });

  socket.on('disconnect', () => {
    console.log('User disconnected');
  });
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

```

## Step 4: Create the HTML File

Open `public/index.html` and paste the following code:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Collaborative Whiteboard</title>
    <link rel="stylesheet" href="/styles.css">
</head>
<body>
    <h1>Collaborative Whiteboard</h1>
    <canvas id="whiteboard" width="800" height="600"></canvas>
    <div>
        <button id="clearBtn">Clear Whiteboard</button>
        <input type="color" id="colorPicker" value="#000000">
    </div>
    <script src="/socket.io/socket.io.js"></script>
    <script src="/main.js"></script>
</body>
</html>

```

## Step 5: Add Some Style

Open `public/styles.css` and paste the following code:

```css
body {
    font-family: Arial, sans-serif;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
}

#whiteboard {
    border: 1px solid black;
    margin-bottom: 10px;
}

button, input {
    margin: 0 5px;
}

```

## Step 6: Create the Client-Side JavaScript

Open `public/main.js` and paste the following code:

```javascript
const socket = io();

const canvas = document.getElementById('whiteboard');
const ctx = canvas.getContext('2d');
const clearBtn = document.getElementById('clearBtn');
const colorPicker = document.getElementById('colorPicker');

let isDrawing = false;
let lastX = 0;
let lastY = 0;

function startDrawing(e) {
    isDrawing = true;
    [lastX, lastY] = [e.offsetX, e.offsetY];
}

function draw(e) {
    if (!isDrawing) return;
    drawLine(lastX, lastY, e.offsetX, e.offsetY, colorPicker.value);
    socket.emit('draw', {
        x0: lastX,
        y0: lastY,
        x1: e.offsetX,
        y1: e.offsetY,
        color: colorPicker.value
    });
    [lastX, lastY] = [e.offsetX, e.offsetY];
}

function drawLine(x0, y0, x1, y1, color) {
    ctx.beginPath();
    ctx.moveTo(x0, y0);
    ctx.lineTo(x1, y1);
    ctx.strokeStyle = color;
    ctx.stroke();
}

function stopDrawing() {
    isDrawing = false;
}

canvas.addEventListener('mousedown', startDrawing);
canvas.addEventListener('mousemove', draw);
canvas.addEventListener('mouseup', stopDrawing);
canvas.addEventListener('mouseout', stopDrawing);

clearBtn.addEventListener('click', () => {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    socket.emit('clear');
});

colorPicker.addEventListener('change', () => {
    ctx.strokeStyle = colorPicker.value;
});

socket.on('draw', (data) => {
    drawLine(data.x0, data.y0, data.x1, data.y1, data.color);
});

socket.on('clear', () => {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
});

// Handle drawing history
socket.on('drawing history', (history) => {
    history.forEach(data => {
        drawLine(data.x0, data.y0, data.x1, data.y1, data.color);
    });
});

```

## Step 7: Update package.json

Open `package.json` and add the following line in the top level of the JSON object:

```json
"type": "module",
```

This tells Node.js to use ES modules, which allows us to use `import` statements in our server code.

## Step 8: Run Your Application

Now you're ready to run your application! In your terminal, run:

```bash
node server.js
```

You should see a message saying "Server running on port 3000".

## Step 9: Test Your Whiteboard

Open a web browser and go to `http://localhost:3000`. You should see your whiteboard application. Try the following:

1. Draw on the whiteboard
2. Change colors using the color picker
3. Clear the whiteboard using the "Clear Whiteboard" button
4. Open another browser window or tab with the same URL to see real-time collaboration in action

Congratulations! You've built a real-time collaborative whiteboard application!

## Step 10: Deploy on the Internet!

You can follow [this guide](https://docs.render.com/deploy-node-express-app) to deploy your app, or just ask!

## Next Steps

- Try to read and comment every single line of the code. Ask lots of questions!
- Write down 1-2 things you want to do to improve this application.
- Ask me how to build those things!
- Or ask ChatGPT to help you out!

Web development is all about experimentation and continuous learning. Don't be afraid to modify the code and see what happens!