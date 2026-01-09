from flask import Flask, request, jsonify
from langchain_groq import ChatGroq

app = Flask(__name__)

llm = ChatGroq(
    model="openai/gpt-oss-20b",
    api_key="ENTEAR YOUR GROQ API KEY ",
    temperature=0.7,
    max_tokens=550,
)

chat_history = []

@app.route("/")
def home():
    return """
<!DOCTYPE html>
<html>
<head>
<title>Baby AI Chatbot</title>
<style>
body{
    margin:0;
    background:#0a0f2c;
    display:flex;
    justify-content:center;
    align-items:center;
    height:100vh;
    font-family:Arial;
}
.chat{
    width:400px;
    height:500px;
    background:#111845;
    border-radius:10px;
    display:flex;
    flex-direction:column;
    color:white;
}
.messages{
    flex:1;
    padding:10px;
    overflow-y:auto;
}
.user{color:#00f2ff;margin-bottom:5px;}
.bot{color:#ffffff;margin-bottom:10px;}
.input{
    display:flex;
}
input{
    flex:1;
    padding:10px;
    border:none;
}
button{
    padding:10px;
    border:none;
    background:#00f2ff;
    cursor:pointer;
}
</style>
</head>
<body>

<div class="chat">
    <div class="messages" id="messages"></div>
    <div class="input">
        <input id="text" placeholder="Ask anything...">
        <button onclick="send()">Send</button>
    </div>
</div>

<script>
async function send(){
    let text = document.getElementById("text");
    let msg = text.value;
    if(!msg) return;

    let box = document.getElementById("messages");
    box.innerHTML += `<div class="user">You: ${msg}</div>`;
    text.value = "";

    let res = await fetch("/chat",{
        method:"POST",
        headers:{"Content-Type":"application/json"},
        body:JSON.stringify({message:msg})
    });

    let data = await res.json();
    box.innerHTML += `<div class="bot">Baby: ${data.reply}</div>`;
    box.scrollTop = box.scrollHeight;
}
</script>

</body>
</html>
"""
@app.route("/chat", methods=["POST"])
def chat():
    global chat_history
    user_msg = request.json["message"]

    chat_history.append(("user", user_msg))
    response = llm.invoke(chat_history)
    reply = response.content
    chat_history.append(("assistant", reply))

    return jsonify({"reply": reply})

if __name__ == "__main__":
    app.run(debug=True)
