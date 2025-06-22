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
    #output { width: 100%; height: 180px; padding: 5px; font-size: 1em; white-space: pre-wrap; }
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
    var schedule = {
        "P21": [
            { name: "Anjo da Guarda", arr: null, dep: "08:00" },
            { name: "Arari", arr: "10:09", dep: "10:16" },
            { name: "Vitória do Mearim", arr: "10:37", dep: "10:40" },
            { name: "Santa Inês", arr: "11:46", dep: "12:00" },
            { name: "Alto Alegre do Pindaré", arr: "12:51", dep: "12:56" },
            { name: "Mineirinho", arr: "13:16", dep: "13:19" },
            { name: "Auzilândia", arr: "13:37", dep: "13:40" },
            { name: "Altamira", arr: "13:59", dep: "14:02" },
            { name: "Presa de Porco (Vila Pindaré)", arr: "14:15", dep: "14:30" },
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
            { name: "Presa de Porco (Vila Pindaré)", arr: "15:20", dep: "15:25" },
            { name: "Altamira", arr: "15:48", dep: "15:51" },
            { name: "Auzilândia", arr: "16:10", dep: "16:13" },
            { name: "Mineirinho", arr: "16:31", dep: "16:34" },
            { name: "Alto Alegre do Pindaré", arr: "16:54", dep: "16:59" },
            { name: "Santa Inês", arr: "17:56", dep: "18:03" },
            { name: "Vitória do Mearim", arr: "19:07", dep: "19:10" },
            { name: "Arari", arr: "19:34", dep: "19:41" },
            { name: "Anjo da Guarda", arr: "22:00", dep: null }
        ]
    };

    var prefixSelect = document.getElementById('prefix');
    var stationSelect = document.getElementById('station');
    var dateInput = document.getElementById('date');
    var arrivalInput = document.getElementById('arrival');
    var departureInput = document.getElementById('departure');
    var noteInput = document.getElementById('note');
    var outputArea = document.getElementById('output');
    var copyBtn = document.getElementById('copy');
    var generateBtn = document.getElementById('generate');

    function populateStations() {
        var prefix = prefixSelect.value;
        stationSelect.innerHTML = "";
        schedule[prefix].forEach(function(station, index) {
            var opt = document.createElement('option');
            opt.value = index;
            opt.textContent = station.name;
            stationSelect.appendChild(opt);
        });
    }
    prefixSelect.addEventListener('change', populateStations);
    populateStations();

    function formatMinutes(mins) {
        var absMin = Math.abs(mins);
        var h = Math.floor(absMin / 60);
        var m = absMin % 60;
        return (h < 10 ? '0' + h : h) + ':' + (m < 10 ? '0' + m : m);
    }

    generateBtn.addEventListener('click', function() {
        var prefix = prefixSelect.value;
        var stationIndex = parseInt(stationSelect.value);
        if (isNaN(stationIndex)) return;
        var stationData = schedule[prefix][stationIndex];
        var stationName = stationData.name;
        var dateVal = dateInput.value;
        var arrVal = arrivalInput.value;
        var depVal = departureInput.value;
        var noteVal = noteInput.value.trim();

        if (!dateVal || !depVal) {
            alert("Preencha a data e a hora de partida.");
            return;
        }
        if (stationIndex !== 0 && !arrVal) {
            alert("Preencha o horário de chegada.");
            return;
        }

        var dateParts = dateVal.split('-');
        var formattedDate = (dateParts.length === 3) 
            ? (dateParts[2] + '/' + dateParts[1] + '/' + dateParts[0]) 
            : dateVal;

        var actualArrMin = arrVal ? (parseInt(arrVal.split(':')[0]) * 60 + parseInt(arrVal.split(':')[1])) : null;
        var actualDepMin = parseInt(depVal.split(':')[0]) * 60 + parseInt(depVal.split(':')[1]);
        var scheduledArrMin = stationData.arr ? (parseInt(stationData.arr.split(':')[0]) * 60 + parseInt(stationData.arr.split(':')[1])) : null;
        var scheduledDepMin = stationData.dep ? (parseInt(stationData.dep.split(':')[0]) * 60 + parseInt(stationData.dep.split(':')[1])) : null;

        var reportLines = [];
        reportLines.push("🚊 Acompanhamento " + prefix);
        reportLines.push("🚉 ESTAÇÃO DE " + stationName.toUpperCase());
        reportLines.push("Data: " + formattedDate);
        if (arrVal) reportLines.push("Posicionado às: " + arrVal + 'h');
        reportLines.push("Partimos às: " + depVal + 'h');

        if (stationIndex !== 0 && actualArrMin != null) {
            var dwellMin = actualDepMin - actualArrMin;
            if (dwellMin < 0) dwellMin += 1440;
            reportLines.push("Tempo de Permanência: " + formatMinutes(dwellMin) + 'h');
        }

        if (stationIndex !== 0 && scheduledArrMin != null && actualArrMin != null) {
            var arrivalDelayMin = actualArrMin - scheduledArrMin;
            var delayStr = formatMinutes(arrivalDelayMin) + 'h';
            if (arrivalDelayMin > 0) {
                reportLines.push("Tempo de Atraso: " + delayStr);
            }
        }

        if (stationIndex < schedule[prefix].length - 1) {
            var nextStation = schedule[prefix][stationIndex + 1];
            var nextArrSched = nextStation.arr;
            var currDepSched = stationData.dep;
            if (nextArrSched && currDepSched) {
                var nextArrMin = parseInt(nextArrSched.split(':')[0]) * 60 + parseInt(nextArrSched.split(':')[1]);
                var currDepMin = parseInt(currDepSched.split(':')[0]) * 60 + parseInt(currDepSched.split(':')[1]);
                var travelMin = nextArrMin - currDepMin;
                if (travelMin < 0) travelMin += 1440;
                var predictedMin = actualDepMin + travelMin;
                if (predictedMin >= 1440) predictedMin -= 1440;
                var h = Math.floor(predictedMin / 60);
                var m = predictedMin % 60;
                var predTimeStr = (h < 10 ? '0' + h : h) + ':' + (m < 10 ? '0' + m : m);
                reportLines.push("Previsão em " + nextStation.name + ": " + predTimeStr + 'h');
            }
        }

        if (noteVal) reportLines.push("📝 " + noteVal);

        outputArea.value = reportLines.join("\n");
        copyBtn.disabled = false;
    });

    copyBtn.addEventListener('click', function() {
        var text = outputArea.value;
        if (!text) return;
        outputArea.select();
        document.execCommand('copy');
        alert("Texto copiado para a área de transferência!");
    });
})();
</script>
</body>
</html>