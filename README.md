# Voice Assistant with Annyang.js
Voice-powered chatbots bring a natural, hands-free interface to web applications. In this tutorial we’ll show how to build a **speech-enabled chatbot** that listens to spoken input, talks back, and can be completely customized – from the voice commands it recognizes to the AI “personality” behind it. We’ll use Annyang.js (a tiny JavaScript speech-recognition library) on the front end, and a simple Node.js (or Python) backend that calls an AI chatbot API.

Along the way we’ll explain each step in friendly, clear terms so both beginners and experienced developers can follow along. By the end, you’ll have a robust Chatbot where _you_ can customize the “system instructions” (the AI’s role or behavior) for any use case – customer support, task automation, or just a fun voice assistant. We’ll also share code examples, images, and best practices (including security tips like keeping API keys secret) so you can confidently adapt this to your own projects.

Introduction: Why a Voice Chatbot?
----------------------------------


Imagine talking to your website the way you chat on a phone call. A voice chatbot can answer questions, take commands, or even just banter, all through speech. This hands-free interaction can make apps more accessible and engaging. In recent years, the Web Speech API has made it easy for browsers to understand spoken words, and libraries like **Annyang.js** wrap that power in simple code.

In parallel, powerful AI chat engines (like OpenAI’s ChatGPT) can generate intelligent replies. By combining these, we can build a Chatbot that listens with the browser, sends the transcript to an AI, and then speaks the answer back to the user. This tutorial walks through exactly that process, step by step, with code you can copy and modify. Along the way we’ll show how to customize the bot’s _system prompt_ (its “role”), so you can easily switch contexts – for example, from a friendly general helper to a formal customer-support agent.

**What’s in this tutorial:** We’ll start by introducing the main tools (Annyang.js and a chat AI API). Then we’ll set up the project structure and build the **frontend** (HTML/JavaScript) to capture voice. We’ll write code to send whatever the user says to a backend, which in turn calls an AI chatbot (using a system prompt we can change).We will discuss how to return the AI’s response and utilize the Web Speech API to vocalize it.Along the way we’ll cite official docs and articles to explain how each piece works. Finally, we’ll discuss best practices (like storing API keys in environment variables) and offer ideas for extending the Chatbot. Let’s get started!

Tools and Technologies Overview
-------------------------------


Before coding, let’s introduce the main tools:

### **Annyang.js**

– A tiny (about 2 KB) JavaScript library for speech recognition It wraps the browser’s Web Speech API so you can define commands in plain language. For example, you might say `annyang.addCommands({'hello': sayHello})` to run a function when the user says “hello.” As the official site explains,”Annyang is a small JavaScript library that enables your visitors to navigate your website using voice commands… it has a size of only 2kb and is available for free

