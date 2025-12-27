<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Carrot Bot - Final Edition</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Comic+Neue:wght@700&family=Kalam:wght@400;700&display=swap');

        body,
        html {
            margin: 0;
            padding: 0;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            /* STATIC DOT BACKGROUND: Purple base with Yellow/Pink dots */
            background-color: #5d0c91;
            background-image: radial-gradient(#f1c40f 15%, transparent 16%),
                radial-gradient(#ff0080 15%, transparent 16%);
            background-size: 60px 60px;
            background-position: 0 0, 30px 30px;
        }

        /* MAIN CONTAINER: Smooth Edges & Realistic Shadow */
        #chat-container {
            width: 400px;
            height: 650px;
            background: #fff;
            border: 4px solid #000;
            border-radius: 25px;
            box-shadow: 12px 12px 0px #000;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        /* HEADER BAR */
        #header-bar {
            background: #000;
            padding: 15px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            color: #fff;
            font-family: 'Comic Neue', sans-serif;
        }

        /* NEW CONNECTED ICON (Modern Pulse) */
        .status-container {
            display: flex;
            align-items: center;
            gap: 8px;
            background: #1a1a1a;
            padding: 5px 12px;
            border-radius: 20px;
            border: 2px solid #333;
        }

        .pulse-dot {
            width: 10px;
            height: 10px;
            background-color: #2ecc71;
            border-radius: 50%;
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0% {
                box-shadow: 0 0 0 0 rgba(46, 204, 113, 0.7);
            }

            70% {
                box-shadow: 0 0 0 10px rgba(46, 204, 113, 0);
            }

            100% {
                box-shadow: 0 0 0 0 rgba(46, 204, 113, 0);
            }
        }

        .status-text {
            font-size: 11px;
            font-weight: bold;
            color: #2ecc71;
            text-transform: uppercase;
        }

        /* YELLOW PAPER CHAT BOX */
        #chat-box {
            flex: 1;
            background-color: #fffb96;
            background-image: linear-gradient(#e1d95d 1px, transparent 1px);
            background-size: 100% 28px;
            padding: 25px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .message-row {
            display: flex;
            align-items: flex-start;
            gap: 12px;
        }

        .bot-msg {
            background: #ff6b6b;
            color: #000;
            padding: 15px;
            border: 3px solid #000;
            border-radius: 18px;
            font-family: 'Kalam', cursive;
            font-weight: 700;
            max-width: 80%;
            box-shadow: 4px 4px 0px #000;
        }

        .user-msg {
            background: #fff;
            align-self: flex-end;
            padding: 12px 18px;
            border: 3px solid #000;
            border-radius: 18px;
            font-family: 'Kalam', cursive;
            box-shadow: 4px 4px 0px #000;
        }

        .bot-avatar {
            width: 45px;
            height: 45px;
            border: 3px solid #000;
            background: #fff;
            border-radius: 10px;
        }

        /* INPUT AREA: Smooth Edges */
        #input-area {
            background: #fff;
            border-top: 4px solid #000;
            padding: 20px;
            display: flex;
            gap: 10px;
        }

        input {
            flex: 1;
            background: #f0f0f0;
            border: 3px solid #000;
            padding: 12px;
            border-radius: 12px;
            font-family: 'Comic Neue', sans-serif;
            font-size: 15px;
            outline: none;
        }

        #send-btn {
            background: #000;
            color: #fff;
            border: none;
            padding: 0 20px;
            font-family: 'Comic Neue', sans-serif;
            font-weight: 900;
            cursor: pointer;
            border-radius: 12px;
        }

        #chat-box::-webkit-scrollbar {
            width: 0;
        }
    </style>
</head>

<body>

    <div id="chat-container">
        <div id="header-bar">
            <span>CARROT BOT</span>
            <div class="status-container">
                <div class="pulse-dot"></div>
                <span class="status-text">ONLINE</span>
            </div>
        </div>

        <div id="chat-box">
            <div class="message-row">
                <img src="https://cdn-icons-png.flaticon.com/512/2040/2040653.png" class="bot-avatar" alt="bot">
                <div class="bot-msg">
                    Carrot Bot activated. I'm rooted to the cloud and ready to talk.
                </div>
            </div>
        </div>

        <div id="input-area">
            <input type="text" id="user-input" placeholder="Type to the Carrot...">
            <button id="send-btn" onclick="sendMessage()">YEET</button>
        </div>
    </div>

    <script type="module">
        /**
         * BACKEND READY SCRIPT
         */
        import { GoogleGenerativeAI } from "https://esm.run/@google/generative-ai";

        // CLIENT-SIDE CONFIGURATION
        // ⚠️ WARNING: Your API key is visible to anyone who inspects the page source.
        const API_KEY = "AIzaSyC7g7scBJ04wfjLn-i-hrBaVQuS_tJ2x70";
        const genAI = new GoogleGenerativeAI(API_KEY);
        const model = genAI.getGenerativeModel({ model: "gemini-flash-latest" });

        window.sendMessage = async function () {
            const inputField = document.getElementById('user-input');
            const text = inputField.value.trim();
            if (!text) return;

            appendMsg(text, 'user-msg');
            inputField.value = '';

            const tid = "t-" + Date.now();
            appendBotMsg("Carrot is thinking...", tid);

            try {
                // Direct call to Gemini from the browser
                const result = await model.generateContent(text);
                const response = await result.response;
                const replyText = response.text();

                document.getElementById(tid).innerText = replyText;
            } catch (error) {
                console.error(error);
                let errMsg = "Connection lost. Please check console.";
                if (error.toString().includes("429")) {
                    errMsg = "Too many requests (Quota Exceeded). Try again later.";
                }
                document.getElementById(tid).innerText = errMsg;
            }
        }

        function appendMsg(t, c) {
            const b = document.getElementById('chat-box');
            const d = document.createElement('div');
            d.className = `message ${c}`;
            d.innerText = t;
            b.appendChild(d);
            b.scrollTop = b.scrollHeight;
        }

        function appendBotMsg(t, id) {
            const b = document.getElementById('chat-box');
            const row = document.createElement('div');
            row.className = 'message-row';
            row.innerHTML = `
            <img src="https://cdn-icons-png.flaticon.com/512/2040/2040653.png" class="bot-avatar" alt="bot">
            <div class="bot-msg" id="${id}">${t}</div>
        `;
            b.appendChild(row);
            b.scrollTop = b.scrollHeight;
        }
    </script>

</body>

</html>
