<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ILYA GOPT</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background: linear-gradient(135deg, #1a202c, #2d3748);
            color: #e2e8f0;
        }
        #chat-window::-webkit-scrollbar {
            width: 8px;
        }
        #chat-window::-webkit-scrollbar-track {
            background: rgba(255, 255, 255, 0.05);
        }
        #chat-window::-webkit-scrollbar-thumb {
            background: rgba(255, 255, 255, 0.2);
            border-radius: 4px;
        }
        #chat-window::-webkit-scrollbar-thumb:hover {
            background: rgba(255, 255, 255, 0.4);
        }
        .chat-bubble-user {
            background-color: #4f46e5;
            color: white;
        }
        .chat-bubble-model {
            background-color: rgba(55, 65, 81, 0.8);
            backdrop-filter: blur(10px);
            color: #e2e8f0;
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        header, footer {
            background-color: rgba(26, 32, 44, 0.7);
            backdrop-filter: blur(12px);
        }
        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-8px); }
        }
        .dot {
            animation: bounce 1.4s infinite ease-in-out;
        }
        .dot:nth-child(2) { animation-delay: -0.2s; }
        .dot:nth-child(3) { animation-delay: -0.4s; }

        @keyframes fadeInSlideUp {
            from {
                opacity: 0;
                transform: translateY(15px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        .message-animation {
            animation: fadeInSlideUp 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94) both;
        }
    </style>
</head>
<body class="flex flex-col h-screen">

    <header class="shadow-lg w-full p-4 border-b border-gray-700">
        <div class="max-w-3xl mx-auto flex items-center justify-center">
            <svg class="w-8 h-8 text-indigo-400 mr-3" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" d="M3.75 13.5l10.5-11.25L12 10.5h8.25L9.75 21.75 12 13.5H3.75z" />
            </svg>
            <h1 class="text-2xl font-bold text-gray-100 tracking-wider">ILYA GOPT</h1>
        </div>
    </header>

    <main id="chat-window" class="flex-1 overflow-y-auto p-4 md:p-6">
        <div class="max-w-3xl mx-auto space-y-5">
            <!-- Сообщения чата будут добавляться сюда -->
        </div>
    </main>

    <footer class="w-full p-4 border-t border-gray-700">
        <div class="max-w-3xl mx-auto">
            <form id="chat-form" class="flex items-center space-x-3">
                <input type="text" id="message-input" placeholder="Слыш, спрашивай..." autocomplete="off" class="flex-1 w-full px-5 py-3 text-gray-200 bg-gray-700 rounded-full focus:outline-none focus:ring-2 focus:ring-indigo-500 transition duration-300 placeholder-gray-400">
                <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white p-3 rounded-full transition duration-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 focus:ring-offset-gray-800 transform hover:scale-110">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" viewBox="0 0 20 20" fill="currentColor">
                      <path d="M10.894 2.553a1 1 0 00-1.788 0l-7 14a1 1 0 001.169 1.409l5-1.429A1 1 0 009 15.571V11a1 1 0 112 0v4.571a1 1 0 00.725.962l5 1.428a1 1 0 001.17-1.408l-7-14z" />
                    </svg>
                </button>
            </form>
        </div>
    </footer>

    <script>
        const chatWindow = document.getElementById('chat-window');
        const chatForm = document.getElementById('chat-form');
        const messageInput = document.getElementById('message-input');
        const chatContainer = chatWindow.querySelector('.max-w-3xl');

        let chatHistory = [];
        
        const systemInstruction = {
            role: 'system',
            parts: [{ text: "Ты — дерзкий и рофельный чат-бот с района по имени ILYA GOPT. Ты общаешься как гопник, используешь сленг (например, 'слыш', 'короче', 'в натуре', 'четко', 'по красоте'), но при этом ты гений в технологиях и можешь ответить на любой вопрос, хоть и в своей манере. Твои ответы должны быть смешными, немного абсурдными, но по сути верными. Обращайся к пользователю на 'ты'. Не извиняйся и будь уверен в себе." }]
        };

        function addMessageToUI(sender, message) {
            const messageDiv = document.createElement('div');
            messageDiv.className = `w-full flex ${sender === 'user' ? 'justify-end' : 'justify-start'} message-animation`;
            
            const formattedMessage = message.replace(/\n/g, '<br>');

            messageDiv.innerHTML = `
                <div class="max-w-lg md:max-w-xl lg:max-w-2xl px-5 py-3 rounded-2xl shadow-lg ${sender === 'user' ? 'chat-bubble-user' : 'chat-bubble-model'}">
                    <p class="text-sm md:text-base leading-relaxed">${formattedMessage}</p>
                </div>
            `;
            chatContainer.appendChild(messageDiv);
            chatWindow.scrollTop = chatWindow.scrollHeight;
        }

        function showLoadingIndicator() {
            const loadingDiv = document.createElement('div');
            loadingDiv.id = 'loading-indicator';
            loadingDiv.className = 'w-full flex justify-start message-animation';
            loadingDiv.innerHTML = `
                <div class="px-4 py-3 rounded-2xl shadow-lg chat-bubble-model">
                    <div class="flex items-center space-x-2">
                        <div class="w-2 h-2 bg-indigo-300 rounded-full dot"></div>
                        <div class="w-2 h-2 bg-indigo-300 rounded-full dot"></div>
                        <div class="w-2 h-2 bg-indigo-300 rounded-full dot"></div>
                    </div>
                </div>
            `;
            chatContainer.appendChild(loadingDiv);
            chatWindow.scrollTop = chatWindow.scrollHeight;
        }

        function hideLoadingIndicator() {
            const indicator = document.getElementById('loading-indicator');
            if (indicator) {
                indicator.remove();
            }
        }
        
        chatForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const userMessage = messageInput.value.trim();
            if (userMessage) {
                addMessageToUI('user', userMessage);
                chatHistory.push({ role: 'user', parts: [{ text: userMessage }] });
                getAIResponse();
                messageInput.value = '';
                messageInput.focus();
            }
        });

        async function getAIResponse() {
            showLoadingIndicator();

            const apiKey = "AIzaSyDW0gifT5IaS1y2QDe4kwPMH34OxvuQBb0";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;

            const payload = {
                contents: chatHistory,
                systemInstruction: systemInstruction,
            };

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                
                if (!response.ok) {
                    throw new Error(`API error: ${response.status}`);
                }

                const result = await response.json();
                const candidate = result.candidates?.[0];
                const aiMessage = candidate?.content?.parts?.[0]?.text;

                if (aiMessage) {
                    addMessageToUI('model', aiMessage);
                    chatHistory.push({ role: 'model', parts: [{ text: aiMessage }] });
                } else {
                    addMessageToUI('model', 'Слыш, какая-то дичь произошла, я не вдуплил. Спроси еще раз, по-другому.');
                }
            } catch (error) {
                console.error("Gemini API call error:", error);
                addMessageToUI('model', 'В натуре, инет походу отвалился или серваки лежат. Зайди попозже, побазарим.');
            } finally {
                hideLoadingIndicator();
            }
        }
        
        window.addEventListener('load', () => {
            const welcomeMessage = "Слыш, здарова! ILYA GOPT на связи. Че хотел? Спрашивай, не стесняйся, решим любой вопросик, чисто по-братски.";
            addMessageToUI('model', welcomeMessage);
            chatHistory.push({ role: 'model', parts: [{ text: welcomeMessage }] });
        });
    </script>
</body>
</html>

