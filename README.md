
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gilvan Park | Pro Management</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.11.5/dist/JsBarcode.all.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800;900&display=swap');
        :root { --primary: #10b981; --dark: #0f172a; }
        body { font-family: 'Inter', sans-serif; background-color: #f8fafc; color: #1e293b; }
        
        /* Otimização de Impressão */
        @media print {
            body * { visibility: hidden; }
            #ticketPrint, #ticketPrint * { visibility: visible; }
            #ticketPrint { position: absolute; left: 0; top: 0; width: 100%; }
        }

        .custom-scroll::-webkit-scrollbar { width: 5px; }
        .custom-scroll::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        .tab-content { display: none; transition: all 0.3s ease; }
        .tab-content.active { display: block; animation: fadeIn 0.3s ease; }
        
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body class="h-screen flex flex-col overflow-hidden">

    <header class="bg-slate-900 text-white px-8 py-4 flex justify-between items-center shadow-lg border-b border-emerald-500/50">
        <div class="flex items-center gap-6">
            <div class="flex flex-col">
                <span class="text-emerald-500 font-black text-2xl tracking-tighter leading-none">GILVAN PARK</span>
                <span class="text-[10px] text-slate-400 font-bold tracking-[0.2em] uppercase">Enterprise v5.5</span>
            </div>
            <div class="h-8 w-[1px] bg-slate-700"></div>
            <div>
                <h2 id="headerShopName" class="text-sm font-bold text-slate-200">Carregando...</h2>
                <div class="flex items-center gap-2">
                    <span id="statusCaixaDot" class="w-2 h-2 rounded-full bg-red-500 animate-pulse"></span>
                    <span id="statusCaixaText" class="text-[10px] font-bold text-slate-400 uppercase">Aguardando Abertura</span>
                </div>
            </div>
        </div>
        <div class="text-right">
            <div id="relogio" class="font-mono text-xl font-black text-white">00:00:00</div>
            <div id="dataAtual" class="text-[10px] font-bold text-emerald-500 uppercase">--/--/----</div>
        </div>
    </header>

    <div class="flex flex-1 overflow-hidden">
        <aside class="w-85 bg-white border-r border-slate-200 p-6 flex flex-col gap-6 shadow-sm z-20">
            <div class="space-y-4">
                <div class="group">
                    <label class="text-[10px] font-black text-slate-400 uppercase mb-2 block tracking-widest">Entrada Rápida</label>
                    <div class="relative">
                        <input id="mainInput" type="text" autofocus autocomplete="off"
                            class="w-full bg-slate-50 border-2 border-slate-200 focus:border-emerald-500 focus:ring-4 focus:ring-emerald-500/10 rounded-2xl p-4 text-2xl font-black text-center uppercase outline-none transition-all" 
                            placeholder="Placa / Ticket">
                    </div>
                </div>
                
                <div class="grid grid-cols-3 gap-2">
                    <button onclick="setVehicleType('CARRO')" id="btn-CARRO" class="type-btn p-3 bg-slate-100 rounded-xl border-2 border-transparent hover:bg-emerald-50 font-bold transition-all flex flex-col items-center">
                        <span class="text-xl">🚗</span><span class="text-[9px] mt-1">CARRO</span>
                    </button>
                    <button onclick="setVehicleType('MOTO')" id="btn-MOTO" class="type-btn p-3 bg-slate-100 rounded-xl border-2 border-transparent hover:bg-emerald-50 font-bold transition-all flex flex-col items-center">
                        <span class="text-xl">🏍️</span><span class="text-[9px] mt-1">MOTO</span>
                    </button>
                    <button onclick="setVehicleType('CAMINHAO')" id="btn-CAMINHAO" class="type-btn p-3 bg-slate-100 rounded-xl border-2 border-transparent hover:bg-emerald-50 font-bold transition-all flex flex-col items-center">
                        <span class="text-xl">🚚</span><span class="text-[9px] mt-1">PESADO</span>
                    </button>
                </div>
            </div>

            <div id="checkoutPanel" class="hidden flex-1 flex flex-col bg-emerald-600 rounded-[2.5rem] p-6 text-white shadow-2xl animate-slide-up relative overflow-hidden">
                <div class="absolute top-0 right-0 p-4">
                    <button onclick="closeCheckout()" class="text-white/50 hover:text-white transition-all text-2xl">&times;</button>
                </div>
                
                <div class="text-center mb-6">
                    <span class="text-[10px] font-black uppercase tracking-widest opacity-60">Veículo Identificado</span>
                    <h3 id="chkPlaca" class="text-4xl font-black tracking-tighter">---</h3>
                    <span id="chkId" class="text-[10px] font-mono opacity-80">ID: #000000</span>
                </div>

                <div class="bg-white/10 backdrop-blur-md rounded-2xl p-4 mb-6 border border-white/20">
                    <div class="flex justify-between items-center mb-2">
                        <span class="text-[10px] font-bold uppercase">Tempo</span>
                        <span id="chkTempo" class="font-black text-sm text-emerald-200">0 min</span>
                    </div>
                    <div class="flex justify-between items-center">
                        <span class="text-[10px] font-bold uppercase">Valor</span>
                        <span id="chkValor" class="text-3xl font-black">R$ 0,00</span>
                    </div>
                </div>

                <div class="space-y-2 mt-auto">
                    <select id="convSelect" onchange="logic.updatePrice()" class="w-full bg-white text-slate-900 p-3 rounded-xl font-bold text-xs outline-none border-none shadow-lg"></select>
                    <div class="grid grid-cols-2 gap-2 mt-2">
                        <button onclick="logic.checkout('DINHEIRO')" class="bg-slate-900 text-white p-3 rounded-xl font-black text-[10px] hover:scale-105 transition-transform">DINHEIRO</button>
                        <button onclick="logic.checkout('PIX')" class="bg-slate-900 text-white p-3 rounded-xl font-black text-[10px] hover:scale-105 transition-transform">PIX</button>
                        <button onclick="logic.checkout('DEBITO')" class="bg-white/20 text-white p-3 rounded-xl font-black text-[10px] hover:bg-white/30">DÉBITO</button>
                        <button onclick="logic.checkout('CREDITO')" class="bg-white/20 text-white p-3 rounded-xl font-black text-[10px] hover:bg-white/30">CRÉDITO</button>
                    </div>
                </div>
            </div>

            <div class="mt-auto bg-slate-900 rounded-3xl p-5 shadow-inner">
                <div class="flex justify-between items-center mb-3">
                    <span class="text-[9px] font-black text-emerald-500 uppercase">Saldo em Caixa</span>
                    <span id="cashValue" class="text-lg font-black text-white">R$ 0,00</span>
                </div>
                <button id="cashBtn" onclick="logic.toggleCashier()" class="w-full py-3 bg-emerald-500 text-slate-900 rounded-xl font-black text-xs uppercase hover:bg-emerald-400 transition-all shadow-lg">Abrir Caixa</button>
            </div>
        </aside>

        <main class="flex-1 flex flex-col overflow-hidden">
            <nav class="bg-white px-8 py-2 border-b flex gap-10">
                <button onclick="showTab('tab-patio')" class="tab-link border-b-2 border-emerald-500 py-4 text-xs font-black uppercase text-slate-900">Pátio Vivo</button>
                <button onclick="showTab('tab-rates')" class="tab-link border-b-2 border-transparent py-4 text-xs font-black uppercase text-slate-400 hover:text-slate-600">Tarifários</button>
                <button onclick="showTab('tab-conv')" class="tab-link border-b-2 border-transparent py-4 text-xs font-black uppercase text-slate-400 hover:text-slate-600">Parcerias</button>
                <button onclick="showTab('tab-fin')" class="tab-link border-b-2 border-transparent py-4 text-xs font-black uppercase text-slate-400 hover:text-slate-600">Financeiro</button>
                <button onclick="showTab('tab-settings')" class="tab-link border-b-2 border-transparent py-4 text-xs font-black uppercase text-slate-400 hover:text-slate-600">Sistema</button>
            </nav>

            <div class="flex-1 overflow-y-auto p-8 custom-scroll">
                <section id="tab-patio" class="tab-content active">
                    <div id="patioGrid" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-4"></div>
                </section>

                <section id="tab-rates" class="tab-content">
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-6" id="ratesContainer"></div>
                    <div class="mt-8 flex justify-end">
                        <button onclick="logic.saveRates()" class="bg-slate-900 text-white px-10 py-4 rounded-2xl font-black uppercase shadow-xl hover:bg-black">Atualizar Tabela de Preços</button>
                    </div>
                </section>

                <section id="tab-fin" class="tab-content">
                    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                        <div class="lg:col-span-2 bg-white p-6 rounded-3xl border shadow-sm">
                            <canvas id="mainChart" height="150"></canvas>
                        </div>
                        <div class="space-y-6">
                            <div class="bg-emerald-500 p-8 rounded-3xl text-white shadow-lg">
                                <span class="text-[10px] font-black uppercase">Faturado Hoje</span>
                                <h2 id="totalToday" class="text-4xl font-black">R$ 0,00</h2>
                            </div>
                            <div id="methodBreakdown" class="bg-white p-6 rounded-3xl border shadow-sm space-y-2"></div>
                        </div>
                    </div>
                </section>

                <section id="tab-settings" class="tab-content">
                    <div class="max-w-xl bg-white p-8 rounded-3xl border shadow-sm space-y-4">
                        <h3 class="font-black text-emerald-600 italic uppercase">Configurações do Negócio</h3>
                        <input id="set-name" placeholder="NOME DO ESTABELECIMENTO" class="w-full p-3 border rounded-xl font-bold uppercase">
                        <input id="set-cnpj" placeholder="CNPJ" class="w-full p-3 border rounded-xl font-bold">
                        <input id="set-tel" placeholder="TELEFONE" class="w-full p-3 border rounded-xl font-bold">
                        <input id="set-end" placeholder="ENDEREÇO" class="w-full p-3 border rounded-xl font-bold uppercase">
                        <button onclick="logic.saveSettings()" class="w-full bg-slate-900 text-white py-4 rounded-xl font-black uppercase">Salvar Alterações</button>
                    </div>
                </section>

                <section id="tab-conv" class="tab-content">
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                        <div class="bg-white p-6 rounded-3xl border shadow-sm h-fit">
                            <h3 class="font-black text-emerald-600 italic uppercase mb-4">Novo Convênio</h3>
                            <input id="conv-name" placeholder="NOME DO PARCEIRO" class="w-full p-3 border rounded-xl font-bold uppercase mb-2">
                            <input id="conv-val" type="number" placeholder="DESCONTO %" class="w-full p-3 border rounded-xl font-bold mb-4">
                            <button onclick="logic.addConv()" class="w-full bg-emerald-500 text-slate-900 py-3 rounded-xl font-black uppercase">Cadastrar</button>
                        </div>
                        <div id="convList" class="md:col-span-2 grid grid-cols-1 sm:grid-cols-2 gap-4"></div>
                    </div>
                </section>
            </div>
        </main>
    </div>

    <div id="ticketPrint" class="hidden"></div>

    <script>
        // --- LOGIC CLASS (Estrutura Comercial) ---
        class ParkLogic {
            constructor() {
                this.db = this.loadDB();
                this.currentType = 'CARRO';
                this.activeCheckout = null;
                this.init();
            }

            loadDB() {
                const defaults = {
                    patio: [], sales: [], convs: [],
                    settings: { nome: "GILVAN PARK PRO", cnpj: "00.000.000/0001-00", tel: "(00) 0000-0000", end: "CENTRO" },
                    rates: {
                        CARRO: { frac: 10, hour: 20 },
                        MOTO: { frac: 5, hour: 10 },
                        CAMINHAO: { frac: 20, hour: 40 }
                    },
                    cashier: { open: false, balance: 0 }
                };
                return { ...defaults, ...JSON.parse(localStorage.getItem('gp_pro_db')) };
            }

            save() {
                localStorage.setItem('gp_pro_db', JSON.stringify(this.db));
                this.render();
            }

            init() {
                this.render();
                setInterval(() => {
                    document.getElementById('relogio').innerText = new Date().toLocaleTimeString();
                    document.getElementById('dataAtual').innerText = new Date().toLocaleDateString();
                }, 1000);

                document.getElementById('mainInput').addEventListener('keypress', (e) => {
                    if(e.key === 'Enter') this.handleInput();
                });
            }

            handleInput() {
                if(!this.db.cashier.open) return alert("Abra o caixa!");
                const val = document.getElementById('mainInput').value.toUpperCase().trim();
                if(val.length < 3) return;

                const match = this.db.patio.find(v => v.placa === val || v.id.toString().slice(-6) === val);
                if(match) this.openCheckout(match);
                else this.entry(val);

                document.getElementById('mainInput').value = '';
            }

            entry(placa) {
                if(placa.length < 7) return alert("Placa inválida!");
                const novo = { id: Date.now(), placa, type: this.currentType, time: new Date().toISOString() };
                this.db.patio.push(novo);
                this.save();
                this.printTicket(novo);
            }

            openCheckout(v) {
                this.activeCheckout = v;
                document.getElementById('checkoutPanel').classList.remove('hidden');
                this.updatePrice();
            }

            updatePrice() {
                if(!this.activeCheckout) return;
                const v = this.activeCheckout;
                const min = Math.ceil((new Date() - new Date(v.time)) / 60000);
                const rate = this.db.rates[v.type];
                const discount = parseFloat(document.getElementById('convSelect').value) || 0;

                let price = (min <= 15) ? rate.frac : Math.ceil(min/60) * rate.hour;
                price = price * (1 - (discount/100));

                document.getElementById('chkPlaca').innerText = v.placa;
                document.getElementById('chkId').innerText = `#${v.id.toString().slice(-6)}`;
                document.getElementById('chkTempo').innerText = `${min} min`;
                document.getElementById('chkValor').innerText = `R$ ${price.toFixed(2)}`;
            }

            checkout(method) {
                const val = parseFloat(document.getElementById('chkValor').innerText.replace('R$ ',''));
                this.db.sales.push({ date: new Date().toISOString(), value: val, method, type: this.activeCheckout.type });
                if(method === 'DINHEIRO') this.db.cashier.balance += val;
                
                this.db.patio = this.db.patio.filter(v => v.id !== this.activeCheckout.id);
                this.activeCheckout = null;
                this.save();
                closeCheckout();
                alert("Checkout Finalizado!");
            }

            toggleCashier() {
                if(!this.db.cashier.open) {
                    const start = prompt("Fundo de troco:", "0");
                    if(start === null) return;
                    this.db.cashier.open = true;
                    this.db.cashier.balance = parseFloat(start) || 0;
                } else {
                    if(confirm("Deseja fechar o caixa e encerrar turno?")) this.db.cashier.open = false;
                }
                this.save();
            }

            // --- UI RENDERERS ---
            render() {
                // Header & Settings
                document.getElementById('headerShopName').innerText = this.db.settings.nome;
                document.getElementById('set-name').value = this.db.settings.nome;
                document.getElementById('set-cnpj').value = this.db.settings.cnpj;
                document.getElementById('set-tel').value = this.db.settings.tel;
                document.getElementById('set-end').value = this.db.settings.end;

                // Caixa
                document.getElementById('cashValue').innerText = `R$ ${this.db.cashier.balance.toFixed(2)}`;
                document.getElementById('cashBtn').innerText = this.db.cashier.open ? "Fechar Turno" : "Abrir Caixa";
                document.getElementById('statusCaixaDot').className = `w-2 h-2 rounded-full ${this.db.cashier.open ? 'bg-emerald-500' : 'bg-red-500 animate-pulse'}`;
                document.getElementById('statusCaixaText').innerText = this.db.cashier.open ? "Sistema Ativo" : "Caixa Fechado";

                // Patio
                const grid = document.getElementById('patioGrid');
                grid.innerHTML = this.db.patio.map(v => `
                    <div onclick="logic.openCheckout(${JSON.stringify(v).replace(/"/g, '&quot;')})" class="bg-white border border-slate-200 p-4 rounded-3xl hover:border-emerald-500 cursor-pointer shadow-sm transition-all group">
                        <div class="flex justify-between items-start mb-2">
                            <span class="text-2xl">${{CARRO:'🚗',MOTO:'🏍️',CAMINHAO:'🚚'}[v.type]}</span>
                            <span class="text-[9px] font-black text-emerald-500 bg-emerald-50 px-2 py-1 rounded-full">${Math.ceil((new Date()-new Date(v.time))/60000)}m</span>
                        </div>
                        <h4 class="font-black text-slate-800 text-lg group-hover:text-emerald-600">${v.placa}</h4>
                    </div>
                `).reverse().join('');

                // Rates
                document.getElementById('ratesContainer').innerHTML = Object.keys(this.db.rates).map(r => `
                    <div class="bg-white p-6 rounded-3xl border shadow-sm">
                        <h4 class="font-black mb-4 uppercase text-slate-400 text-xs tracking-widest">${r}</h4>
                        <div class="space-y-3">
                            <div class="flex justify-between items-center text-sm font-bold">
                                <span>Tolerância/Frac</span>
                                <input id="rate-${r}-frac" type="number" value="${this.db.rates[r].frac}" class="w-16 p-2 bg-slate-50 rounded-lg text-center">
                            </div>
                            <div class="flex justify-between items-center text-sm font-bold">
                                <span>Hora Cheia</span>
                                <input id="rate-${r}-hour" type="number" value="${this.db.rates[r].hour}" class="w-16 p-2 bg-slate-50 rounded-lg text-center">
                            </div>
                        </div>
                    </div>
                `).join('');

                // Convs
                document.getElementById('convSelect').innerHTML = '<option value="0">Tabela Padrão</option>' + 
                    this.db.convs.map(c => `<option value="${c.val}">${c.name} (-${c.val}%)</option>`).join('');
                
                document.getElementById('convList').innerHTML = this.db.convs.map(c => `
                    <div class="bg-white p-4 rounded-2xl border flex justify-between items-center">
                        <div><p class="font-black text-xs uppercase">${c.name}</p><span class="text-[10px] text-emerald-500 font-bold">${c.val}% de Desconto</span></div>
                        <button onclick="logic.delConv(${c.id})" class="text-red-400 hover:text-red-600 font-bold text-xs">Excluir</button>
                    </div>
                `).join('');

                this.renderChart();
            }

            renderChart() {
                const hoje = new Date().toISOString().split('T')[0];
                const vendasHoje = this.db.sales.filter(s => s.date.startsWith(hoje));
                const total = vendasHoje.reduce((a,b) => a + b.value, 0);
                document.getElementById('totalToday').innerText = `R$ ${total.toFixed(2)}`;

                const ctx = document.getElementById('mainChart').getContext('2d');
                if(window.myChart) window.myChart.destroy();
                window.myChart = new Chart(ctx, {
                    type: 'bar',
                    data: {
                        labels: ['Dinheiro', 'PIX', 'Débito', 'Crédito'],
                        datasets: [{
                            label: 'Vendas Hoje',
                            data: ['DINHEIRO','PIX','DEBITO','CREDITO'].map(m => vendasHoje.filter(s => s.method === m).reduce((a,b)=>a+b.value,0)),
                            backgroundColor: '#10b981', borderRadius: 10
                        }]
                    },
                    options: { responsive: true, plugins: { legend: { display: false } } }
                });
            }

            // --- CRUD HELPERS ---
            saveRates() {
                Object.keys(this.db.rates).forEach(r => {
                    this.db.rates[r].frac = parseFloat(document.getElementById(`rate-${r}-frac`).value);
                    this.db.rates[r].hour = parseFloat(document.getElementById(`rate-${r}-hour`).value);
                });
                this.save();
                alert("Configurações Salvas!");
            }

            saveSettings() {
                this.db.settings = {
                    nome: document.getElementById('set-name').value.toUpperCase(),
                    cnpj: document.getElementById('set-cnpj').value,
                    tel: document.getElementById('set-tel').value,
                    end: document.getElementById('set-end').value.toUpperCase()
                };
                this.save();
                alert("Dados da empresa atualizados!");
            }

            addConv() {
                const name = document.getElementById('conv-name').value;
                const val = document.getElementById('conv-val').value;
                if(!name || !val) return;
                this.db.convs.push({ id: Date.now(), name: name.toUpperCase(), val: parseFloat(val) });
                this.save();
                document.getElementById('conv-name').value = '';
                document.getElementById('conv-val').value = '';
            }

            delConv(id) { this.db.convs = this.db.convs.filter(c => c.id !== id); this.save(); }

            printTicket(v) {
                const html = `
                    <div style="text-align:center; font-family:monospace; padding:20px; border:1px solid #000; width:300px">
                        <h2 style="margin:0">${this.db.settings.nome}</h2>
                        <p style="font-size:10px">${this.db.settings.end}</p>
                        <hr>
                        <h1 style="font-size:40px; margin:10px 0">${v.placa}</h1>
                        <p>${v.type} | Entrada: ${new Date(v.time).toLocaleTimeString()}</p>
                        <hr>
                        <p style="font-size:9px">Conserve este ticket.</p>
                    </div>`;
                document.getElementById('ticketPrint').innerHTML = html;
                window.print();
            }
        }

        // --- GLOBAL UI FUNCTIONS ---
        const logic = new ParkLogic();

        function setVehicleType(type) {
            logic.currentType = type;
            document.querySelectorAll('.type-btn').forEach(b => b.classList.replace('border-emerald-500', 'border-transparent'));
            document.getElementById(`btn-${type}`).classList.replace('border-transparent', 'border-emerald-500');
            document.getElementById('mainInput').focus();
        }

        function closeCheckout() { 
            document.getElementById('checkoutPanel').classList.add('hidden'); 
            logic.activeCheckout = null; 
            document.getElementById('mainInput').focus();
        }

        function showTab(id) {
            document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
            document.querySelectorAll('.tab-link').forEach(l => l.classList.replace('border-emerald-500', 'border-transparent'));
            document.querySelectorAll('.tab-link').forEach(l => l.classList.replace('text-slate-900', 'text-slate-400'));
            
            document.getElementById(id).classList.add('active');
            event.currentTarget.classList.replace('border-transparent', 'border-emerald-500');
            event.currentTarget.classList.replace('text-slate-400', 'text-slate-900');
        }

        setVehicleType('CARRO');
    </script>
</body>
</html>
