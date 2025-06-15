<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>האתר שלי</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    /* עיצוב לבועה */
    #chat-button {
      position: fixed;
      bottom: 30px;
      left: 30px;
      width: 60px;
      height: 60px;
      background-color: #facc15;
      color: black;
      border-radius: 50%;
      border: none;
      font-size: 30px;
      cursor: pointer;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
      z-index: 1000;
      transition: transform 0.2s;
    }

    #chat-button:hover {
      transform: scale(1.1);
    }

    #chat-container {
      display: none;
      flex-direction: column;
      position: fixed;
      bottom: 100px;
      left: 30px;
      width: 320px;
      height: 400px;
      background-color: white;
      border-radius: 10px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.3);
      padding: 10px;
      z-index: 999;
    }

    #messages {
      flex: 1;
      overflow-y: auto;
      padding: 10px;
      border-bottom: 1px solid #ccc;
    }

    #user-input {
      display: flex;
      padding-top: 10px;
    }

    #user-input input {
      flex: 1;
      border: 1px solid #ccc;
      padding: 8px;
      border-radius: 5px;
    }

    #user-input button {
      margin-right: 5px;
      background-color: #facc15;
      border: none;
      padding: 8px 12px;
      border-radius: 5px;
      cursor: pointer;
    }
  </style>
</head>
<body class="bg-gray-100 text-gray-800 font-sans min-h-screen">

  <!-- תפריט -->
  <header class="bg-black shadow-md sticky top-0 z-10">
    <div class="container mx-auto flex justify-between items-center p-4">
      <h1 class="text-2xl font-bold text-yellow-400">האתר שלי</h1>
      <nav class="space-x-4 space-x-reverse text-sm md:text-base text-white">
        <a href="index.chatbot14-04-25-my-site-.html" class="hover:text-yellow-300 transition">בית</a>
        <a href="about.14-04-25-my-site-1.html" class="hover:text-yellow-300 transition">אודות</a>
        <a href="projects.html" class="hover:text-yellow-300 transition">פרויקטים</a>
        <a href="contact.html" class="hover:text-yellow-300 transition">צור קשר</a>
      </nav>
    </div>
  </header>

  <!-- תוכן -->
  <main class="container mx-auto p-10 text-center">
    <h2 class="text-4xl md:text-5xl font-bold mb-6 text-yellow-500 drop-shadow-lg">ברוכה הבאה!</h2>
    <p class="text-lg md:text-xl mb-6 text-gray-700">
      כאן תמצאו השראה, פרויקטים טכנולוגיים, ואהבה גדולה לפיתוח ולאוטומציה.
    </p>
    <img 
      src="https://cdn.gamma.app/63po27z2su370mq/generated-images/B-dboUOMjT6zbNKcbz8_H.png" 
      alt="טכנולוגיה ואוטומציה"
      class="rounded-2xl shadow-2xl mx-auto w-full max-w-4xl mt-10 transition hover:scale-105"
    >
  </main>

<!-- כפתור הצ'אט -->
<button id="chat-button">💬</button>

<!-- חלונית הצ'אט -->
<div id="chat-container" style="display: none;">
  <div id="messages"></div>
  <div id="user-input">
    <input type="text" id="inputText" placeholder="כתוב כאן שאלה..." />
    <button id="sendButton">שלח</button>
  </div>
</div>

<!-- קובץ ה-JS הראשי -->
<script type="module" src="chatbot.js"></script>

<!-- סקריפט מקומי לצ'אט -->
<script>
  // בודק שהכפתור פותח וסוגר את הצ'אט
  const chatButton = document.getElementById("chat-button");
  const chatContainer = document.getElementById("chat-container");

  chatButton.addEventListener("click", () => {
    if (chatContainer.style.display === "none" || chatContainer.style.display === "") {
      chatContainer.style.display = "flex";
    } else {
      chatContainer.style.display = "none";
    }
  });

  // מאזין לכפתור השליחה
  document.getElementById("sendButton").addEventListener("click", sendMessage);

  // פונקציה להצגת הודעה בצ'אט
  function appendMessage(sender, text) {
    const messagesDiv = document.getElementById("messages");
    const messageElem = document.createElement("div");
    messageElem.innerHTML = `<strong>${sender}:</strong> ${text}`;
    messagesDiv.appendChild(messageElem);
  }

  // פונקציית שליחה לצ'אט
  async function sendMessage() {
    const userMsg = document.getElementById("inputText").value.trim();
    console.log("המשתמש שואל:", userMsg);
    if (!userMsg) return;

    appendMessage("את", userMsg);
    document.getElementById("inputText").value = "";

    try {
      // קריאה ל-Firebase לבדוק אם יש תשובה מוכנה
      const q = query(collection(db, "questions"), where("question", "==", userMsg));
      const snapshot = await getDocs(q);
      console.log("תוצאות Firebase:", snapshot);

      if (!snapshot.empty) {
        const doc = snapshot.docs[0];
        appendMessage("צ'אטבוט", doc.data().answer);
      } else {
        // אין תשובה – פונים ל-OpenAI
        const aiResponse = await getAIResponse(userMsg);
        appendMessage("צ'אטבוט", aiResponse);

        // שומרים את השאלה והתשובה במסד
        await addDoc(collection(db, "qa"), {
          question: userMsg,
          answer: aiResponse,
          timestamp: new Date()
        });
      }
    } catch (error) {
      console.error("שגיאה בשליחה:", error);
      appendMessage("צ'אטבוט", "אירעה שגיאה. נסה שוב.");
    }
  }
</script>
<script
  src='https://cdn.jotfor.ms/agent/embedjs/019732599424766291c12d20a0e56dcc8016/embed.js?skipWelcome=1&maximizable=1'>
</script>
</body>
</html>
