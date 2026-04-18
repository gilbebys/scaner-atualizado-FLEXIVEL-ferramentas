<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scanner Web Pro - Auto Park</title>
    <style>
        :root {
            --bg: #0f172a; --card: #1e293b; --border: #334155;
            --text-main: #f8fafc; --text-dim: #94a3b8;
            --blue: #38bdf8; --green: #22c55e; --purple: #a855f7; --orange: #f59e0b; --red: #ef4444;
        }
        body { font-family: 'Segoe UI', Tahoma, sans-serif; background: var(--bg); color: var(--text-main); margin: 0; padding: 20px; }
        .container { max-width: 1200px; margin: 0 auto; }
        
        /* Dashboard & Header */
        header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; border-bottom: 1px solid var(--border); padding-bottom: 10px; }
        .dashboard { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; margin-bottom: 20px; }
        .card { background: var(--card); border: 1px solid var(--border); padding: 15px; border-radius: 10px; border-left: 4px solid var(--blue); }
        .card p { margin: 0; font-size: 0.7rem; color: var(--text-dim); text-transform: uppercase; }
        .card h2 { margin: 5px 0 0; font-size: 1.5rem; }

        /* Controls */
        .controls { display: flex; gap: 10px; margin-bottom: 20px; flex-wrap: wrap; align-items: center; background: var(--card); padding: 15px; border-radius: 10px; border: 1px solid var(--border); }
        input, button { padding: 10px 15px; border-radius: 6px; border: 1px solid var(--border); background: #020617; color: white; outline: none; }
        #inputBase { border-color: var(--blue); font-weight: bold; width: 150px; text-align: center; }
        #btnScan { background: var(--green); color: #000; font-weight: bold; border: none; cursor: pointer; }
        #btnScan:disabled { opacity: 0.5; }
        .btn-excel { background: #1D6F42; color: white; font-weight: bold; border: none; cursor: pointer; } /* Cor do Excel */

        /* Table */
        table { width: 100%; border-collapse: collapse; background: var(--card); border-radius: 10px; overflow: hidden; }
        th { background: #334155; color: var(--blue); text-align: left; padding: 15px; font-size: 0.8rem; text-transform: uppercase; }
        td { padding: 12px 15px; border-bottom: 1px solid var(--border); font-size: 0.9rem; }
        tr:hover { background: #2d3748; }

        .name-input { background: rgba(0,0,0,0.3); border: 1px solid transparent; color: var(--purple); font-weight: bold; width: 90%; padding: 6px; border-radius: 4px; }
        .name-input:focus { border-color: var(--purple); }

        .latency-badge { padding: 4px 8px; border-radius: 5px; font-family: monospace; font-weight: bold; font-size: 0.8rem; background: rgba(34, 197, 94, 0.1); color: var(--green); }

        /* Tools */
        .action-group { display: flex; gap: 6px; }
        .btn-tool { text-decoration: none; font-size: 0.7rem; font-weight: bold; border-radius: 4px; padding: 6px 10px; color: #fff; transition: 0.2s; }
        .btn-web { background: var(--blue); color: #000; }
        .btn-putty { background: #512da8; }
        .btn-winscp { background: #d32f2f; }
        .btn-tool:hover { filter: brightness(1.2); transform: scale(1.05); }

        #progress-bar { height: 4px; background: var(--blue); width: 0%; transition: width 0.2s; margin-bottom: 10px; border-radius: 2px; }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>🌐 TI GILVAN Scanner</h1>
        <div id="clock" style="font-family: monospace; color: var(--text-dim);"></div>
    </header>

    <div class="dashboard">
        <div class="card">
            <p>Dispositivos Online</p>
            <h2 id="cOnline">0</h2>
        </div>
        <div class="card" style="border-color: var(--green)">
            <p>Rede Alvo</p>
            <h2 id="dispRede">---</h2>
        </div>
    </div>

    <div id="progress-bar"></div>

    <div class="controls">
        <div style="display:flex; flex-direction:column; gap:5px">
            <small style="color:var(--text-dim)">Faixa de IP (Base)</small>
            <input type="text" id="inputBase" value="192.168.142" placeholder="192.168.1">
        </div>
        <button id="btnScan" onclick="executarScan()">▶ INICIAR SCAN</button>
        <button class="btn-excel" onclick="exportarParaExcel()">📊 BAIXAR PLANILHA (EXCEL)</button>
        <input type="text" id="filtro" placeholder="Filtrar resultados..." onkeyup="filtrarTabela()" style="flex-grow:1">
    </div>

    <table>
        <thead>
            <tr>
                <th width="150">Endereço IP</th>
                <th>Nome / Identificação</th>
                <th width="120">Latência</th>
                <th width="250">Acesso Rápido</th>
            </tr>
        </thead>
        <tbody id="lista">
            <tr><td colspan="4" style="text-align:center; padding: 30px; color: var(--text-dim)">Defina a faixa de IP e clique em Iniciar Scan.</td></tr>
        </tbody>
    </table>
</div>

<script>
let dispositivosOnline = [];
let nomesSalvos = JSON.parse(localStorage.getItem('autopark_names')) || {};

function tocarBip() {
    const context = new (window.AudioContext || window.webkitAudioContext)();
    const osc = context.createOscillator();
    const gain = context.createGain();
    osc.connect(gain); gain.connect(context.destination);
    osc.frequency.value = 800; gain.gain.setValueAtTime(0.05, context.currentTime);
    osc.start(); osc.stop(context.currentTime + 0.1);
}

function salvarNome(ip, valor) {
    nomesSalvos[ip] = valor;
    localStorage.setItem('autopark_names', JSON.stringify(nomesSalvos));
}

async function executarScan() {
    const base = document.getElementById("inputBase").value;
    const btn = document.getElementById("btnScan");
    const prog = document.getElementById("progress-bar");
    
    document.getElementById("dispRede").innerText = base + ".x";
    document.getElementById("lista").innerHTML = "";
    dispositivosOnline = [];
    btn.disabled = true;
    btn.innerText = "Buscando...";

    for (let i = 1; i <= 254; i++) {
        const ip = `${base}.${i}`;
        verificarHost(ip);
        prog.style.width = (i / 254 * 100) + "%";
    }

    setTimeout(() => {
        btn.disabled = false;
        btn.innerText = "▶ INICIAR SCAN";
        if(dispositivosOnline.length === 0) {
            document.getElementById("lista").innerHTML = "<tr><td colspan='4' style='text-align:center'>Nenhum dispositivo encontrado na faixa " + base + ".</td></tr>";
        }
    }, 4000);
}

async function verificarHost(ip) {
    const controller = new AbortController();
    const timeout = setTimeout(() => controller.abort(), 3500);
    const start = performance.now();

    try {
        await fetch(`http://${ip}`, { mode: 'no-cors', signal: controller.signal });
        const lat = Math.round(performance.now() - start);
        adicionarLinha(ip, lat);
        tocarBip();
    } catch (e) { } finally { clearTimeout(timeout); }
}

function adicionarLinha(ip, lat) {
    if (dispositivosOnline.some(d => d.ip === ip)) return;
    
    const nome = nomesSalvos[ip] || "Equipamento Desconhecido";
    dispositivosOnline.push({ ip, nome, lat });
    dispositivosOnline.sort((a, b) => parseInt(a.ip.split('.')[3]) - parseInt(b.ip.split('.')[3]));

    renderizarTabela();
}

function renderizarTabela() {
    const tbody = document.getElementById("lista");
    tbody.innerHTML = "";
    dispositivosOnline.forEach(d => {
        tbody.innerHTML += `
            <tr>
                <td><strong style="color:var(--blue)">${d.ip}</strong></td>
                <td><input type="text" class="name-input" value="${d.nome}" onchange="salvarNome('${d.ip}', this.value)"></td>
                <td><span class="latency-badge">${d.lat} ms</span></td>
                <td>
                    <div class="action-group">
                        <a href="http://${d.ip}" target="_blank" class="btn-tool btn-web">Navegador</a>
                        <a href="ssh://${d.ip}" class="btn-tool btn-putty">PuTTY</a>
                        <a href="sftp://${d.ip}" class="btn-tool btn-winscp">WinSCP</a>
                    </div>
                </td>
            </tr>`;
    });
    document.getElementById("cOnline").innerText = dispositivosOnline.length;
}

function exportarParaExcel() {
    if (dispositivosOnline.length === 0) {
        alert("Não há dados para exportar. Faça um scan primeiro.");
        return;
    }

    // Criando o conteúdo do CSV (Excel entende CSV com separador ponto e vírgula)
    let csvContent = "\uFEFF"; // BOM para acentuação correta no Excel
    csvContent += "ENDERECO IP;IDENTIFICACAO;LATENCIA (MS)\n";

    dispositivosOnline.forEach(d => {
        const linha = `${d.ip};${d.nome};${d.lat}\n`;
        csvContent += linha;
    });

    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    
    const dataAtual = new Date().toLocaleDateString().replace(/\//g, '-');
    link.setAttribute("href", url);
    link.setAttribute("download", `Relatorio_Rede_AutoPark_${dataAtual}.csv`);
    link.style.visibility = 'hidden';
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
}

function filtrarTabela() {
    const busca = document.getElementById("filtro").value.toLowerCase();
    document.querySelectorAll("#lista tr").forEach(row => {
        const texto = row.innerText.toLowerCase() + row.querySelector('input').value.toLowerCase();
        row.style.display = texto.includes(busca) ? "" : "none";
    });
}

setInterval(() => { document.getElementById('clock').innerText = new Date().toLocaleString(); }, 1000);
</script>
</body>
</html>
