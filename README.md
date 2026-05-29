# -<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Instagram Pro Full Clone</title>
<meta name="viewport" content="width=device-width, initial-scale=1">

<style>
*{box-sizing:border-box;}
body{margin:0;font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,Oxygen,Ubuntu,Cantarell,sans-serif;background:#fafafa;}
[data-theme="dark"]{background:#0f172a;color:white;}
nav{display:flex;justify-content:space-between;align-items:center;padding:12px 20px;background:white;border-bottom:1px solid #dbdbdb;position:sticky;top:0;z-index:100;}
[data-theme="dark"] nav{background:#1e293b;border-color:#334155;}
.container{max-width:500px;margin:auto;padding:10px;}
input,button,textarea{padding:10px;margin:5px;border-radius:8px;border:1px solid #dbdbdb;font-family:inherit;font-size:14px;}
[data-theme="dark"] input,[data-theme="dark"] button,[data-theme="dark"] textarea{background:#1e293b;color:white;border-color:#334155;}
button{background:#ff2d55;color:white;cursor:pointer;border:none;font-weight:600;}
button:hover{background:#ff1744;opacity:0.9;}
button:disabled{opacity:0.5;cursor:not-allowed;}
input[type="file"]{padding:5px;}
.post,.box{background:white;margin:10px 0;padding:15px;border-radius:10px;border:1px solid #dbdbdb;}
[data-theme="dark"] .post,[data-theme="dark"] .box{background:#1e293b;border-color:#334155;}
img{max-width:100%;border-radius:10px;}
.row{display:flex;gap:10px;align-items:center;margin:10px 0;}
.avatar{width:40px;height:40px;border-radius:50%;object-fit:cover;}
.story{width:60px;height:60px;border-radius:50%;border:3px solid #ff2d55;cursor:pointer;object-fit:cover;margin:5px;}
#chatBox{height:300px;overflow-y:auto;border:1px solid #dbdbdb;padding:10px;margin:10px 0;border-radius:8px;background:#fafafa;}
[data-theme="dark"] #chatBox{background:#0f172a;border-color:#334155;}
.msg{margin:8px 0;padding:8px;background:white;border-radius:6px;}
[data-theme="dark"] .msg{background:#334155;}
.msg.sent{text-align:right;background:#ff2d55;color:white;}
.msg.received{text-align:left;}
.post-header{display:flex;justify-content:space-between;align-items:center;margin-bottom:10px;}
.post-meta{color:#65676b;}
[data-theme="dark"] .post-meta{color:#9ca3af;}
.loading{text-align:center;color:#65676b;}
.error{color:#dc2626;padding:10px;background:#fee2e2;border-radius:6px;margin:10px 0;}
[data-theme="dark"] .error{background:#7f1d1d;color:#fca5a5;}
.success{color:#16a34a;padding:10px;background:#dcfce7;border-radius:6px;margin:10px 0;}
[data-theme="dark"] .success{background:#15803d;color:#bbf7d0;}
.btn-group{display:flex;gap:5px;}
.btn-group button{flex:1;margin:0;}
section{margin:20px 0;}
h3{margin:15px 0 10px 0;}
.input-group{display:flex;gap:5px;}
.input-group input{flex:1;}
.input-group button{margin:0;}
</style>
</head>

<body>
<div class="container" id="app"></div>

<!-- FIREBASE COMPAT -->
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-storage-compat.js"></script>

<script>
/* 🔥 FIREBASE CONFIG */
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_BUCKET",
  appId: "YOUR_APP_ID"
};

firebase.initializeApp(firebaseConfig);

const auth = firebase.auth();
const db = firebase.firestore();
const storage = firebase.storage();

let user = null;
let peer;
let localStream;
let listeners = []; // Track listeners for cleanup

/* THEME */
let theme = localStorage.getItem("theme") || "light";
document.body.setAttribute("data-theme", theme);

/* UTILITIES */
function sanitize(str) {
  if (!str) return "";
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}

function showMessage(msg, type = "error") {
  const msgDiv = document.createElement("div");
  msgDiv.className = type;
  msgDiv.textContent = msg;
  const app = document.getElementById("app");
  app.insertBefore(msgDiv, app.firstChild);
  setTimeout(() => msgDiv.remove(), 4000);
}

function unsubscribeAll() {
  listeners.forEach(unsub => unsub());
  listeners = [];
}

/* ================= LOGIN ================= */
function loginPage() {
  document.getElementById("app").innerHTML = `
    <div style="max-width:400px;margin:50px auto;text-align:center;">
      <h1>📸 Instagram Pro</h1>
      <p>Clone Build with Firebase</p>
      <input id="email" type="email" placeholder="Email" autocomplete="email">
      <input id="pass" type="password" placeholder="Password" autocomplete="current-password">
      <div class="btn-group">
        <button onclick="login()" style="flex:1;">Login</button>
        <button onclick="register()" style="flex:1;">Register</button>
      </div>
      <p style="margin-top:20px;font-size:12px;color:#65676b;">Demo: Use any email & password to test</p>
    </div>
  `;
}

async function register() {
  const email_val = document.getElementById("email")?.value?.trim();
  const pass_val = document.getElementById("pass")?.value?.trim();

  if (!email_val || !pass_val) {
    showMessage("Please fill all fields", "error");
    return;
  }

  if (pass_val.length < 6) {
    showMessage("Password must be at least 6 characters", "error");
    return;
  }

  try {
    const u = await auth.createUserWithEmailAndPassword(email_val, pass_val);
    await db.collection("users").doc(u.user.uid).set({
      username: "user_" + Math.floor(Math.random() * 99999),
      avatar: `https://i.pravatar.cc/100?img=${Math.floor(Math.random() * 70)}`,
      bio: "New user",
      createdAt: Date.now()
    });
    showMessage("Account created! Logging in...", "success");
  } catch (err) {
    showMessage("Register error: " + (err.message || "Unknown error"), "error");
  }
}

async function login() {
  const email_val = document.getElementById("email")?.value?.trim();
  const pass_val = document.getElementById("pass")?.value?.trim();

  if (!email_val || !pass_val) {
    showMessage("Please fill all fields", "error");
    return;
  }

  try {
    await auth.signInWithEmailAndPassword(email_val, pass_val);
    showMessage("Login successful!", "success");
  } catch (err) {
    showMessage("Login error: " + (err.message || "Unknown error"), "error");
  }
}

/* ================= APP ================= */
auth.onAuthStateChanged(async u => {
  if (!u) {
    unsubscribeAll();
    loginPage();
    return;
  }

  user = u;
  try {
    let doc = await db.collection("users").doc(u.uid).get();
    user.profile = doc.data() || {
      username: "user_" + Math.floor(Math.random() * 99999),
      avatar: `https://i.pravatar.cc/100`,
      bio: ""
    };
    render();
    initListeners();
  } catch (err) {
    showMessage("Error loading profile: " + err.message, "error");
  }
});

function initListeners() {
  unsubscribeAll();
  listeners.push(feedListener());
  listeners.push(storiesListener());
  listeners.push(chatListener());
}

/* ================= UI ================= */
function render() {
  document.getElementById("app").innerHTML = `
    <nav>
      <b style="font-size:20px;">📸 Pro Insta</b>
      <div class="btn-group" style="margin:0;gap:10px;">
        <button onclick="themeToggle()" style="margin:0;width:auto;">🌙</button>
        <button onclick="logout()" style="margin:0;width:auto;">Logout</button>
      </div>
    </nav>

    <div class="container">
      <div class="box" style="text-align:center;">
        <img class="avatar" src="${user.profile.avatar || 'https://i.pravatar.cc/100'}" style="width:80px;height:80px;">
        <p><strong>@${sanitize(user.profile.username)}</strong></p>
        <p style="font-size:14px;color:#65676b;">${sanitize(user.profile.bio || "")}</p>
      </div>

      <section>
        <h3>Create Post</h3>
        <input id="img" type="url" placeholder="Image URL">
        <textarea id="cap" placeholder="Write a caption..." style="width:100%;resize:vertical;"></textarea>
        <button onclick="post()" style="width:100%;">📤 Post</button>
      </section>

      <section>
        <h3>Stories (24h)</h3>
        <input type="file" accept="image/*" onchange="uploadStory(event)">
        <div id="stories"></div>
      </section>

      <section>
        <h3>Direct Messages</h3>
        <div class="input-group">
          <input id="to" type="text" placeholder="User ID to chat">
        </div>
        <div id="chatBox"><p class="loading">No messages yet</p></div>
        <div class="input-group">
          <input id="msg" type="text" placeholder="Type message...">
          <button onclick="sendMsg()" style="margin:0;">Send</button>
        </div>
      </section>

      <section>
        <h3>Video Call (WebRTC)</h3>
        <div class="btn-group">
          <button onclick="startCall()">📞 Start Call</button>
          <button onclick="answerCall()">✓ Answer</button>
          <button onclick="endCall()">✕ End</button>
        </div>
      </section>

      <section>
        <h3>Feed</h3>
        <div id="feed"><p class="loading">Loading posts...</p></div>
      </section>
    </div>
  `;
}

/* ================= POSTS ================= */
async function post() {
  const image_url = document.getElementById("img")?.value?.trim();
  const caption = document.getElementById("cap")?.value?.trim();

  if (!image_url || !caption) {
    showMessage("Please fill image URL and caption", "error");
    return;
  }

  // Validate URL
  try {
    new URL(image_url);
  } catch {
    showMessage("Invalid image URL", "error");
    return;
  }

  try {
    await db.collection("posts").add({
      uid: user.uid,
      username: user.profile.username,
      avatar: user.profile.avatar,
      image: image_url,
      caption: caption,
      time: Date.now(),
      likes: 0
    });
    document.getElementById("img").value = "";
    document.getElementById("cap").value = "";
    showMessage("Post created!", "success");
  } catch (err) {
    showMessage("Error posting: " + err.message, "error");
  }
}

function feedListener() {
  return db.collection("posts").orderBy("time", "desc").limit(50)
    .onSnapshot(s => {
      let html = "";
      s.forEach(d => {
        let p = d.data();
        let timeAgo = getTimeAgo(p.time);
        html += `
          <div class="post">
            <div class="post-header">
              <div class="row" style="margin:0;">
                <img class="avatar" src="${p.avatar || 'https://i.pravatar.cc/100'}">
                <div>
                  <strong>@${sanitize(p.username || "Anonymous")}</strong>
                  <p style="margin:0;font-size:12px;color:#65676b;">${timeAgo}</p>
                </div>
              </div>
            </div>
            <img src="${p.image}" onerror="this.src='https://via.placeholder.com/500x500?text=Image+Not+Found'">
            <p style="margin:10px 0;">${sanitize(p.caption)}</p>
            <p style="color:#65676b;font-size:14px;">❤️ ${p.likes || 0} likes</p>
          </div>`;
      });
      const feed = document.getElementById("feed");
      if (feed) feed.innerHTML = html || "<p class='loading'>No posts yet</p>";
    }, err => {
      console.error("Feed error:", err);
      showMessage("Error loading feed", "error");
    });
}

function getTimeAgo(timestamp) {
  const seconds = Math.floor((Date.now() - timestamp) / 1000);
  if (seconds < 60) return "now";
  if (seconds < 3600) return Math.floor(seconds / 60) + "m ago";
  if (seconds < 86400) return Math.floor(seconds / 3600) + "h ago";
  return Math.floor(seconds / 86400) + "d ago";
}

/* ================= STORIES ================= */
async function uploadStory(e) {
  const file = e.target.files?.[0];
  if (!file) return;

  // Validate file
  if (!file.type.startsWith("image/")) {
    showMessage("Please upload an image file", "error");
    return;
  }

  if (file.size > 5 * 1024 * 1024) {
    showMessage("File size must be less than 5MB", "error");
    return;
  }

  try {
    const filename = `${user.uid}_${Date.now()}`;
    const ref = storage.ref("stories/" + filename);
    await ref.put(file);
    const url = await ref.getDownloadURL();

    await db.collection("stories").add({
      uid: user.uid,
      username: user.profile.username,
      avatar: user.profile.avatar,
      img: url,
      time: Date.now()
    });

    e.target.value = "";
    showMessage("Story uploaded!", "success");
  } catch (err) {
    showMessage("Error uploading story: " + err.message, "error");
  }
}

function storiesListener() {
  return db.collection("stories").orderBy("time", "desc")
    .onSnapshot(s => {
      let html = "";
      s.forEach(d => {
        let st = d.data();
        if (Date.now() - st.time < 86400000) { // 24 hours
          html += `<img class="story" src="${st.img}" title="@${sanitize(st.username || "user")}" onclick="alert('Story by ' + '${sanitize(st.username || "user")}')">`;
        }
      });
      const stories = document.getElementById("stories");
      if (stories) stories.innerHTML = html || "<p class='loading'>No stories yet</p>";
    }, err => console.error("Stories error:", err));
}

/* ================= CHAT (DM) ================= */
function sendMsg() {
  const to = document.getElementById("to")?.value?.trim();
  const msg = document.getElementById("msg")?.value?.trim();

  if (!to || !msg) {
    showMessage("Please fill User ID and message", "error");
    return;
  }

  try {
    db.collection("chats").add({
      from: user.uid,
      fromName: user.profile.username,
      to: to,
      msg: msg,
      time: Date.now(),
      read: false
    });

    document.getElementById("msg").value = "";
    showMessage("Message sent!", "success");
  } catch (err) {
    showMessage("Error sending: " + err.message, "error");
  }
}

function chatListener() {
  return db.collection("chats").orderBy("time", "desc").limit(50)
    .onSnapshot(s => {
      let html = "";
      const to = document.getElementById("to")?.value?.trim();

      s.forEach(d => {
        let m = d.data();
        // Only show messages for current conversation
        if ((m.from === user.uid && m.to === to) || (m.to === user.uid && m.from === to)) {
          const isSent = m.from === user.uid;
          html = `<div class="msg ${isSent ? 'sent' : 'received'}">
            <strong>${isSent ? 'You' : sanitize(m.fromName || 'User')}:</strong><br>
            ${sanitize(m.msg)}
            <br><span style="font-size:11px;opacity:0.7;">${getTimeAgo(m.time)}</span>
          </div>` + html;
        }
      });

      const box = document.getElementById("chatBox");
      if (box) {
        box.innerHTML = html || "<p class='loading'>No messages yet</p>";
        box.scrollTop = box.scrollHeight;
      }
    }, err => console.error("Chat error:", err));
}

/* ================= CALLS (WebRTC) ================= */
async function startCall() {
  if (!navigator.mediaDevices?.getUserMedia) {
    showMessage("Webcam not supported on this device", "error");
    return;
  }

  try {
    peer = new RTCPeerConnection({
      iceServers: [
        { urls: ["stun:stun.l.google.com:19302", "stun:stun1.l.google.com:19302"] }
      ]
    });

    localStream = await navigator.mediaDevices.getUserMedia({ audio: true, video: { width: 640, height: 480 } });
    localStream.getTracks().forEach(t => peer.addTrack(t, localStream));

    peer.onicecandidate = e => {
      if (e.candidate) {
        console.log("ICE Candidate:", JSON.stringify(e.candidate));
      }
    };

    let offer = await peer.createOffer();
    await peer.setLocalDescription(offer);

    showMessage("Call started! Check console for offer to send", "success");
    console.log("SEND THIS OFFER:", JSON.stringify(peer.localDescription));
  } catch (err) {
    showMessage("Error starting call: " + err.message, "error");
  }
}

async function answerCall() {
  if (!navigator.mediaDevices?.getUserMedia) {
    showMessage("Webcam not supported on this device", "error");
    return;
  }

  try {
    const offer = prompt("Paste the offer JSON:");
    if (!offer) return;

    peer = new RTCPeerConnection({
      iceServers: [
        { urls: ["stun:stun.l.google.com:19302", "stun:stun1.l.google.com:19302"] }
      ]
    });

    localStream = await navigator.mediaDevices.getUserMedia({ audio: true, video: { width: 640, height: 480 } });
    localStream.getTracks().forEach(t => peer.addTrack(t, localStream));

    await peer.setRemoteDescription(new RTCSessionDescription(JSON.parse(offer)));

    let answer = await peer.createAnswer();
    await peer.setLocalDescription(answer);

    console.log("SEND THIS ANSWER:", JSON.stringify(peer.localDescription));
    showMessage("Answer created! Check console", "success");
  } catch (err) {
    showMessage("Error answering call: " + err.message, "error");
  }
}

function endCall() {
  if (localStream) {
    localStream.getTracks().forEach(t => t.stop());
  }
  if (peer) {
    peer.close();
  }
  showMessage("Call ended", "success");
}

/* ================= UTIL ================= */
function themeToggle() {
  theme = theme === "light" ? "dark" : "light";
  document.body.setAttribute("data-theme", theme);
  localStorage.setItem("theme", theme);
}

function logout() {
  if (confirm("Are you sure you want to logout?")) {
    unsubscribeAll();
    auth.signOut().catch(err => showMessage("Logout error: " + err.message, "error"));
  }
}

// Initial page
loginPage();
</script>
</body>
</html>
تطبيق احترافي 
