<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Gerador de Reporte - Trem EFC P21/P22</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f4f4f4; padding: 20px; }
    h1 { font-size: 1.5em; }
    .field { margin-bottom: 12px; }
    label { font-weight: bold; display: inline-block; width: 180px; }
    select, input, textarea { font-size: 1em; padding: 5px; width: auto; }
    textarea { width: 100%; height: 220px; white-space: pre-wrap; }
    button { font-size: 1em; padding: 8px 12px; margin-right: 10px; }
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
  <label for="arrival">Posicionado às:</label>
  <input type="time" id="arrival">
</div>

<div class="field">
  <label for="departure">Partimos às:</label>
  <input type="time" id="departure">
</div>

<div class="field">
  <label for="note">Nota (opcional):</label>
  <input type="text" id="note" placeholder="Ex: Aguardamos cumprimento de horário.">
</div>

<button id="generate">Gerar Texto</button>
<button onclick="copyText()">📋 Copiar Texto</button>

<textarea id="output" readonly></textarea>

<script>
const schedule = {
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

function parseTime(str) {
  const [h, m] = str.split(":").map(Number);
  return h * 60 + m;
}

function formatTime(mins) {
  const h = String(Math.floor(mins / 60)).padStart(2, '0');
  const m = String(mins % 60).padStart(2, '0');
  return `${h}:${m}`;
}

function updateStations() {
  const prefix = document.getElementById("prefix").value;
  const stationList = document.getElementById("station");
  stationList.innerHTML = "";
  schedule[prefix].forEach((station, idx) => {
    const opt = document.createElement("option");
    opt.value = idx;
    opt.textContent = station.name;
    stationList.appendChild(opt);
  });
}
document.getElementById("prefix").addEventListener("change", updateStations);
updateStations();

document.getElementById("generate").addEventListener("click", function () {
  const prefix = document.getElementById("prefix").value;
  const stationIndex = Number(document.getElementById("station").value);
  const station = schedule[prefix][stationIndex];
  const arrival = document.getElementById("arrival").value;
  const departure = document.getElementById("departure").value;
  const date = document.getElementById("date").value.split("-").reverse().join("/");
  const note = document.getElementById("note").value;

  const lines = [];
  lines.push(`*🚊 Acompanhamento ${prefix}*\n`);
  lines.push(`*🚉 ${station.name.toUpperCase()}*\n`);
  lines.push(`Data : ${date}\n`);

  if (arrival) lines.push(`Posicionado às: ${arrival}h`);
  if (departure) lines.push(`Partimos às: ${departure}h`);

  if (arrival && departure) {
    const diff = (parseTime(departure) - parseTime(arrival) + 1440) % 1440;
    lines.push(`Tempo de Permanência: ${formatTime(diff)}h \n`);
  }

  // Atraso na chegada (exceto estações de origem)
  if (arrival && station.arr) {
    const delay = parseTime(arrival) - parseTime(station.arr);
    if (delay > 0) lines.push(`Tempo de Atraso: ${formatTime(delay)}h\n`);
  }

  // Atraso na partida nas origens
  if (!station.arr && station.dep && departure) {
    const delay = parseTime(departure) - parseTime(station.dep);
    if (delay > 0) lines.push(`Partida atrasada em: ${formatTime(delay)}h\n`);
  }

  // Previsão próxima estação
  const nextStation = schedule[prefix][stationIndex + 1];
  if (nextStation && station.dep && nextStation.arr && departure) {
    const baseTravel = parseTime(nextStation.arr) - parseTime(station.dep);
    const predicted = (parseTime(departure) + baseTravel + 1440) % 1440;
    lines.push(`Previsão em ${nextStation.name}:  ${formatTime(predicted)}h\n`);
  }

  if (note) lines.push(`\n*📝 Nota:* ${note}`);

  document.getElementById("output").value = lines.join("\n");
});

function copyText() {
  const text = document.getElementById("output");
  text.select();
  document.execCommand("copy");
  alert("Texto copiado para a área de transferência!");
}
</script>

</body>
</html>