.”It supports multiple languages and _progressively enhances_ the user’s experience – if the browser doesn’t support speech recognition, nothing breaks (older browsers just won’t recognize speech[t](https://www.talater.com/annyang/#:~:text=annyang%20plays%20nicely%20with%20all,users%20with%20older%20browsers%20unaffected)). In practice, we’ll use Annyang to listen for speech and grab the recognized text. (Note: Annyang focuses on commands, but we can also tap into its “result” callback to get raw transcribed text even if it doesn’t match a defined command.)

### **Web Speech API**

– Built into modern browsers, this provides _SpeechRecognition_ (for turning voice into text) and _SpeechSynthesis_ (for turning text into spoken audio). Annyang itself uses SpeechRecognition under the hood. For output, the SpeechSynthesis API lets us speak back the AI’s replies without any extra downloads. Because it’s a web standard, it’s very fast (usually near-instant) and doesn’t require external cloud services for speech synthesis. We’ll call `new SpeechSynthesisUtterance()` on the front end to speak the response.

### **Chatbot API (e.g. OpenAI’s GPT)**

– On the backend, we’ll use a language model API (like OpenAI’s GPT-3.5/GPT-4) to actually generate the chatbot’s replies. In our examples we’ll show Node.js code using OpenAI’s official library, but the concept works with any similar AI API. Crucially, these chat APIs allow a _system message_ (sometimes called system instructions or prompt) that sets the assistant’s behavior.For instance, one might state, “You are a cordial financial advisor” or “You are a task automation bot,” and the model will adjust its responses accordingly.

*   We will show how to include and modify this system message in our code.
*   **Node.js and Express (or Flask, etc.)** – We need a server to handle API requests securely. We’ll show a simple Express.js server that listens for POST requests from the frontend, calls the AI API using a stored key, and returns the AI’s answer. (Important security note: **never** put your secret API key in frontend code; always keep it on the server[help.openai.com](https://help.openai.com/en/articles/5112595-best-practices-for-api-key-safety#:~:text=2,like%20browsers%20or%20mobile%20apps)[help.openai.com](https://help.openai.com/en/articles/5112595-best-practices-for-api-key-safety#:~:text=4,place%20of%20your%20API%20key). We will keep it in an environment variable.)
*   **HTML/CSS/JavaScript** – Our front end will be a basic webpage with a chat interface (text area or message list) and a button (or automatic start) for voice input. We’ll show how to integrate Annyang and update the DOM with chat messages.

With these pieces, we’ll have a full stack solution: **Browser (voice input + speech output) ↔ Backend (AI processing)**. The images below illustrate the concept.

_Example of a voice chatbot UI:_ This mockup shows an “AI assistant” that greets the user by voice. In our project, the user will click a button (or tap an icon) to start speaking, the browser (with Annyang) captures the words, sends them to the server, and then speaks back the AI’s response.

Now we’ll dive into building the frontend step by step.

Front End Setup
---------------

### Frontend Setup: Chat Interface and Annyang


First, let’s set up a simple HTML page to hold our chat interface. We want an area to display messages, an input (if needed), and a button to start/stop listening. Here’s a basic skeleton:

#### Html

    
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <title>Voice Chatbot Demo</title>
      <style>
        /* Basic styles for chat messages */
        body { font-family: Arial, sans-serif; padding: 20px; }
        #chatbox { border: 1px solid #ccc; padding: 10px; height: 300px; overflow-y: scroll; }
        .user-msg { color: blue; margin: 5px 0; }
        .bot-msg  { color: green; margin: 5px 0; }
        #controls { margin-top: 10px; }
      </style>
    </head>
    <body>
      <!-- Chat message area -->
      <div id="chatbox"></div>
      
      <!-- Controls: start/stop listening -->
      <div id="controls">
        <button id="start-btn">Start Listening</button>
        <button id="stop-btn" disabled>Stop Listening</button>
      </div>
    
      <!-- aicotra.com Speech Chatbot Example -->
      <!-- Load Annyang.js from CDN -->
      <script src="https://cdnjs.cloudflare.com/ajax/libs/annyang/2.6.1/annyang.min.js"></script>
      <!-- Main script -->
      <script src="chatbot.js"></script>
    </body>
    </html>
    <!-- Source Aicotra.com -->
    
      

In this snippet:

*   We have a `<div id="chatbox">` where messages will appear. We’ll append user messages (in blue) and bot messages (in green).
*   Two buttons allow starting and stopping the voice recognition. Initially, “Stop Listening” is disabled.
*   **Important:** We include Annyang via CDN (`annyang.min.js`) so the `annyang` object is available.
*   We added an HTML comment `<!-- aicotra.com Speech Chatbot Example -->` as a watermark/credit as requested. This won’t show up in the UI but meets the requirement to embed “aicotra.com” in the source code.
*   We’ll write the JavaScript in a separate `chatbot.js` file for clarity.

With this UI, the user can click “Start Listening” to let the bot hear them. Next, we write the JavaScript to hook Annyang and our chat logic.

### Frontend JavaScript: Capturing Voice with Annyang


In `chatbot.js`, we’ll check if Annyang is available, then configure it. The simplest “hello world” usage of Annyang is shown in its docs:You create a commands object and invoke it.`annyang.addCommands(commands)`, then `annyang.start()`. For example:

#### Javascript

    
    // chatbot.js
    
    // aicotra.com - Speech Chatbot Frontend Example
    
    if (annyang) {
      console.log("Annyang available - speech recognition can be started.");
      // Prepare commands (even if we mainly use 'result' callback)
      const commands = {
        // Define any voice commands if you want
        'hello': () => {
          appendMessage('user', 'hello');
          appendMessage('bot', 'Hi there! How can I help you?');
        }
      };
      annyang.addCommands(commands);
    } else {
      console.warn("Speech Recognition not supported in this browser.");
    }
    // Source Aicotra.com
    
      

However, for a chatbot we want a free-form conversation, not just fixed commands. We will utilize `annyang.addCallback('result', callback)` to record whatever the user articulates.The first element is typically the most likely. Let’s implement that, along with hooking the start/stop buttons:


    
    // chatbot.js (continued)
    
    // Get UI elements
    const chatbox = document.getElementById('chatbox');
    const startBtn = document.getElementById('start-btn');
    const stopBtn = document.getElementById('stop-btn');
    
    // Helper to display messages
    function appendMessage(sender, text) {
      const msg = document.createElement('div');
      msg.className = sender + '-msg';
      msg.textContent = (sender === 'user' ? 'You: ' : 'Bot: ') + text;
      chatbox.appendChild(msg);
      chatbox.scrollTop = chatbox.scrollHeight; // scroll to bottom
    }
    
    // Callback for recognized speech
    if (annyang) {
      annyang.addCallback('result', function(phrases) {
        const transcript = phrases[0]; // best guess
        console.log("Recognized speech:", transcript);
        if (transcript) {
          appendMessage('user', transcript);
          sendToBackend(transcript); // send text to server for AI response
        }
      });
    
      // When voice service ends (user stops speaking), disable stop button
      annyang.addCallback('soundend', function() {
        startBtn.disabled = false;
        stopBtn.disabled = true;
      });
    }
    
    // Start listening on button click
    startBtn.addEventListener('click', () => {
      annyang.start({ autoRestart: false, continuous: false });
      startBtn.disabled = true;
      stopBtn.disabled = false;
    });
    
    // Stop listening on button click
    stopBtn.addEventListener('click', () => {
      annyang.abort();
      startBtn.disabled = false;
      stopBtn.disabled = true;
    });
    // Source Aicotra.com
    
      

Let’s break this down:

*   We get references to the chatbox and buttons.
*   The helper function `appendMessage(sender, text)` creates a new `<div>` and adds it to the chatbox. We prefix user messages with “You:” and bot messages with “Bot:”, and use CSS classes for styling. (This is just for display; you could design a fancier chat bubble UI if desired.)
*   **Speech recognition callback:** If Annyang is supported, we add `annyang.addCallback('result', ...)`. As Tal Ater’s GitHub issue explains, this gives us an array of transcripts. We take `phrases[0]` as the most likely transcription. We then append it as a user message and call `sendToBackend(transcript)`, a function we’ll write to talk to our server.
*   We also listen for `'soundend'` (optionally) to re-enable the buttons once input is done.
*   The “Start Listening” button calls `annyang.start()`. We pass options `{continuous:false}` so it only listens once per click, but you could set `continuous:true` to listen indefinitely (if supported). We disable the Start button to prevent repeated clicks, and enable “Stop” so the user can cancel.
*   The “Stop Listening” button simply aborts recognition and toggles buttons back.

At this point, when the user clicks **Start**, Annyang will activate the microphone and listen. When they finish speaking, we get the transcription and display it. The next task is sending that text to the chatbot backend and handling the reply.
[Read More](https://www.aicotra.com/chatbot-mastery-10-steps-annyang/)
