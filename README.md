<!DOCTYPE html>
<html lang="bn">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ad Bank</title>

<script src="https://cdn.tailwindcss.com"></script>
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;600;700&display=swap" rel="stylesheet">

<style>
body{
  font-family:'Montserrat',sans-serif;
  background:#020617;
  color:white;
}
.glass{
  background:rgba(255,255,255,0.08);
  backdrop-filter:blur(16px);
  border:1px solid rgba(255,255,255,0.12);
}
.fade{animation:fade .3s ease}
@keyframes fade{
  from{opacity:0;transform:translateY(8px)}
  to{opacity:1;transform:translateY(0)}
}
</style>
</head>

<body>
<div id="app"></div>

<script>
let user = JSON.parse(localStorage.getItem("adbank_user"));
let page="home";
let admin=false;
let cooldown=30;

function save(){localStorage.setItem("adbank_user",JSON.stringify(user));}
function getReq(){return JSON.parse(localStorage.getItem("withdraw_requests")||"[]");}
function setReq(r){localStorage.setItem("withdraw_requests",JSON.stringify(r));}

/* SIGNUP */
function signup(){
  const name=document.getElementById("name").value.trim();
  if(!name) return alert("নাম লিখুন");
  user={
    name,
    balance:0,
    referrals:0,
    code:Math.random().toString(36).substr(2,6),
    withdrawn:false,
    photo:""
  };
  save(); render();
}

/* SCRATCH */
function scratch(){
  if(cooldown>0) return;
  user.balance+=1;
  cooldown=30;
  save(); render();
}
setInterval(()=>{
  if(cooldown>0){
    cooldown--;
    const b=document.getElementById("scratchBtn");
    if(b) b.innerText=cooldown>0?"Wait "+cooldown+"s":"Scratch & Win ৳1";
  }
},1000);

/* WITHDRAW */
function withdraw(){
  const m=document.getElementById("method").value;
  const n=document.getElementById("number").value.trim();
  if(!n) return alert("নাম্বার দিন");

  const min=user.withdrawn?500:10;
  if(user.balance<min) return alert("Minimum ৳"+min);
  if(user.balance<5000 && user.withdrawn && user.referrals<3)
    return alert(" 3টি referral লাগবে");

  const r=getReq();
  r.push({
    name:user.name,
    amount:user.balance,
    method:m,
    number:n,
    time:new Date().toLocaleString()
  });
  setReq(r);
  localStorage.setItem("admin_notify",Date.now());

  user.balance=0;
  user.withdrawn=true;
  save();

  alert("Withdraw request sent");
  render();
}

/* PHOTO UPLOAD */
function uploadPhoto(e){
  const f=e.target.files[0];
  if(!f) return;
  const r=new FileReader();
  r.onload=()=>{user.photo=r.result; save(); render();}
  r.readAsDataURL(f);
}

/* ADMIN LOGIN */
function adminLogin(){
  if(prompt("Admin Password")==="mdmiraj%৳"){
    admin=true;
    page="admin";
    render();
  }
}

/* RENDER */
function render(){
  const app=document.getElementById("app");

  if(!user){
    app.innerHTML=`
    <div class="min-h-screen flex items-center justify-center p-4">
      <div class="glass p-6 rounded-3xl w-full max-w-sm fade">
        <h1 class="text-2xl font-bold text-center mb-4">Ad Bank</h1>
        <input id="name" class="w-full p-3 bg-black/30 rounded-xl mb-4" placeholder="Your Name">
        <button onclick="signup()" class="w-full p-3 rounded-xl bg-gradient-to-r from-cyan-500 to-blue-600">
          Create Account
        </button>
      </div>
    </div>`;
    return;
  }

  let c="";

  if(page==="home"){
    c=`
    <div class="p-4 fade">
      <div class="glass p-6 rounded-3xl mb-4">
        <p class="opacity-70">Balance</p>
        <h2 class="text-4xl font-bold">৳${user.balance}</h2>
        <p class="text-sm">Referrals: ${user.referrals}</p>
      </div>

      <button id="scratchBtn" onclick="scratch()"
        class="w-full h-36 rounded-3xl text-2xl font-bold
        bg-gradient-to-br from-cyan-500 to-blue-700">
        Scratch & Win ৳1
      </button>
    </div>`;
  }

  /* COMBINED PROFILE PAGE */
  if(page==="profile"){
    c=`
    <div class="p-4 fade space-y-4">

      <!-- PROFILE (PHOTO + NAME ONLY, NO REFERRAL CODE) -->
      <div class="glass p-6 rounded-3xl text-center">
        <label>
          <img src="${user.photo||'https://i.imgur.com/8Km9tLL.png'}"
            class="w-28 h-28 rounded-full mx-auto mb-3 object-cover border-2 border-cyan-400">
          <input type="file" accept="image/*" hidden onchange="uploadPhoto(event)">
        </label>
        <h2 class="text-xl font-bold">${user.name}</h2>
        <button onclick="adminLogin()" class="text-xs opacity-60 mt-3">⚙ Admin</button>
      </div>

      <!-- WITHDRAW -->
      <div class="glass p-6 rounded-3xl">
        <h3 class="font-bold mb-3">Withdraw Balance</h3>
        <select id="method" class="w-full p-3 bg-black/30 rounded-xl mb-3">
          <option>bKash</option>
          <option>Nagad</option>
          <option>Rocket</option>
        </select>
        <input id="number" class="w-full p-3 bg-black/30 rounded-xl mb-4" placeholder="Wallet Number">
        <button onclick="withdraw()" class="w-full p-3 rounded-xl bg-gradient-to-r from-cyan-500 to-blue-600">
          Submit Withdraw
        </button>
      </div>

      <!-- REFER (ONLY HERE) -->
      <div class="glass p-6 rounded-3xl text-center">
        <h3 class="font-bold mb-2">Invite & Earn</h3>
        <p class="text-sm opacity-70 mb-3">
          বন্ধু Invite করলে ৳5 Bonus
        </p>
        <div class="bg-black/40 p-3 rounded-xl font-mono text-lg">
          ${user.code}
        </div>
      </div>

    </div>`;
  }

  if(page==="admin" && admin){
    const r=getReq();
    c=`
    <div class="p-4 fade">
      <div class="glass p-6 rounded-3xl">
        <h2 class="font-bold mb-3">Withdraw Requests</h2>
        ${r.length? r.map(x=>`
        <div class="bg-black/30 p-3 rounded-xl mb-3">
          <b>${x.name}</b><br>
          ৳${x.amount}<br>
          ${x.method} - ${x.number}
        </div>`).join(""):"No Requests"}
      </div>
    </div>`;
  }

  app.innerHTML=`
  <div class="pb-28">${c}</div>
  <div class="fixed bottom-3 left-3 right-3 glass rounded-3xl flex justify-around py-3">
    <button onclick="page='home';render()">🏠</button>
    <button onclick="page='profile';render()">👤</button>
    ${admin?`<button onclick="page='admin';render()">🛠</button>`:""}
  </div>`;
}
render();
</script>
</body>
</html>
