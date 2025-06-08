# async function ladeDaten() {
    const leitzins = await fetchLeitzins();
    const euribor = await fetchEURIBOR();
    const inflation = await fetchInflation();
    const umlaufrendite = 2.1;
    const bauzins = 3.5;
    const immopreis = 108;
    const zinsstruktur = -0.2;

    const prognose = 0.25 * leitzins + 0.2 * euribor + 0.15 * inflation + 0.15 * umlaufrendite + 0.1 * bauzins + 0.05 * ((immopreis - 100) / 100) + 0.1 * zinsstruktur;

    document.getElementById("werte").innerHTML = `
        <h4>📊 Live-Werte:</h4>
        EZB-Leitzins: ${leitzins.toFixed(2)}%<br>
        EURIBOR: ${euribor.toFixed(2)}%<br>
        Inflation: ${inflation.toFixed(2)}%<br>
        Umlaufrendite: ${umlaufrendite}%<br>
        Immobilienindex: ${immopreis}<br>
        Zinsstruktur-Spread: ${zinsstruktur}%<br>
    `;

    document.getElementById("prognose").innerText = `📉 Erwarteter Bauzins:
    • in 7 Tagen: ${(bauzins + (prognose - bauzins) * 0.1).toFixed(2)} %
    • in 2 Wochen: ${(bauzins + (prognose - bauzins) * 0.2).toFixed(2)} %
    • in 3 Monaten: ${prognose.toFixed(2)} %
    • in 1 Jahr: ${(bauzins + (prognose - bauzins) * 1.5).toFixed(2)} %`;

    const ctx = document.getElementById('zinsChart').getContext('2d');
    new Chart(ctx, {
        type: 'line',
        data: {
            labels: ['Heute', '+7 Tage', '+2 Wochen', '+3 Monate', '+1 Jahr'],
            datasets: [{
                label: 'Bauzins-Prognose (%)',
                data: [
                    bauzins,
                    (bauzins + (prognose - bauzins) * 0.1),
                    (bauzins + (prognose - bauzins) * 0.2),
                    prognose,
                    (bauzins + (prognose - bauzins) * 1.5)
                ],
                borderColor: 'blue',
                fill: false,
                tension: 0.2
            }]
        }
    });
}

async function fetchEURIBOR() {
    const response = await fetch("https://api.api-ninjas.com/v1/euribor", {
        headers: { "X-Api-Key": "DEIN_API_KEY" }
    });
    const data = await response.json();
    return parseFloat(data.find(e => e.name.includes("3 months")).rate_pct);
}

async function fetchInflation() {
    const response = await fetch("https://api.api-ninjas.com/v1/inflationhistorical?country=Germany", {
        headers: { "X-Api-Key": "DEIN_API_KEY" }
    });
    const data = await response.json();
    return parseFloat(data[data.length - 1].value);
}

async function fetchLeitzins() {
    const url = "https://data-api.ecb.europa.eu/service/data/FM/M.U2.EUR.4F.KR.MRR_FR.LEV?format=jsondata";
    const response = await fetch(url);
    const json = await response.json();
    const obs = json.dataSets[0].series["0:0:0:0:0"].observations;
    const lastKey = Object.keys(obs).pop();
    return obs[lastKey][0];
}
