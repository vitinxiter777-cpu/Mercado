<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Atacadão Store - Mobile PDV</title>
    <script src="https://unpkg.com/html5-qrcode"></script>
    <style>
        body { font-family: sans-serif; background: #f0f2f5; margin: 0; padding: 0; }
        #tela-bloqueio { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #1a237e; display: flex; justify-content: center; align-items: center; z-index: 9999; }
        .card-login { background: white; padding: 30px; border-radius: 8px; width: 90%; max-width: 360px; text-align: center; box-shadow: 0 4px 15px rgba(0,0,0,0.3); }
        #sistema-principal { display: none; }
        
        .topo-sistema { display: flex; background: #1a237e; align-items: center; justify-content: space-between; border-bottom: 2px solid #0d47a1; }
        .menu-abas { display: flex; flex: 1; }
        .aba-botao { flex: 1; padding: 15px 5px; text-align: center; color: white; font-weight: bold; cursor: pointer; border: none; background: none; font-size: 14px; }
        .aba-botao.ativa { background: #0d47a1; border-bottom: 4px solid #29b6f6; }
        
        .btn-sair { background: #d32f2f; color: white; border: none; padding: 10px 12px; font-weight: bold; border-radius: 4px; margin-right: 5px; cursor: pointer; font-size: 12px; }
        .btn-sair.aviso { background: #ff9100; }
        
        .conteudo-aba { display: none; padding: 15px; max-width: 500px; margin: 0 auto; }
        .conteudo-aba.ativa { display: block; }
        .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); margin-bottom: 15px; }
        
        #camera-caixa { width: 100%; max-width: 400px; height: 250px; margin: 10px auto; border: 2px solid #333; border-radius: 8px; overflow: hidden; background: #000; position: relative; }
        #camera-caixa video { width: 100% !important; height: 100% !important; object-fit: cover; }
        
        input { width: 100%; padding: 12px; font-size: 16px; margin: 8px 0; border: 1px solid #ccc; border-radius: 4px; box-sizing: border-box; }
        button { width: 100%; padding: 12px; font-size: 16px; background: #00c853; color: white; border: none; border-radius: 4px; font-weight: bold; cursor: pointer; }
        .btn-login { background: #1a237e; margin-top: 10px; }
        .btn-finalizar { background: #ff9100; color: white; font-size: 18px; margin-top: 12px; width: 100%; box-shadow: 0 3px 6px rgba(0,0,0,0.2); }
        .btn-finalizar:active { background: #e08100; }
        .lista-produtos { border: 1px solid #e0e0e0; border-radius: 4px; max-height: 180px; overflow-y: auto; background: #fafafa; padding: 10px; margin-top: 10px; }
        .item-produto { display: flex; justify-content: space-between; padding: 8px; border-bottom: 1px solid #eee; background: white; margin-bottom: 5px; }
        .painel-total { background: #1a237e; color: white; padding: 15px; border-radius: 8px; text-align: center; margin-top: 15px; }
        .total-valor { font-size: 28px; font-weight: bold; color: #00c853; }
        .info-licenca { text-align: center; color: #666; font-size: 12px; margin-top: 10px; }
        #status-camera { color: #ff9100; font-size: 14px; text-align: center; margin-top: 5px; font-weight: bold; }
    </style>
</head>
<body>

    <!-- TELA DE LOGIN -->
    <div id="tela-bloqueio">
        <div class="card-login">
            <h2>🔑 Atacadão PDV</h2>
            <p>Insira sua Chave de Acesso.</p>
            <input type="text" id="inputKey" placeholder="Cole sua KEY aqui...">
            <button class="btn-login" onclick="fazerLoginComKey()">Ativar Sistema</button>
            <p id="erroLogin" style="color: red; font-size: 14px; margin-top: 10px; display: none;"></p>
        </div>
    </div>

    <!-- SISTEMA PRINCIPAL -->
    <div id="sistema-principal">
        <div class="topo-sistema">
            <div class="menu-abas">
                <button id="aba-btn-compras" class="aba-botao ativa" onclick="abrirAba('compras')">🛒 Caixa PDV</button>
                <button id="aba-btn-personalizar" class="aba-botao" onclick="abrirAba('personalizar')">⚙️ Cadastrar Preços</button>
            </div>
            <button id="botaoSair" class="btn-sair" onclick="confirmarSaida()">Sair</button>
        </div>

        <!-- ABA 1: CAIXA COMPLETO -->
        <div id="compras" class="conteudo-aba ativa">
            <div class="card">
                <h2>🛒 Caixa - Scanner</h2>
                <div id="camera-caixa"></div>
                <div id="status-camera">Aguardando permissão da câmera...</div>
                <h3>Itens Passados:</h3>
                <div class="lista-produtos" id="espacoLista"></div>
            </div>
            
            <div class="painel-total">
                <div>TOTAL DA COMPRA</div>
                <div class="total-valor">R$ <span id="valorTotal">0,00</span></div>
            </div>
            
            <button class="btn-finalizar" onclick="finalizarCompraAtual()">💸 Finalizar Compra</button>

            <!-- DIGITAR MANUALMENTE EMBAIXO DO FINALIZAR -->
            <div class="card" style="margin-top: 20px;">
                <h3>⌨️ Lançar Código Manual</h3>
                <input type="text" id="codigoManualInput" placeholder="Digite o código de barras">
                <button style="background: #1a237e; margin-top: 5px; padding: 10px;" onclick="lancarCodigoManual()">Lançar Item</button>
            </div>
        </div>

        <!-- ABA 2: CONFIGURAR PREÇOS (SALVA NO CACHÊ) -->
        <div id="personalizar" class="conteudo-aba">
            <div class="card">
                <h2>⚙️ Cadastrar Produto</h2>
                <p style="font-size: 13px; color: #666; margin: 0 0 10px 0;">Os produtos ficam salvos na memória do seu Chrome.</p>
                <input type="text" id="codConfig" placeholder="Código de barras do produto">
                <input type="number" id="precoConfig" placeholder="Preço (R$)" step="0.01">
                <button onclick="salvarPreco()">Salvar Preço no Sistema</button>
            </div>
            <div class="info-licenca">Validade: <span id="dataExpira" style="font-weight: bold; color: #00c853;">--/--/----</span></div>
        </div>
    </div>

<script>
    let cliqueSair = false; let timerSair; let loopTempo; let html5QrCodeInstance = null;
    let precosDoMercado = {}; 
    let totalGeralCompra = 0; let ultimoCodigoLido = ""; let tempoUltimaLeitura = 0;

    function carregarProdutosDoCache() {
        const salvos = localStorage.getItem("banco_produtos_mercado");
        if (salvos) { precosDoMercado = JSON.parse(salvos); }
    }

    function confirmarSaida() {
        const btn = document.getElementById("botaoSair");
        if (!cliqueSair) {
            cliqueSair = true; btn.innerText = "De novo!"; btn.classList.add("aviso");
            timerSair = setTimeout(() => { cliqueSair = false; btn.innerText = "Sair"; btn.classList.remove("aviso"); }, 3000);
        } else {
            clearTimeout(timerSair); clearInterval(loopTempo);
            fecharCamera().then(() => {
                const keyAtual = localStorage.getItem("key_atual");
                try {
                    const decifrado = atob(keyAtual.replace("ATCD-", "")).split("||");
                    if(decifrado[2] === "unico") {
                        let queimadas = JSON.parse(localStorage.getItem("keys_queimadas") || "[]");
                        queimadas.push(decifrado[0]); localStorage.setItem("keys_queimadas", JSON.stringify(queimadas));
                    }
                } catch(e){}
                localStorage.removeItem("key_atual"); window.location.reload();
            });
        }
    }

    window.onload = function() {
        carregarProdutosDoCache();
        const keySalva = localStorage.getItem("key_atual");
        if (keySalva) { rodarSistema(keySalva); }
        document.getElementById("codigoManualInput").addEventListener("keypress", function(event) {
            if (event.key === "Enter") { event.preventDefault(); lancarCodigoManual(); }
        });
    }

    function fazerLoginComKey() {
        const keyDigitada = document.getElementById("inputKey").value.trim();
        const erro = document.getElementById("erroLogin");
        if (!keyDigitada.startsWith("ATCD-")) { erro.innerText = "Chave Inválida!"; erro.style.display = "block"; return; }
        try {
            const decifrado = atob(keyDigitada.replace("ATCD-", "")).split("||");
            let queimadas = JSON.parse(localStorage.getItem("keys_queimadas") || "[]");
            if (queimadas.includes(decifrado[0])) { erro.innerText = "Chave já utilizada!"; erro.style.display = "block"; return; }
            if (decifrado[2] === "tempo" && Date.now() > parseInt(decifrado[3])) { erro.innerText = "Chave expirada!"; erro.style.display = "block"; return; }
            localStorage.setItem("key_atual", keyDigitada); erro.style.display = "none"; rodarSistema(keyDigitada);
        } catch(e) { erro.innerText = "Chave corrompida!"; erro.style.display = "block"; }
    }

    function rodarSistema(key) {
        try {
            const decifrado = atob(key.replace("ATCD-", "")).split("||");
            if (decifrado[2] === "tempo") {
                document.getElementById("dataExpira").innerText = new Date(parseInt(decifrado[3])).toLocaleString('pt-BR');
                loopTempo = setInterval(() => {
                    if (Date.now() > parseInt(decifrado[3])) {
                        alert("Sua licença expirou!"); fecharCamera().then(() => { localStorage.removeItem("key_atual"); window.location.reload(); });
                    }
                }, 1000);
            } else { document.getElementById("dataExpira").innerText = "Uso Único"; }
            document.getElementById("tela-bloqueio").style.display = "none";
            document.getElementById("sistema-principal").style.display = "block";
            setTimeout(inicializarCameraDireta, 300);
        } catch(e) { localStorage.removeItem("key_atual"); window.location.reload(); }
    }

    function inicializarCameraDireta() {
        const statusTxt = document.getElementById("status-camera");
        if (!html5QrCodeInstance) { html5QrCodeInstance = new Html5Qrcode("camera-caixa"); }
        
        Html5Qrcode.getCameras().then(devices => {
            if (devices && devices.length > 0) {
                let cameraId = devices[0].id;
                for (let i = 0; i < devices.length; i++) {
                    const label = devices[i].label.toLowerCase();
                    if (label.includes('back') || label.includes('traseira') || label.includes('environment') || label.includes('0')) { 
                        cameraId = devices[i].id; 
                        break; 
                    }
                }
                html5QrCodeInstance.start(cameraId, { fps: 20, qrbox: { width: 280, height: 150 } }, onScanSuccess, () => {})
                .then(() => { statusTxt.style.display = "none"; })
                .catch(() => { statusTxt.innerText = "Erro ao iniciar. Verifique as permissões."; });
            } else { statusTxt.innerText = "Nenhuma câmera detectada."; }
        }).catch(() => { statusTxt.innerText = "Permissão de câmera negada."; });
    }

    function fecharCamera() {
        return new Promise((resolve) => {
            if (html5QrCodeInstance && html5QrCodeInstance.isScanning) { html5QrCodeInstance.stop().then(() => resolve()).catch(() => resolve()); }
            else { resolve(); }
        });
    }
    
    function abrirAba(idAba) {
        document.querySelectorAll('.conteudo-aba').forEach(aba => aba.classList.remove('ativa'));
        document.querySelectorAll('.aba-botao').forEach(btn => btn.classList.remove('ativa'));
        document.getElementById(idAba).classList.add('ativa');
        document.getElementById('aba-btn-' + idAba).classList.add('ativa');
        
        if (idAba === 'compras') { setTimeout(inicializarCameraDireta, 200); }
        else { fecharCamera(); }
    }
    
    function salvarPreco() {
        const codigo = document.getElementById("codConfig").value.trim();
        const precoInformado = parseFloat(document.getElementById("precoConfig").value);
        if (!codigo || isNaN(precoInformado)) return alert("Preencha tudo!");
        
        precosDoMercado[codigo] = { preco: precoInformado };
        localStorage.setItem("banco_produtos_mercado", JSON.stringify(precosDoMercado));
        
        alert(`Produto ${codigo} Salvo com Sucesso!`);
        document.getElementById("codConfig").value = ""; document.getElementById("precoConfig").value = "";
    }
    
    function lancarCodigoManual() {
        const inputManual = document.getElementById("codigoManualInput");
        const codigo = inputManual.value.trim(); if (!codigo) return;
        processarItemVendido(codigo); inputManual.value = "";
    }

    function onScanSuccess(decodedText) {
        const agora = Date.now(); 
        if (decodedText === ultimoCodigoLido && (agora - tempoUltimaLeitura) < 3000) return;
        ultimoCodigoLido = decodedText; tempoUltimaLeitura = agora; 
        try { window.navigator.vibrate(100); } catch(e){}
        processarItemVendido(decodedText);
    }

    function processarItemVendido(codigo) {
        const lista = document.getElementById("espacoLista");
        const campoTotal = document.getElementById("valorTotal");
        
        if (precosDoMercado[codigo]) {
            const prod = precosDoMercado[codigo];
            const itemHTML = `<div class="item-produto"><span>📌 Item: ${codigo}</span><b>R$ ${prod.preco.toFixed(2).replace('.', ',')}</b></div>`;
            lista.innerHTML += itemHTML; lista.scrollTop = lista.scrollHeight;
            totalGeralCompra += prod.preco; campoTotal.innerText = totalGeralCompra.toFixed(2).replace('.', ',');
        } else { 
            alert(`Código: ${codigo} não cadastrado! Direcionando para cadastro.`); 
            document.getElementById("codConfig").value = codigo; 
            abrirAba('personalizar');
        }
    }

    function finalizarCompraAtual() {
        if (totalGeralCompra === 0) {
            alert("Não há itens no carrinho!");
            return;
        }
        alert(`🛒 Compra Finalizada com Sucesso!\n💰 Total: R$ ${totalGeralCompra.toFixed(2).replace('.', ',')}`);
        totalGeralCompra = 0; ultimoCodigoLido = "";
        document.getElementById("espacoLista").innerHTML = "";
        document.getElementById("valorTotal").innerText = "0,00";
    }
</script>
</body>
</html>
