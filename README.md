<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>外場</title>
</head>
<body>

<h2>🧍 外場接單</h2>

<div id="orders"></div>

<script>
const owner = "你的帳號";
const repo = "你的repo";
const path = "orders.json";

async function load(){
    const res = await fetch(`https://raw.githubusercontent.com/${owner}/${repo}/main/${path}`);
    const data = await res.json();

    const el = document.getElementById("orders");
    el.innerHTML = "";

    data.forEach((o,i)=>{
        if(o.status === "new"){
            const div = document.createElement("div");

            div.innerHTML = `
                <pre>${format(o)}</pre>
                <button onclick="sendKitchen(${i})">送內場</button>
            `;

            el.appendChild(div);
        }
    });
}

function format(o){
    return JSON.stringify(o.items, null, 2);
}

async function sendKitchen(index){
    const res = await fetch(`https://api.github.com/repos/${owner}/${repo}/contents/${path}`);
    const file = await res.json();

    const data = JSON.parse(atob(file.content));

    data[index].status = "kitchen";

    await fetch(`https://api.github.com/repos/${owner}/${repo}/contents/${path}`, {
        method: "PUT",
        headers: {
            "Authorization": "token 你的GitHubToken"
        },
        body: JSON.stringify({
            message: "send to kitchen",
            content: btoa(JSON.stringify(data)),
            sha: file.sha
        })
    });

    load();
}

setInterval(load, 2000);
load();
</script>

</body>
</html>
