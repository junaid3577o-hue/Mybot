<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Universal AI Chat</title>

<script src="https://cdn.tailwindcss.com"></script>
<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>

</head>

<body class="bg-gray-100 min-h-screen flex flex-col">

<div id="loading" class="fixed inset-0 flex items-center justify-center bg-white text-xl">
Loading App...
</div>

<div class="flex flex-1">

<!-- Sidebar -->
<div id="sidebar" class="w-64 bg-white border-r hidden md:flex flex-col">

<div class="p-4 border-b font-bold text-lg">
AI Chat
</div>

<button id="newChat" class="m-4 p-2 bg-blue-500 text-white rounded">
New Chat
</button>

<div id="chatList" class="flex-1 overflow-y-auto p-2 space-y-2"></div>

<button id="settingsBtn" class="m-4 p-2 bg-gray-200 rounded">
Settings
</button>

</div>

<!-- Main -->
<div class="flex-1 flex flex-col">

<!-- Header -->
<div class="p-3 border-b flex items-center justify-between bg-white">

<button id="menuBtn" class="md:hidden text-xl">☰</button>

<div class="font-semibold">
Universal AI Chat
</div>

<button id="darkToggle">
🌙
</button>

</div>

<!-- Messages -->
<div id="messages" class="flex-1 overflow-y-auto p-4 space-y-4"></div>

<!-- Input -->
<div class="border-t bg-white p-3">

<div class="flex gap-2">

<input id="messageInput"
class="flex-1 border rounded p-2"
placeholder="Ask anything..."/>

<button id="sendBtn"
class="bg-blue-500 text-white px-4 rounded">
Send
</button>

</div>

</div>

</div>
</div>

<!-- Settings Modal -->
<div id="settingsModal"
class="fixed inset-0 bg-black bg-opacity-40 hidden items-center justify-center">

<div class="bg-white p-6 rounded w-96 space-y-4">

<h2 class="font-bold text-lg">Settings</h2>

<input id="apiKey"
type="password"
placeholder="OpenRouter API Key"
class="w-full border p-2 rounded"/>

<input id="modelName"
placeholder="Model"
class="w-full border p-2 rounded"
value="deepseek/deepseek-r1:free"/>

<button id="saveSettings"
class="bg-blue-500 text-white px-4 py-2 rounded w-full">
Save
</button>

</div>

</div>

<script>

const loading = document.getElementById("loading")

const messagesEl = document.getElementById("messages")
const input = document.getElementById("messageInput")

const apiKeyInput = document.getElementById("apiKey")
const modelInput = document.getElementById("modelName")

const settingsModal = document.getElementById("settingsModal")

const sendBtn = document.getElementById("sendBtn")

let chatHistory = JSON.parse(localStorage.getItem("chat") || "[]")

apiKeyInput.value = localStorage.getItem("apiKey") || ""
modelInput.value = localStorage.getItem("model") || "deepseek/deepseek-r1:free"

function renderMessages(){

messagesEl.innerHTML=""

chatHistory.forEach(m=>{

const div = document.createElement("div")

div.className = m.role==="user"
? "text-right"
: "text-left"

div.innerHTML =
`<div class="inline-block bg-white border p-3 rounded max-w-xl">
${marked.parse(m.content)}
</div>`

messagesEl.appendChild(div)

})

messagesEl.scrollTop = messagesEl.scrollHeight

}

async function sendMessage(){

const text = input.value.trim()

if(!text) return

input.value=""

chatHistory.push({role:"user",content:text})

renderMessages()

const apiKey = localStorage.getItem("apiKey")

if(!apiKey){

alert("Add API key in settings")

return

}

const model = localStorage.getItem("model")

const res = await fetch(
"https://openrouter.ai/api/v1/chat/completions",
{
method:"POST",
headers:{
"Authorization":"Bearer "+apiKey,
"Content-Type":"application/json"
},
body:JSON.stringify({
model:model,
messages:chatHistory
})
}
)

const data = await res.json()

const reply = data.choices?.[0]?.message?.content || "Error"

chatHistory.push({role:"assistant",content:reply})

localStorage.setItem("chat",JSON.stringify(chatHistory))

renderMessages()

}

sendBtn.onclick = sendMessage

input.addEventListener("keypress",e=>{
if(e.key==="Enter") sendMessage()
})

document.getElementById("settingsBtn").onclick=()=>{
settingsModal.classList.remove("hidden")
settingsModal.classList.add("flex")
}

document.getElementById("saveSettings").onclick=()=>{

localStorage.setItem("apiKey",apiKeyInput.value)
localStorage.setItem("model",modelInput.value)

settingsModal.classList.add("hidden")

}

document.getElementById("darkToggle").onclick=()=>{
document.body.classList.toggle("bg-gray-900")
document.body.classList.toggle("text-white")
}

document.getElementById("newChat").onclick=()=>{
chatHistory=[]
localStorage.removeItem("chat")
renderMessages()
}

renderMessages()

loading.style.display="none"

</script>

</body>
</html>
