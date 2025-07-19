import React, { useState, useEffect, useRef } from 'react';

// Main App component for the chatbot
const App = () => {
  // State to store chat messages (sender and text)
  const [messages, setMessages] = useState([]);
  // State to store the current input text from the user
  const [input, setInput] = useState('');
  // State to indicate if the LLM is currently processing a request
  const [isLoading, setIsLoading] = useState(false);

  // Ref for the chat history div to enable auto-scrolling
  const chatHistoryRef = useRef(null);

  // Effect to scroll to the bottom of the chat history whenever messages change
  useEffect(() => {
    if (chatHistoryRef.current) {
      chatHistoryRef.current.scrollTop = chatHistoryRef.current.scrollHeight;
    }
  }, [messages]);

  // Function to handle sending a message
  const handleSendMessage = async () => {
    if (input.trim() === '') return; // Don't send empty messages

    const userMessage = { sender: 'user', text: input.trim() };
    // Add user's message to the chat history
    setMessages((prevMessages) => [...prevMessages, userMessage]);
    setInput(''); // Clear the input field
    setIsLoading(true); // Show loading indicator

    try {
      // Prepare chat history for the LLM API call
      let chatHistory = messages.map(msg => ({
        role: msg.sender === 'user' ? 'user' : 'model',
        parts: [{ text: msg.text }]
      }));
      // Add the current user message to the chat history for the API call
      chatHistory.push({ role: 'user', parts: [{ text: userMessage.text }] });

      const payload = { contents: chatHistory };
      const apiKey = ""; // API key is provided by the Canvas environment for gemini-2.0-flash
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

      // Make the API call to the Gemini LLM
      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      const result = await response.json();

      let botResponseText = "ขออภัย เกิดข้อผิดพลาดในการรับคำตอบ"; // Default error message
      if (result.candidates && result.candidates.length > 0 &&
          result.candidates[0].content && result.candidates[0].content.parts &&
          result.candidates[0].content.parts.length > 0) {
        botResponseText = result.candidates[0].content.parts[0].text;
      } else {
        console.error("Unexpected API response structure:", result);
      }

      const botMessage = { sender: 'bot', text: botResponseText };
      // Add bot's response to the chat history
      setMessages((prevMessages) => [...prevMessages, botMessage]);

    } catch (error) {
      console.error("Error calling Gemini API:", error);
      // Display an error message to the user
      setMessages((prevMessages) => [
        ...prevMessages,
        { sender: 'bot', text: "ขออภัย เกิดข้อผิดพลาดในการเชื่อมต่อกับแชทบอท กรุณาลองใหม่อีกครั้ง" }
      ]);
    } finally {
      setIsLoading(false); // Hide loading indicator
    }
  };

  // Function to handle input field changes (for controlled component)
  const handleInputChange = (e) => {
    setInput(e.target.value);
  };

  // Function to handle key presses in the input field (e.g., Enter key)
  const handleKeyPress = (e) => {
    if (e.key === 'Enter' && !isLoading) {
      handleSendMessage();
    }
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-purple-50 to-indigo-100 flex items-center justify-center p-4 font-inter">
      <div className="w-full max-w-2xl bg-white rounded-xl shadow-2xl flex flex-col h-[80vh] md:h-[70vh] overflow-hidden">
        {/* Chat Header */}
        <div className="bg-gradient-to-r from-purple-600 to-indigo-700 text-white p-4 rounded-t-xl shadow-md flex items-center justify-between">
          <h1 className="text-2xl font-bold flex items-center">
            <svg xmlns="http://www.w3.org/2000/svg" className="h-7 w-7 mr-3" viewBox="0 0 24 24" fill="currentColor">
              <path d="M20 2H4c-1.1 0-2 .9-2 2v18l4-4h14c1.1 0 2-.9 2-2V4c0-1.1-.9-2-2-2zm-2 12H6v-2h12v2zm0-3H6V9h12v2zm0-3H6V6h12v2z"/>
            </svg>
            แชทบอทตอบคำถาม
          </h1>
        </div>

        {/* Chat History Display Area */}
        <div ref={chatHistoryRef} className="flex-1 p-4 overflow-y-auto custom-scrollbar bg-gray-50">
          {messages.length === 0 && (
            <div className="text-center text-gray-500 mt-10">
              <p className="text-lg">สวัสดีครับ! ฉันคือแชทบอทของคุณ</p>
              <p className="text-md">คุณมีคำถามอะไรไหม? พิมพ์คำถามของคุณด้านล่างได้เลย</p>
            </div>
          )}
          {messages.map((msg, index) => (
            <div
              key={index}
              className={`flex mb-4 ${msg.sender === 'user' ? 'justify-end' : 'justify-start'}`}
            >
              <div
                className={`max-w-[75%] p-3 rounded-lg shadow-md ${
                  msg.sender === 'user'
                    ? 'bg-indigo-500 text-white rounded-br-none'
                    : 'bg-gray-200 text-gray-800 rounded-bl-none'
                }`}
              >
                {msg.text}
              </div>
            </div>
          ))}
          {isLoading && (
            <div className="flex justify-start mb-4">
              <div className="max-w-[75%] p-3 rounded-lg shadow-md bg-gray-200 text-gray-800 rounded-bl-none">
                <div className="flex items-center">
                  <span className="dot-animation bg-gray-500"></span>
                  <span className="dot-animation bg-gray-500 delay-150"></span>
                  <span className="dot-animation bg-gray-500 delay-300"></span>
                </div>
              </div>
            </div>
          )}
        </div>

        {/* Input Area */}
        <div className="p-4 bg-gray-100 border-t border-gray-200 flex items-center rounded-b-xl">
          <input
            type="text"
            value={input}
            onChange={handleInputChange}
            onKeyPress={handleKeyPress}
            placeholder="พิมพ์คำถามของคุณที่นี่..."
            className="flex-1 p-3 border border-gray-300 rounded-full focus:outline-none focus:ring-2 focus:ring-indigo-500 transition-all duration-200 text-gray-800"
            disabled={isLoading}
          />
          <button
            onClick={handleSendMessage}
            className="ml-3 px-6 py-3 bg-indigo-600 text-white rounded-full shadow-lg hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition-all duration-200 flex items-center justify-center disabled:opacity-50 disabled:cursor-not-allowed"
            disabled={isLoading}
          >
            {isLoading ? (
              <svg className="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
                <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
              </svg>
            ) : (
              <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6" viewBox="0 0 24 24" fill="currentColor">
                <path d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/>
              </svg>
            )}
            <span className="ml-2 hidden sm:inline">{isLoading ? 'กำลังส่ง...' : 'ส่ง'}</span>
          </button>
        </div>
      </div>

      {/* Tailwind CSS and Custom Styles */}
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        .font-inter {
          font-family: 'Inter', sans-serif;
        }
        .custom-scrollbar::-webkit-scrollbar {
          width: 8px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
          background: #f1f1f1;
          border-radius: 10px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
          background: #888;
          border-radius: 10px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover {
          background: #555;
        }
        .dot-animation {
          width: 8px;
          height: 8px;
          border-radius: 50%;
          margin: 0 2px;
          animation: bounce 1.4s infinite ease-in-out both;
        }
        .dot-animation.delay-150 {
          animation-delay: -0.16s;
        }
        .dot-animation.delay-300 {
          animation-delay: -0.32s;
        }
        @keyframes bounce {
          0%, 80%, 100% {
            transform: scale(0);
          }
          40% {
            transform: scale(1);
          }
        }
      `}</style>
    </div>
  );
};

export default App;
