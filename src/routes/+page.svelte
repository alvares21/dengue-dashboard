<script lang="ts">
  import { onMount } from 'svelte';

  const projectTitle = "Dengue Analysis: Climate & Chemical Factors";

  type Record = { month: string; temp: number; rain: number; cases: number };
  
  let records: Record[] = [
    { month: "Jan", temp: 25, rain: 20, cases: 10 },
    { month: "Feb", temp: 27, rain: 35, cases: 15 },
    { month: "Mar", temp: 30, rain: 50, cases: 28 }
  ];

  let inputMonth = "";
  let inputTemp: number | null = null;
  let inputRain: number | null = null;
  let inputCases: number | null = null;

  let chartCanvas: HTMLCanvasElement;
  let chartInstance: any = null;
  let ChartLib: any;

  function addData() {
    if (inputMonth && inputTemp !== null && inputRain !== null && inputCases !== null) {
      records = [...records, { month: inputMonth, temp: inputTemp, rain: inputRain, cases: inputCases }];
      inputMonth = ""; inputTemp = null; inputRain = null; inputCases = null;
    } else {
      alert("Please fill out all fields before adding.");
    }
  }

  function clearData() {
    records = [];
  }

  $: avgTemp = records.length > 0 ? records.reduce((sum, r) => sum + r.temp, 0) / records.length : 0;
  $: avgRain = records.length > 0 ? records.reduce((sum, r) => sum + r.rain, 0) / records.length : 0;
  $: avgCases = records.length > 0 ? records.reduce((sum, r) => sum + r.cases, 0) / records.length : 0;

  $: riskAnalysis = records.map(r => ({
    ...r,
    isHighRisk: r.temp > avgTemp && r.rain > avgRain
  }));

  $: if (ChartLib && chartCanvas && records) {
    if (chartInstance) chartInstance.destroy();
    
    chartInstance = new ChartLib(chartCanvas, {
      type: 'line',
      data: {
        labels: records.map(r => r.month),
        datasets: [
          {
            label: 'Dengue Cases',
            data: records.map(r => r.cases),
            borderColor: '#f85149',
            backgroundColor: 'rgba(248, 81, 73, 0.1)',
            fill: true,
            tension: 0.4,
            yAxisID: 'y'
          },
          {
            label: 'Temp (°C)',
            data: records.map(r => r.temp),
            type: 'bar',
            backgroundColor: 'rgba(88, 166, 255, 0.8)',
            borderRadius: 4,
            yAxisID: 'y1'
          },
          {
            label: 'Rain (mm)',
            data: records.map(r => r.rain),
            type: 'bar',
            backgroundColor: 'rgba(63, 185, 80, 0.8)',
            borderRadius: 4,
            yAxisID: 'y1'
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        interaction: { mode: 'index', intersect: false },
        plugins: {
          legend: { labels: { color: '#c9d1d9', font: { family: 'system-ui' } } }
        },
        scales: {
          x: { grid: { color: '#30363d' }, ticks: { color: '#8b949e' } },
          y: { 
            type: 'linear', display: true, position: 'left', 
            grid: { color: '#30363d' }, ticks: { color: '#f85149' },
            title: { display: true, text: 'Cases', color: '#8b949e' }
          },
          y1: { 
            type: 'linear', display: true, position: 'right', 
            grid: { drawOnChartArea: false }, ticks: { color: '#8b949e' },
            title: { display: true, text: 'Climate Metrics', color: '#8b949e' }
          }
        }
      }
    });
  }

  onMount(async () => {
    const module = await import('chart.js/auto');
    ChartLib = module.default;
  });
</script>

<main class="dashboard">
  <header class="hero">
    <h1>{projectTitle}</h1>
    <p class="subtitle">An interdisciplinary analysis of environmental catalysts and chemical interventions.</p>
  </header>

  <div class="grid-layout">
    <section class="left-col">
      <div class="panel data-entry">
        <h3>Input Climate Data</h3>
        <div class="input-group">
          <div class="field"><label>Month</label><input type="text" bind:value={inputMonth} placeholder="e.g., Apr"></div>
          <div class="field"><label>Temp (°C)</label><input type="number" bind:value={inputTemp} placeholder="0"></div>
          <div class="field"><label>Rain (mm)</label><input type="number" bind:value={inputRain} placeholder="0"></div>
          <div class="field"><label>Cases</label><input type="number" bind:value={inputCases} placeholder="0"></div>
        </div>
        <div class="actions">
          <button class="btn-primary" on:click={addData}>Append Data</button>
          <button class="btn-danger" on:click={clearData}>Wipe Engine</button>
        </div>
      </div>

      <div class="stats-grid">
        <div class="stat-card">
          <span class="label">Avg Temp</span>
          <span class="value">{avgTemp.toFixed(1)}°</span>
        </div>
        <div class="stat-card">
          <span class="label">Avg Rain</span>
          <span class="value">{avgRain.toFixed(1)}mm</span>
        </div>
        <div class="stat-card">
          <span class="label">Avg Cases</span>
          <span class="value">{avgCases.toFixed(1)}</span>
        </div>
      </div>
    </section>

    <section class="right-col panel chart-container">
      <h3>Live Data Visualization</h3>
      <div class="canvas-wrapper">
        <canvas bind:this={chartCanvas}></canvas>
      </div>
    </section>
  </div>

  <div class="bottom-layout">
    <section class="panel">
      <h3>Risk Prediction Matrix</h3>
      <div class="table-wrapper">
        <table>
          <thead>
            <tr>
              <th>Month</th>
              <th>Temperature</th>
              <th>Rainfall</th>
              <th>Cases</th>
              <th>Risk Assessment</th>
            </tr>
          </thead>
          <tbody>
            {#each riskAnalysis as row}
              <tr class={row.isHighRisk ? 'row-high-risk' : 'row-low-risk'}>
                <td>{row.month}</td>
                <td>{row.temp}°C</td>
                <td>{row.rain} mm</td>
                <td>{row.cases}</td>
                <td class={row.isHighRisk ? 'text-danger' : 'text-safe'}>
                  {row.isHighRisk ? 'Critical Outbreak Risk' : 'Baseline Risk'}
                </td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
    </section>

    <section class="panel chem-panel">
      <h3>Chemical Intervention Vectors</h3>
      <div class="chem-grid">
        <div class="chem-card">
          <h4>Synthetic Repellents</h4>
          <ul>
            <li><strong>DEET:</strong> Blocks olfactory receptors directly.</li>
            <li><strong>Picaridin:</strong> Advanced synthetic, lower dermal toxicity.</li>
          </ul>
        </div>
        <div class="chem-card">
          <h4>Larvicidal Compounds</h4>
          <p><strong>Temephos ($C_{16}H_{20}O_{6}PS_{2}$):</strong> Organophosphate water treatment. Induces nervous system failure in larvae by inhibiting cholinesterase.</p>
        </div>
        <div class="chem-card">
          <h4>Green Alternatives</h4>
          <p>Biochemically derived solutions like Lemon Eucalyptus and engineered biodegradable polymer matrices for slow-release mitigation.</p>
        </div>
      </div>
    </section>
  </div>
</main>

<style>
  :global(body) {
    background-color: #0d1117;
    color: #c9d1d9;
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen-Sans, Ubuntu, Cantarell, sans-serif;
    -webkit-font-smoothing: antialiased;
  }

  .dashboard {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
  }

  h1, h2, h3, h4 { color: #f0f6fc; margin-top: 0; }
  h1 { font-size: 2.5rem; letter-spacing: -0.05em; margin-bottom: 0.5rem; }
  .subtitle { color: #8b949e; font-size: 1.1rem; margin-bottom: 3rem; }

  .panel {
    background: #161b22;
    border: 1px solid #30363d;
    border-radius: 12px;
    padding: 1.5rem;
    box-shadow: 0 4px 20px rgba(0,0,0,0.3);
  }

  .grid-layout {
    display: grid;
    grid-template-columns: 1fr 2fr;
    gap: 1.5rem;
    margin-bottom: 1.5rem;
  }
  
  .bottom-layout {
    display: grid;
    grid-template-columns: 3fr 2fr;
    gap: 1.5rem;
  }

  @media (max-width: 900px) {
    .grid-layout, .bottom-layout { grid-template-columns: 1fr; }
  }

  .input-group {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 1rem;
    margin-bottom: 1.5rem;
  }
  
  .field { display: flex; flex-direction: column; }
  .field label { font-size: 0.8rem; color: #8b949e; margin-bottom: 0.3rem; text-transform: uppercase; letter-spacing: 0.05em; }
  
  input {
    background: #0d1117;
    border: 1px solid #30363d;
    color: #c9d1d9;
    padding: 0.6rem;
    border-radius: 6px;
    font-size: 1rem;
    transition: all 0.2s ease;
  }
  input:focus { outline: none; border-color: #58a6ff; box-shadow: 0 0 0 3px rgba(88, 166, 255, 0.1); }

  .actions { display: flex; gap: 0.75rem; }
  button {
    padding: 0.6rem 1.2rem;
    border-radius: 6px;
    font-weight: 600;
    cursor: pointer;
    border: none;
    transition: all 0.2s ease;
    flex: 1;
  }
  .btn-primary { background: #238636; color: #fff; }
  .btn-primary:hover { background: #2ea043; }
  .btn-danger { background: transparent; color: #f85149; border: 1px solid rgba(248, 81, 73, 0.4); }
  .btn-danger:hover { background: #f85149; color: #fff; }

  .stats-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1rem;
    margin-top: 1.5rem;
  }
  .stat-card {
    background: #161b22;
    border: 1px solid #30363d;
    padding: 1rem;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }
  .stat-card .label { font-size: 0.75rem; color: #8b949e; text-transform: uppercase; margin-bottom: 0.5rem; }
  .stat-card .value { font-size: 1.5rem; font-weight: bold; color: #58a6ff; }

  .chart-container { display: flex; flex-direction: column; }
  .canvas-wrapper { position: relative; height: 300px; width: 100%; flex-grow: 1; }

  .table-wrapper { overflow-x: auto; }
  table { width: 100%; border-collapse: separate; border-spacing: 0 0.5rem; }
  th { text-align: left; padding: 0.75rem 1rem; color: #8b949e; font-size: 0.85rem; text-transform: uppercase; border-bottom: 1px solid #30363d; }
  td { padding: 1rem; background: #0d1117; }
  td:first-child { border-top-left-radius: 6px; border-bottom-left-radius: 6px; }
  td:last-child { border-top-right-radius: 6px; border-bottom-right-radius: 6px; }
  
  .row-high-risk td { border-top: 1px solid rgba(248, 81, 73, 0.2); border-bottom: 1px solid rgba(248, 81, 73, 0.2); }
  .row-high-risk td:first-child { border-left: 3px solid #f85149; }
  .row-low-risk td:first-child { border-left: 3px solid #3fb950; }
  
  .text-danger { color: #f85149; font-weight: bold; }
  .text-safe { color: #3fb950; }

  .chem-grid { display: flex; flex-direction: column; gap: 1rem; }
  .chem-card { background: #0d1117; padding: 1rem; border-radius: 8px; border-left: 3px solid #a371f7; }
  .chem-card h4 { color: #a371f7; margin-bottom: 0.5rem; font-size: 1rem; }
  .chem-card p, .chem-card ul { color: #8b949e; font-size: 0.9rem; line-height: 1.5; margin: 0; }
  .chem-card ul { padding-left: 1.2rem; }
</style>