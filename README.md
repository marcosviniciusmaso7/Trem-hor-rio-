<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Gerador de Reporte - Trem EFC P21/P22</title>
<style>
    body { font-family: Arial, sans-serif; margin: 20px; background: #f4f4f4; }
    h1 { font-size: 1.5em; }
    .field { margin-bottom: 10px; }
    label { font-weight: bold; margin-right: 5px; }
    select, input, textarea { padding: 5px; font-size: 1em; }
    #output { width: 100%; height: 200px; padding: 5px; font-size: 1em; white-space: pre-wrap; font-family: 'Courier New', monospace; }
    button { padding: 8px 12px; font-size: 1em; margin-right: 5px; cursor: pointer; }
</style>
</head>
<body>
<h1>🚆 Gerador de Reporte de Estação - Trem P21 / P22</h1>

<div class="field">
    <label for="prefix">Prefixo:</label>
    <select id="prefix">
        <option value="P21">P21 (São Luís → Parauapebas)</option>
        <option value="P22">P22 (Parauapebas → São Luís)</option>
    </select>
</div>

<div class="field">
    <label for="station">Estação:</label>
    <select id="station"></select>
</div>

<div class="field">
    <label for="date">Data:</label>
    <input type="date" id="date">
</div>

<div class="field">
    <label for="arrival">Posicionado às (chegada real):</label>
    <input type="time" id="arrival">
</div>

<div class="field">
    <label for="departure">Partida às (real):</label>
    <input type="time" id="departure">
</div>

<div class="field">
    <label for="note">Nota (opcional):</label>
    <input type="text" id="note" placeholder="Observações...">
</div>

<div class="field">
    <button id="generate">Gerar Texto</button>
    <button id="copy" disabled>Copiar Texto</button>
</div>

<div class="field">
    <textarea id="output" readonly placeholder="O texto de reporte aparecerá aqui..."></textarea>
</div>

<script>
(function(){
    const schedule = {
        "P21": [
            { name: "Anjo da Guarda", arr: null, dep: "08:00" },
            { name: "Arari", arr: "10:09", dep: "10:16" },
            { name: "Vitória do Mearim", arr: "10:37", dep: "10:40" },
            { name: "Santa Inês", arr: "11:46", dep: "11:56" },
            { name: "Alto Alegre do Pindaré", arr: "12:51", dep: "12:56" },
            { name: "Mineirinho", arr: "13:16", dep: "13:19" },
            { name: "Auzilândia", arr: "13:37", dep: "13:40" },
            { name: "Altamira", arr: "13:59", dep: "14:02" },
            { name: "Vila Pindaré", arr: "14:25", dep: "14:30" },
            { name: "Nova Vida", arr: "15:21", dep: "15:26" },
            { name: "Açailândia", arr: "17:31", dep: "17:41" },
            { name: "São Pedro da Água Branca", arr: "19:54", dep: "19:57" },
            { name: "Marabá", arr: "21:21", dep: "21:31" },
            { name: "Itainópolis", arr: "22:19", dep: "22:22" },
            { name: "Parauapebas", arr: "23:50", dep: null }
        ],
        "P22": [
            { name: "Parauapebas", arr: null, dep: "06:00" },
            { name: "Itainópolis", arr: "07:28", dep: "07:31" },
            { name: "Marabá", arr: "08:19", dep: "08:29" },
            { name: "São Pedro da Água Branca", arr: "09:53", dep: "09:56" },
            { name: "Açailândia", arr: "12:09", dep: "12:19" },
            { name: "Nova Vida", arr: "14:24", dep: "14:29" },
            { name: "Vila Pindaré", arr: "15:20", dep: "15:25" },
            { name: "Altamira", arr: "15:48", dep: "15:51" },
            { name: "Auzilândia", arr: "16:10", dep: "16:13" },
            { name: "Mineirinho", arr: "16:31", dep: "16:34" },
            { name: "Alto Alegre do Pindaré", arr: "16:54", dep: "16:59" },
            { name: "Santa Inês", arr: "17:54", dep: "18:04" },
            { name: "Vitória do Mearim", arr: "19:10", dep: "19:13" },
            { name: "Arari", arr: "19:34", dep: "19:41" },
            { name: "Anjo da Guarda", arr: "22:00", dep: null }
        ]
    };

    const ESTACOES = ["AÇAILÂNDIA", "MARABÁ", "PARAUAPEBAS", "SANTA INÊS", "ANJO DA GUARDA"];

    const prefixSelect = document.getElementById('prefix');
    const stationSelect = document.getElementById('station');
    const dateInput = document.getElementById('date');
    const arrivalInput = document.getElementById('arrival');
    const departureInput = document.getElementById('departure');
    const noteInput = document.getElementById('note');
    const outputArea = document.getElementById('output');
    const copyBtn = document.getElementById('copy');

    prefixSelect.addEventListener('change', populateStations);
    function populateStations() {
        const prefix = prefixSelect.value;
        stationSelect.innerHTML = "";
        schedule[prefix].forEach((station, i) => {
            const opt = document.createElement('option');
            opt.value = i;
            opt.textContent = station.name;
            stationSelect.appendChild(opt);
        });
    }
    populateStations();

    function toMinutes(t) {
        const [h, m] = t.split(':').map(Number);
        return h * 60 + m;
    }

    function formatMinutes(mins) {
        const h = Math.floor(Math.abs(mins) / 60);
        const m = Math.abs(mins) % 60;
        return ${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')};
    }

    function gerarTexto() {
        const prefix = prefixSelect.value;
        const i = parseInt(stationSelect.value);
        const estacao = schedule[prefix][i];
        const data = dateInput.value ? dateInput.value.split('-').reverse().join('/') : '';
        const chegada = arrivalInput.value;
        const partida = departureInput.value;
        const nota = noteInput.value.trim();
        const ehOrigem = (i === 0);
        const ehDestino = (i === schedule[prefix].length - 1);

        const nomeFormatado = estacao.name.toUpperCase();
        let tipo = "";

        if (ESTACOES.includes(nomeFormatado)) {
            tipo = "🚉 ESTAÇÃO DE";
        } else {
            tipo = "🚉 PONTO DE PARADA DE";
        }

        let texto = *🚊 Acompanhamento ${prefix}*\n\n*${tipo} ${nomeFormatado}*\n\nData: ${data}\n;

        if (chegada) texto += Posicionado às: ${chegada}h\n;
        if (!ehDestino && partida) texto += Partimos às: ${partida}h\n;

        if (!ehOrigem && !ehDestino && chegada && partida) {
            const permanencia = toMinutes(partida) - toMinutes(chegada);
            texto += Tempo de Permanência: ${formatMinutes(permanencia)}h\n;
        }

        let atraso = 0;
        if (chegada && estacao.arr) atraso += toMinutes(chegada) - toMinutes(estacao.arr);
        if (partida && estacao.dep) atraso += toMinutes(partida) - toMinutes(estacao.dep);
        if (atraso < -720) atraso += 1440;

        if (chegada || partida) texto += \nTempo de Atraso: ${formatMinutes(atraso)}h\n;

        if (!ehDestino && partida && estacao.dep) {
            const proxima = schedule[prefix][i + 1];
            const tempo = toMinutes(proxima.arr) - toMinutes(estacao.dep);
            const chegadaPrevista = toMinutes(partida) + (tempo < 0 ? tempo + 1440 : tempo);
            const h = Math.floor(chegadaPrevista % 1440 / 60);
            const m = chegadaPrevista % 1440 % 60;
            texto += \nPrevisão em ${proxima.name}: ${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}h\n;
        }

        if (nota) texto += \n*📝 Nota:* ${nota};

        outputArea.value = texto;
        copyBtn.disabled = false;
    }

    document.getElementById('generate').addEventListener('click', gerarTexto);
    copyBtn.addEventListener('click', function(){
        navigator.clipboard.writeText(outputArea.value).then(() => alert("Texto copiado!"));
    });
})();
</script>
</body>
</html>
