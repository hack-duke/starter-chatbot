# PDF Question Helper: A Beginner's Project

Welcome to **PDF Question Helper**, a simple app that allows users to upload a PDF, extract its text, and ask questions about the document using ChatGPT! This project is designed to help beginners learn how to integrate front-end, back-end, and third-party APIs.

---

## ✨ Features

- Upload a PDF file to extract its text content.
- Ask ChatGPT questions about the PDF's content.
- Simple React frontend and Express backend setup.
- Integration with `pdf-parse` and OpenAI API.

---
## 🚀 Project Guide

### 1. Setup

This guide assumes you have basic familiarity with React and Javascript. If you need a refresher, check out the [React Quick Start](https://react.dev/learn). Before we get started with coding, we need to install a few dependencies. 

Node.js is a JavaScript runtime environment that allows developers to run JavaScript code on the server side or outside a web browser. It comes pre-packaged with npm, a package manager that allows developers to install open-source libraries and modules. You can install Node.js from [this website](https://nodejs.org/). 

After installation, open your terminal or command prompt and run:
```bash
node -v
npm -v
```

For this project, we also need an API key to communicate with the OpenAI API. An API key is a unique identifier used to authenticate requests made to an API (Application Programming Interface). We will provide the API key in your application or scripts when making requests to OpenAI's API.

Sign up for an account at [OpenAI](https://platform.openai.com/signup). Navigate to the API Keys section in your OpenAI dashboard and click Create New Secret Key to generate a new API key.

![OpenAI Dashboard](assets/openai_dashboard.png)

Copy the API key and save it securely. (You won't be able to see it again once you close the window.)

### 2. Primer: What is an API?

An API (Application Programming Interface) is a set of rules and tools that allow different software applications to communicate with each other. Think of it as a bridge that enables one application to request and receive data or functionality from another.

In simpler terms, APIs allow developers to access features or data from external services, without needing to know how those services work internally. For example:
- Social media APIs let you embed posts or retrieve user data.
- Weather APIs provide real-time weather updates.
- OpenAI’s API allows developers to use AI capabilities, such as ChatGPT, in their own apps.

When you use an API, you're essentially making a request to a server, and the server sends back a response. This is typically done over the internet using protocols like HTTP. Here’s the basic workflow:
1. **Request**: Your application sends a request to the API endpoint (a URL). This request might include parameters, such as what data you need or actions you want to perform.
2. **Authentication**: Many APIs require you to provide an API key to verify your identity.
3. **Response**: The server processes your request and sends back a response, usually in JSON format (a lightweight data format that’s easy to read and work with). It also returns a status code, which indicates whether the request was successful (e.g., `200 OK`) or if there was an error (e.g., `404 Not Found`).

![What is a REST API](assets/what_is_rest_api.png)

#### Understanding REST APIs

A REST API (Representational State Transfer) is a widely used style for building APIs. It relies on standard web technologies like HTTP, making it simple and accessible for developers to use. REST APIs are often used for client-server communication.

With REST APIs, communication is organized around "resources." A resource is any object or piece of data that the API manages, such as a PDF document, a user profile, or a question. Each resource is identified by a URL, known as an endpoint.

You interact with these resources by performing actions, which are represented by HTTP methods. For example:
- If you want to retrieve information, you use the GET method.
- To send data or create something new, you use the POST method.
- Updating or modifying a resource typically uses PUT or PATCH.
- Deleting a resource involves the DELETE method.

REST APIs also often include parameters to customize the request. For instance, query parameters allow you to filter or sort results (e.g., `?search=pdf`), while data in the request body lets you send more complex information, like a question to analyze text.

An endpoint is a specific URL that identifies a resource or action within the API. It is like the address where you send your request to interact with a resource. For example:
- `https://api.example.com/documents` might represent all documents.
- `https://api.example.com/documents/123` could represent a specific document with an ID of `123`.

Endpoints often combine with HTTP methods to perform different actions:
- GET `/documents` retrieves all documents.
- POST `/documents` creates a new document.
- GET `/documents/123` retrieves a specific document with ID 123.
- DELETE `/documents/123` deletes the document with ID 123

![REST API Request Anatomy](assets/rest_anatomy.png)

When interacting with an endpoint, a request typically includes:
- The endpoint URL that specifies the resource (e.g., /documents).
- An HTTP method that defines the action (e.g., GET to retrieve data).
- Headers that provide metadata (e.g., an API key for authentication).
- Query parameters in the URL (e.g., ?search=pdf).
- Body parameters in JSON format for sending complex data.

In this project, we’ll use a REST API to send questions and context to the OpenAI API via HTTP POST requests. The API will analyze the PDF text and return a JSON response with the answers. Understanding these basics will help as we integrate the OpenAI API with our backend. Let’s move on to setting up the project!

### 3. Creating a New Project

Our project will be organized into two folders.  

```plaintext
pdf-question-helper/
├── frontend/       # React Frontend
└── backend/       # Express Backend
```

The frontend contains the user-facing interface, built with React. It handles:
- UI/UX design and layout.
- User interaction and event handling.
- Fetching and displaying data from the backend.

The backend contains the server-side code, built with Express. It handles our APIs to process requests from the frontend.

#### Frontend
`create-react-app` is a popular tool to bootstrap React applications. It sets up a new React project with all the essential configurations and tools pre-installed, so you can start coding immediately without hassle.

1. Create a React app
   ```bash
   npx create-react-app frontend
   ```
2. Navigate to the project folder:
   ```bash
   cd frontend
   ```
3. Install Axios for making API requests:
   ```bash
   npm install axios
   ```

4. Start the frontend:
   ```bash
   npm start
   ```

---

#### Backend

1. Navigate back to the root directory and create the server folder:
   ```bash
   mkdir backend && cd backend
   ```
2. Initialize a Node.js project:
   ```bash
   npm init -y
   ```
3. Install dependencies:
   ```bash
   npm install express multer pdf-parse openai cors
   ```
4. Create a file `index.js` and add the following code. 

   ```javascript
   // IMPORT LIBRARIES
   const express = require('express');
   const multer = require('multer'); // Multer is used to handle file uploads in Node.js.
   const pdfParse = require('pdf-parse'); // pdf-parse is a library to extract text from PDF files.
   const { OpenAI } = require('openai');
   const cors = require('cors');

   // INITIALIZATION 
   const app = express();
   const upload = multer();
   const openai = new OpenAI('YOUR_OPENAI_API_KEY');

   app.use(cors()); // Enables cross-origin resource sharing, allowing the frontend (running on a different port) to send requests to the backend
   app.use(express.json());

   // START SERVER
   app.listen(5001, () => console.log('Server running on http://localhost:5001'));
   // The server listens on port 5001 and logs a message when it's running.
   ```

5. Start the backend in a separate terminal window (the frontend and backend should be running at the same time!):
   ```bash
   node index.js
   ```

---

### 4. Creating a User Interface

Now, we'll create a simple UI to upload the PDF, display extracted text, and allow users to ask questions.

Open `frontend/src/App.js`. This is the file that runs the main React component of your application, serving as the entry point for building the user interface.

1. Replace `App.js` with the following code. Here, we use `useState` to initialize three state variables: `pdfText` to store the extracted text from the uploaded PDF, `question` to hold the user's input for their question, and `answer` to store the response (or answer) to the question. 
   ```javascript
   import React, { useState } from 'react';
   import './App.css'
   
   function App() {
       const [pdfText, setPdfText] = useState('');
       const [question, setQuestion] = useState('');
       const [answer, setAnswer] = useState('');
   
       return <div className="App"></div>;
   }
   
   export default App;
   ```
   
2. Add the ability for users to upload a PDF file. To handle the uploaded file, we define a function called `handleFileUpload`. When a file is uploaded, this function is triggered via the `onChange` event listener on the input field. The extracted text is stored in the `pdfText` state variable using the `setPdfText` function. For now, we will omit the API call for simplictiy.
   ```javascript
   function App() {
    const [pdfText, setPdfText] = useState('');
    const [question, setQuestion] = useState('');
    const [answer, setAnswer] = useState('');

    const handleFileUpload = async (event) => {
        const file = event.target.files[0];
        // API CALL OMITTED
        setPdfText('This is the mock text extracted from the PDF.');
    };

    return (
        <div className="App">
            <h1>PDF Q&A</h1>
            <input type="file" accept="application/pdf" onChange={handleFileUpload} />
        </div>
    );
   }
      
   export default App;
   ```

3. Now that we can upload a file and store mock text in the pdfText state, the next step is to display this extracted text to the user. We use the `{}` to tell the JSX parser that the content inside should be interpreted as JavaScript rather than a string. 
   ```javascript
   return (
    <div className="App">
        <h1>PDF Q&A</h1>
        <input type="file" accept="application/pdf" onChange={handleFileUpload} />
        <div>
            <h2>Extracted Text</h2>
            <pre style={{ whiteSpace: 'pre-wrap', wordWrap: 'break-word' }}>
                {pdfText}
            </pre>
        </div>
    </div>
   );
   ```

4. Next, we give the user the ability to ask a question about the displayed text. This is achieved by adding a text input field and binding it to the question state. Whenever the user types in this input field, the onChange event updates the question state using the setQuestion function. Again, we omit the API call for now. We also add a button to allow users to submit their question.
   ```javascript
   const handleAskQuestion = async () => {
       // API CALL OMITTED
       setAnswer('This is the mocked answer based on the question.');
   };
   
   return (
       <div className="App">
           <h1>PDF Q&A</h1>
           <input type="file" accept="application/pdf" onChange={handleFileUpload} />
           <div>
               <h2>Extracted Text</h2>
               <pre style={{ whiteSpace: 'pre-wrap', wordWrap: 'break-word' }}>
                   {pdfText}
               </pre>
           </div>
           <div>
               <input
                   type="text"
                   value={question}
                   onChange={(e) => setQuestion(e.target.value)}
                   placeholder="Ask a question"
               />
               <button onClick={handleAskQuestion}>Ask</button>
           </div>
       </div>
   );
   ```

5. Finally, we display the mocked answer below the question input by conditionally rendering the `answer` state if it is not empty.
   ```javascript
   {answer && (
    <div>
        <strong>Answer:</strong> {answer}
    </div>
   )}
   ```
   
6. To make everything look nice, include the following styles in `frontend/src/App.css`. CSS (Cascading Style Sheets) is used to define the look and feel of a web application, such as colors, layouts, fonts, spacing, and responsiveness. Don't worry about the details for now! You can read more about CSS [here](https://www.w3schools.com/css/).
     ```css
      body {
        font-family: Arial, sans-serif;
        background-color: #f4f4f9;
        margin: 0;
        padding: 20px;
      }
      
      .App {
        max-width: 600px;
        margin: 0 auto;
        background: white;
        padding: 20px;
        border-radius: 8px;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
      }
      
      h1 {
        color: #333;
        text-align: center;
      }
      
      input[type="file"],
      input[type="text"] {
        width: 100%;
        padding: 10px;
        margin: 10px 0;
        border: 1px solid #ccc;
        border-radius: 4px;
      }
      
      button {
        width: 100%;
        padding: 10px;
        background-color: #007bff;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        margin-top: 10px;
      }
      
      button:hover {
        background-color: #0056b3;
      }
      
      pre {
        background-color: #f8f8f8;
        padding: 10px;
        border-radius: 4px;
        overflow-x: auto;
      }
      
      strong {
        display: block;
        margin-top: 20px;
        color: #333;
      }
   ```
   ![Frontend UI](assets/frontend_ui.png)

### 5. Defining API Endpoints
The backend will consist of two main API endpoints:
- `/upload`: This endpoint processes the uploaded PDF file and extracts its text using pdf-parse.
- `/ask`: This endpoint takes a question and the extracted text, sends it to OpenAI’s API, and returns the AI-generated answer.

Navigate to `backend/server.js`. 

1. Define an endpoint at `/upload` that allows the frontend to upload a PDF file. This code extracts the text from the PDF file in the request payload (`req.file.buffer`) using `pdfParse` and uses `res.json()` to send the result as a JSON response back to the client. We use a `try`-`except` block to handle errors with the API call. 
   ```javascript
   // Endpoint to handle PDF upload and extract text
   app.post('/upload', upload.single('file'), async (req, res) => {
       try {
           const data = await pdfParse(req.file.buffer); // Extract text from the uploaded PDF
           res.json({ text: data.text }); // Return the extracted text
       } catch (error) {
           console.error('Error parsing PDF:', error); // Log a debug message on error
           res.status(500).json({ error: 'Failed to parse PDF' });
       }
   });
   ```

2. Define the `/ask` endpoint to handle user questions based on the text extracted from a PDF. This endpoint accepts a POST request containing a JSON payload with two fields: question, representing the user’s query, and pdfText, which is the text extracted from the uploaded PDF file.

   Inside the function, the OpenAI API is invoked with a request to the GPT-4 model. The `messages` parameter passed to the API is an array where the user's query and the PDF text are formatted together. The prompt combines the extracted text (`pdfText`) and appends the user’s `question` (Q: ...), followed by an answer placeholder (A:). This setup allows the model to generate a response as if it were answering the question based on the given context.

   The stream mode is enabled in the API call, which means the response is returned in small chunks as it is generated. This is particularly useful for handling longer or more complex answers without waiting for the entire response to be generated. Within the `for await` loop, each incoming chunk is processed, and its content (if present) is appended to the `answer` variable. Finally, the assembled answer is sent back to the client as a JSON object with the `answer` field.

   ```javascript
   app.post('/ask', async (req, res) => {
       const { question, pdfText } = req.body;
       try {
           const stream = await openai.chat.completions.create({
               model: "gpt-4o-mini",
               messages: [{ role: "user", content: `${pdfText}\n\nQ: ${question}\nA:` }],
               store: true,
               stream: true,
           });
   
           let answer = '';
           for await (const chunk of stream) {
               answer += chunk.choices[0]?.delta?.content || "";
           }
   
           res.json({ answer });
       } catch (error) {
           console.error('Error with OpenAI API:', error);
           res.status(500).json({ error: 'Failed to get response from OpenAI' });
       }
   });
   ```

### 6. Putting It All Together

Lastly, we need to call our backend API endpoints from the appropriate frontend handler functions. To do this, we'll use Axios, a popular JavaScript library used to make HTTP requests from a web application to a server. Returning to  `frontend/src/App.js`:

1. Add `import axios from 'axios';` to the top of the file. This imports the Axios library into our app. 
2. Edit the `handleFileUpload` function to include the following Axios `POST` request to the `upload` endpoint.
   ```javascript
   const handleFileUpload = async (event) => {
    const file = event.target.files[0]; // Get the selected file from the file input element

    const formData = new FormData(); // Create a new FormData instance to prepare the file for upload
    formData.append('file', file); // Append the file to the FormData object with the key 'file'

    try {
        // Make a POST request to the backend API to upload the file
        const response = await axios.post('http://localhost:5001/upload', formData, {
            headers: {
                'Content-Type': 'multipart/form-data' // Set the content type to handle file uploads
            }
        });

        setPdfText(response.data.text); // Update the state with the returned text
    } catch (error) {
        console.error('Error uploading file:', error);
    }
   };
   ```
3. Modify the `handleAskQuestion` function to make a `POST` request to the backend `/ask` endpoint. Make sure to pass in the `question` and `pdfText` variables as parameters into our `axios.post()` call.
   ```javascript
   const handleAskQuestion = async () => {
        try {
            const response = await axios.post('http://localhost:5001/ask', {
                question,
                pdfText
            });
            setAnswer(response.data.answer);
        } catch (error) {
            console.error('Error asking question:', error);
        }
   };
   ```

### 7. Conclusion
Congratulations! You've just completed building the PDF Question Helper app, integrating a React frontend, an Express backend, and OpenAI's API. This project has taken you through essential concepts and tools, including:
- Setting up and structuring a full-stack application.
- Using `pdf-parse` to extract text from PDF files.
- Making HTTP requests from a React frontend to an Express backend using Axios.
- Leveraging OpenAI’s API to process and respond to user questions.
- Understanding and working with REST APIs and JSON data.

If you missed anything, feel free to reference this repository, where a working version of the project is implemented.

Good luck, and happy hacking! 🚀

## 🌵 Credit
The code and write-up for this starter project was created by the 2025 HackDuke Tech Team. This project was inspired by [TreeHacks' Hackpacks](https://github.com/TreeHacks). Check them out for more sample projects and guides!
