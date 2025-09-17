<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Calcolatore Multi-Canale Moderno</title>
<style>
body { font-family: Arial, sans-serif; padding: 20px; transition: background-color 0.3s; }

.channel { border: 1px solid #ccc; padding: 10px; margin-bottom: 10px; border-radius: 8px; background: #fff; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
.subchannel {
    display: inline-block;
    background: #f9f9f9;
    border: 1px solid #ccc;
    padding: 10px;
    margin: 5px;
    border-radius: 6px;
    min-width: 100px;
    text-align: center;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    font-family: 'Courier New', monospace;
}
.subchannel button {
    margin: 5px 2px;
    padding: 5px 10px;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-weight: bold;
    transition: all 0.2s ease;
}
.subchannel button.increment { background-color: #4CAF50; color: white; }
.subchannel button.increment:hover { background-color: #45a049; }
.subchannel button.decrement { background-color: #f44336; color: white; }
.subchannel button.decrement:hover { background-color: #d32f2f; }

table { border-collapse: collapse; margin-top: 10px; width: 100%; background: #fff; }
table, th, td { border: 1px solid #aaa; }
th, td { padding: 5px; text-align: left; }
.window { margin: 5px 0; }
.flash { background-color: yellow !important; }

/* Menu a tendina generale */
.dropdown-menu {
  position: fixed;
  top: 10px;
  right: 10px;
  z-index: 1001;
  font-family: Arial, sans-serif;
}

.dropbtn {
  background-color: #4CAF50;
  color: white;
  padding: 8px 14px;
  font-size: 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s;
}

.dropbtn:hover {
  background-color: #45a049;
}

.dropdown-content {
  display: none;
  position: absolute;
  right: 0;
  top: 40px;
  background-color: #f1f1f1;
  min-width: 320px;
  max-height: 500px;
  overflow-y: auto;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
  padding: 15px;
  z-index: 1000;
}

.dropdown-content label,
.dropdown-content input,
.dropdown-content select,
.dropdown-content button {
  display: block;
  margin-top: 10px;
  width: 100%;
  box-sizing: border-box;
}

.dropdown-content hr {
  margin: 15px 0;
}

.dropdown-menu.show .dropdown-content {
  display: block;
}
</style>
</head>
<body>

<h2>Calcolatore Multi-Canale</h2>
<div id="clock" style="font-size:18px; font-weight:bold; margin-bottom:10px;"></div>

<div id="channels"></div>

<!-- Menu a tendina generale -->
<div class="dropdown-menu">
  <button class="dropbtn">⚙️ Menu</button>
  <div class="dropdown-content" id="generalMenu">
    <label>Seleziona Canale</label>
    <select id="menuChannelSelect" onchange="updateSubSelect()"></select>
    <button onclick="menuAddChannel()">Aggiungi Canale</button>
    <button onclick="menuDeleteChannel()">Elimina Canale</button>

    <hr>

    <label>Seleziona Subcanale</label>
    <select id="menuSubSelect"></select>
    <button onclick="menuAddSub()">Aggiungi Subcanale</button>
    <button onclick="menuDeleteSub()">Elimina Subcanale</button>

    <hr>

    <label>Rinomina Canale</label>
    <input type="text" id="menuRenameChannel" placeholder="Nuovo nome canale">
    <button onclick="menuRenameChannel()">Applica</button>

    <label>Rinomina Subcanale</label>
    <input type="text" id="menuRenameSub" placeholder="Nuovo nome subcanale">
    <button onclick="menuRenameSub()">Applica</button>

    <label>Cambia colore Canale</label>
    <input type="color" id="menuColor" onchange="menuChangeColor(this.value)">

    <hr>

    <label>Finestre Orarie</label>
    <input type="time" id="startTime">
    <input type="time" id="endTime">
    <button onclick="addWindow()">Aggiungi Finestra</button>
    <div id="windows"></div>

    <hr>

    <button onclick="exportHistoryCSV()">Esporta CSV Storico</button>
    <button onclick="exportHistoryExcel()">Esporta Excel Storico</button>
    <button onclick="resetAll()">Reset Totale Contatori</button>
    <button onclick="clearHistory()">Elimina Tabella Storico</button>
  </div>
</div>

<audio id="alertSound">
    <source src="https://actions.google.com/sounds/v1/alarms/alarm_clock.ogg" type="audio/ogg">
</audio>

<h3>📜 Storico Finestre</h3>
<div id="historyContainer"></div>

<script>
// =========================
// Variabili principali
// =========================
let channels = [];
let windows = [];
let lastWindow = null;

// =========================
// Menu toggle generale
// =========================
document.querySelector(".dropbtn").onclick = () => {
    document.querySelector(".dropdown-menu").classList.toggle("show");
};

window.onclick = function(e) {
    const menu = document.querySelector(".dropdown-menu");
    if (!menu.contains(e.target)) {
        menu.classList.remove("show");
    }
};

// =========================
// Carica e salva dati
// =========================
function loadData() {
    const data = localStorage.getItem("channelsData");
    if (data) channels = JSON.parse(data);
    const winData = localStorage.getItem("windowsData");
    if (winData) windows = JSON.parse(winData);

    if (channels.length === 0) {
        const defaultChannel = {name: "Veicoli", subchannels: [
            {name: "AUTO", count:0},
            {name: "VEIC. COM", count:0},
            {name: "VEIC. Pes.", count:0},
            {name: "Bus", count:0}
        ], color:"#ffffff"};
        channels.push(defaultChannel);
        saveData();
    }
    populateMenuChannel();
    updateHistoryTables();
}
function saveData() { localStorage.setItem("channelsData", JSON.stringify(channels)); }
function saveWindows() { localStorage.setItem("windowsData", JSON.stringify(windows)); renderWindows(); }

// =========================
// Popolamento menu
// =========================
function populateMenuChannel() {
    const select = document.getElementById("menuChannelSelect");
    select.innerHTML = "";
    channels.forEach((c,i)=>{
        const opt = document.createElement("option");
        opt.value = i;
        opt.text = c.name;
        select.add(opt);
    });
    updateSubSelect();
}
function updateSubSelect() {
    const chIndex = parseInt(document.getElementById("menuChannelSelect").value);
    const select = document.getElementById("menuSubSelect");
    select.innerHTML = "";
    if (channels[chIndex]) {
        channels[chIndex].subchannels.forEach((s,i)=>{
            const opt = document.createElement("option");
            opt.value = i;
            opt.text = s.name;
            select.add(opt);
        });
    }
}

// =========================
// Menu actions
// =========================
function menuRenameChannel() {
    const chIndex = parseInt(document.getElementById("menuChannelSelect").value);
    const name = document.getElementById("menuRenameChannel").value.trim();
    if (name) { channels[chIndex].name = name; render(); saveData(); populateMenuChannel(); updateHistoryTables(); }
}
function menuRenameSub() {
    const chIndex = parseInt(document.getElementById("menuChannelSelect").value);
    const subIndex = parseInt(document.getElementById("menuSubSelect").value);
    const name = document.getElementById("menuRenameSub").value.trim();
    if (name) { channels[chIndex].subchannels[subIndex].name = name; render(); saveData(); updateSubSelect(); updateHistoryTables(); }
}
function menuChangeColor(color) {
    const chIndex = parseInt(document.getElementById("menuChannelSelect").value);
    channels[chIndex].color = color; render(); saveData();
}
function menuAddChannel() {
    const channel = {name:`Canale ${channels.length+1}`, subchannels:[], color:"#ffffff"};
    channels.push(channel); render(); saveData(); populateMenuChannel(); updateHistoryTables();
}
function menuAddSub() {
    const chIndex = parseInt(document.getElementById("menuChannelSelect").value);
    const name = prompt("Nome del subcanale:");
    if (!name) return;
    channels[chIndex].subchannels.push({name: name, count:0});
    render(); saveData(); updateSubSelect(); updateHistoryTables();
}
function menuDeleteChannel() {
    const chIndex = parseInt(document.getElementById("menuChannelSelect").value);
    if(confirm(`Eliminare il canale "${channels[chIndex].name}"?`)) {
        channels.splice(chIndex,1); render(); saveData(); populateMenuChannel(); updateHistoryTables();
    }
}
function menuDeleteSub() {
    const chIndex = parseInt(document.getElementById("menuChannelSelect").value);
    const subIndex = parseInt(document.getElementById("menuSubSelect").value);
    if(confirm(`Eliminare il subcanale "${channels[chIndex].subchannels[subIndex].name}"?`)) {
        channels[chIndex].subchannels.splice(subIndex,1); render(); saveData(); updateSubSelect(); updateHistoryTables();
    }
}
function clearHistory() {
    if(confirm("Eliminare tutta la tabella storico finestre?")) {
        channels.forEach((c,i) => localStorage.setItem(`historyData_${i}`, ""));
        updateHistoryTables();
    }
}

// =========================
// Incremento/Decremento
// =========================
function increment(c,s){ channels[c].subchannels[s].count+=1; render(); saveData(); }
function decrement(c,s){ channels[c].subchannels[s].count-=1; render(); saveData(); }
function resetAll(){ if(!confirm("Azzerare tutti i contatori?")) return; channels.forEach(c=>c.subchannels.forEach(s=>s.count=0)); render(); saveData(); }

// =========================
// Render canali
// =========================
function render(){
    const container = document.getElementById("channels");
    container.innerHTML="";
    channels.forEach((c,i)=>{
        const div = document.createElement("div");
        div.className = "channel";
        div.style.backgroundColor = c.color;
        const title = document.createElement("h3");
        title.textContent = c.name;
        div.appendChild(title);
        c.subchannels.forEach((s,j)=>{
            const subDiv = document.createElement("div");
            subDiv.className = "subchannel";
            const label = document.createElement("div");
            label.textContent = `${s.name}: ${s.count}`;
            subDiv.appendChild(label);
            const inc = document.createElement("button");
            inc.textContent = "+1";
            inc.className = "increment";
            inc.onclick = ()=>increment(i,j);
            const dec = document.createElement("button");
            dec.textContent = "-1";
            dec.className = "decrement";
            dec.onclick = ()=>decrement(i,j);
            subDiv.appendChild(inc);
            subDiv.appendChild(dec);
            div.appendChild(subDiv);
        });
        container.appendChild(div);
    });
}

// =========================
// Storico dinamico per canale
// =========================
function updateHistoryTables() {
    const container = document.getElementById("historyContainer");
    container.innerHTML = "";
    channels.forEach((c, i) => {
        const div = document.createElement("div");
        div.style.marginBottom = "20px";
        const title = document.createElement("h4");
        title.textContent = `Storico: ${c.name}`;
        div.appendChild(title);

        const table = document.createElement("table");
        table.id = `historyTable_${i}`;
        table.style.width = "100%";
        table.style.borderCollapse = "collapse";

        const thead = document.createElement("thead");
        const headerRow = document.createElement("tr");
        headerRow.innerHTML = "<th>Ora chiusura</th>";
        c.subchannels.forEach(s => {
            const th = document.createElement("th");
            th.textContent = s.name;
            headerRow.appendChild(th);
        });
        thead.appendChild(headerRow);
        table.appendChild(thead);

        const tbody = document.createElement("tbody");
        const histData = localStorage.getItem(`historyData_${i}`);
        if (histData) tbody.innerHTML = histData;
        table.appendChild(tbody);

        div.appendChild(table);
        container.appendChild(div);
    });
}

function saveHistory() {
    const timestamp = new Date().toLocaleTimeString();
    channels.forEach((c, i) => {
        const table = document.getElementById(`historyTable_${i}`);
        const tbody = table.querySelector("tbody");
        const row = document.createElement("tr");
        row.innerHTML = `<td>${timestamp}</td>`;
        c.subchannels.forEach(s => {
            row.innerHTML += `<td>${s.count}</td>`;
        });
        tbody.appendChild(row);
        localStorage.setItem(`historyData_${i}`, tbody.innerHTML);
    });
}

// =========================
// Finestre
// =========================
function addWindow(){ 
    const start=document.getElementById("startTime").value; 
    const end=document.getElementById("endTime").value; 
    if(!start||!end) return alert("Orari non validi"); 
    windows.push({start,end}); 
    saveWindows(); 
}
function removeWindow(i){ windows.splice(i,1); saveWindows(); }
function renderWindows(){ 
    const div=document.getElementById("windows"); 
    div.innerHTML=""; 
    windows.forEach((w,i)=>{
        const el=document.createElement("div"); 
        el.className="window"; 
        el.innerHTML=`🕒 ${w.start} - ${w.end} <button onclick="removeWindow(${i})">Elimina</button>`; 
        div.appendChild(el); 
    }); 
}

// =========================
// Check finestre e notifiche
// =========================
function checkWindows(){ 
    const now=new Date(); 
    const current=now.toTimeString().slice(0,5); 
    windows.forEach(w=>{
        if(current===w.end){
            if(lastWindow!==w.start+w.end){
                saveHistory(); resetCounters(); triggerNotification(); lastWindow=w.start+w.end;
            }
        }
    }); 
}
function resetCounters(){ channels.forEach(c=>c.subchannels.forEach(s=>s.count=0)); render(); saveData(); }
function triggerNotification(){ document.getElementById("alertSound").play(); document.body.classList.add("flash"); setTimeout(()=>document.body.classList.remove("flash"),2000); }
function updateClock(){ document.getElementById("clock").textContent="⏰ Ora: "+new Date().toLocaleTimeString(); }

// =========================
// Esporta CSV / Excel
// =========================
function exportHistoryCSV(){ 
    channels.forEach((c,i) => {
        const tbody = document.querySelector(`#historyTable_${i} tbody`);
        let csvHeaders = ["Ora chiusura"];
        c.subchannels.forEach(s => csvHeaders.push(s.name));
        let csv = "\ufeff" + csvHeaders.join(";") + "\n";
        Array.from(tbody.querySelectorAll("tr")).forEach(r => {
            const cells = r.querySelectorAll("td"); 
            csv += Array.from(cells).map(c => c.textContent).join(";") + "\n"; 
        });
        const blob = new Blob([csv],{type:"text/csv;charset=utf-8;"}); 
        const url = URL.createObjectURL(blob); 
        const link = document.createElement("a"); 
        link.href = url; 
        link.download = `storico_${c.name}.csv`; 
        document.body.appendChild(link); 
        link.click(); 
        document.body.removeChild(link); 
    });
}

function exportHistoryExcel(){
    channels.forEach((c,i) => {
        const tbody = document.querySelector(`#historyTable_${i} tbody`);
        let csvHeaders = ["Ora chiusura"];
        c.subchannels.forEach(s => csvHeaders.push(s.name));
        let csv = "\ufeff" + csvHeaders.join(";") + "\n";
        Array.from(tbody.querySelectorAll("tr")).forEach(r => {
            const cells = r.querySelectorAll("td"); 
            csv += Array.from(cells).map(c => c.textContent).join(";") + "\n"; 
        });
        const blob = new Blob([csv], {type: "application/vnd.ms-excel"});
        const url = URL.createObjectURL(blob);
        const link = document.createElement("a");
        link.href = url;
        link.download = `storico_${c.name}.xls`;
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    });
}

// =========================
// Avvio
// =========================
loadData();
render();
renderWindows();
setInterval(updateClock,1000);
setInterval(checkWindows,1000);

</script>
</body>
</html>
