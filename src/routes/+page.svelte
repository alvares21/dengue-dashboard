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
            borderColor: '#ff6b6b',
            backgroundColor: 'rgba(255, 107, 107, 0.15)',
            fill: true,
            tension: 0.5,
            borderWidth: 3,
            pointBackgroundColor: '#0d1117',
            pointBorderColor: '#ff6b6b',
            pointBorderWidth: 2,
            pointRadius: 4,
            yAxisID: 'y'
          },
          {
            label: 'Temp (°C)',
            data: records.map(r => r.temp),
            type: 'bar',
            backgroundColor: 'rgba(77, 171, 247, 0.2)',
            borderColor: 'rgba(77, 171, 247, 0.8)',
            borderWidth: 1,
            borderRadius: 8,
            borderSkipped: false,
            yAxisID: 'y1'
          },
          {
            label: 'Rain (mm)',
            data: records.map(r => r.rain),
            type: 'bar',
            backgroundColor: 'rgba(81, 207, 102, 0.2)',
            borderColor: 'rgba(81, 207, 102, 0.8)',
            borderWidth: 1,
            borderRadius: 8,
            borderSkipped: false,
            yAxisID: 'y1'
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        interaction: { mode: 'index', intersect: false },
        plugins: {
          legend: { 
            labels: { color: '#a5b4fc', font: { family: 'Inter, system-ui', size: 13 }, usePointStyle: true }
          },
          tooltip: {
            backgroundColor: 'rgba(15, 23, 42, 0.9)',
            titleColor: '#f1f5f9',
            bodyColor: '#cbd5e1',
            padding: 12,
            cornerRadius: 12,
            borderColor: 'rgba(255,255,255,0.1)',
            borderWidth: 1
          }
        },
        scales: {
          x: { 
            grid: { display: false }, 
            ticks: { color: '#64748b', font: { family: 'Inter, system-ui' } } 
          },
          y: { 
            type: 'linear', display: true, position: 'left', 
            grid: { color: 'rgba(255,255,255,0.03)' }, 
            ticks: { color: '#ff6b6b' }
          },
          y1: { 
            type: 'linear', display: true, position: 'right', 
            grid: { drawOnChartArea: false }, 
            ticks: { color: '#94a3b8' }
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
  <div class="background-glow"></div>
  
  <header class="hero">
    <h1>{projectTitle}</h1>
    <p class="subtitle">An interdisciplinary analysis of environmental catalysts and chemical interventions.</p>
  </header>

  <div class="grid-layout">
    <section class="left-col">
      <div class="glass-panel data-entry">
        <h3>Input Climate Data</h3>
        <div class="input-group">
          <div class="field">
            <input type="text" bind:value={inputMonth} placeholder="Month (e.g. Apr)">
          </div>
          <div class="field">
            <input type="number" bind:value={inputTemp} placeholder="Temp (°C)">
          </div>
          <div class="field">
            <input type="number" bind:value={inputRain} placeholder="Rain (mm)">
          </div>
          <div class="field">
            <input type="number" bind:value={inputCases} placeholder="Cases">
          </div>
        </div>
        <div class="actions">
          <button class="btn-primary" on:click={addData}>Append Data</button>
          <button class="btn-danger" on:click={clearData}>Wipe</button>
        </div>
      </div>

      <div class="stats-grid">
        <div class="stat-card">
          <span class="label">Avg Temp</span>
          <span class="value">{avgTemp.toFixed(1)}°</span>
        </div>
        <div class="stat-card">
          <span class="label">Avg Rain</span>
          <span class="value">{avgRain.toFixed(1)}<span class="unit">mm</span></span>
        </div>
        <div class="stat-card">
          <span class="label">Avg Cases</span>
          <span class="value">{avgCases.toFixed(1)}</span>
        </div>
      </div>
    </section>

    <section class="right-col glass-panel chart-container">
      <h3>Live Visualization</h3>
      <div class="canvas-wrapper">
        <canvas bind:this={chartCanvas}></canvas>
      </div>
    </section>
  </div>

  <div class="bottom-layout">
    <section class="glass-panel">
      <h3>Risk Prediction Matrix</h3>
      <div class="table-wrapper">
        <table>
          <thead>
            <tr>
              <th>Month</th>
              <th>Temperature</th>
              <th>Rainfall</th>
              <th>Cases</th>
              <th>Status</th>
            </tr>
          </thead>
          <tbody>
            {#each riskAnalysis as row}
              <tr class={row.isHighRisk ? 'row-high-risk' : 'row-low-risk'}>
                <td><strong>{row.month}</strong></td>
                <td>{row.temp}°C</td>
                <td>{row.rain} mm</td>
                <td>{row.cases}</td>
                <td>
                  <span class="badge {row.isHighRisk ? 'badge-danger' : 'badge-safe'}">
                    {row.isHighRisk ? 'Critical Risk' : 'Baseline'}
                  </span>
                </td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
    </section>

    <section class="glass-panel chem-panel">
      <h3>Chemical Vectors</h3>
      <div class="chem-grid">
        <div class="chem-card">
          <h4>Synthetic Repellents</h4>
          <p><strong>DEET:</strong> Blocks olfactory receptors.</p>
          <p><strong>Picaridin:</strong> Lower dermal toxicity alternative.</p>
        </div>
        <div class="chem-card">
          <h4>Larvicidal Compounds</h4>
          <p><strong>Temephos:</strong> Organophosphate. Induces nervous system failure by inhibiting cholinesterase.</p>
        </div>
        <div class="chem-card">
          <h4>Green Alternatives</h4>
          <p>Lemon Eucalyptus and biodegradable polymer matrices for slow-release.</p>
        </div>
      </div>
    </section>
  </div>
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;800&display=swap');

  :global(body) {
    background-color: #050505;
    background-image: radial-gradient(circle at 50% 0%, #1a1a2e 0%, #050505 70%);
    color: #f1f5f9;
    margin: 0;
    min-height: 100vh;
    font-family: 'Inter', sans-serif;
    -webkit-font-smoothing: antialiased;
  }

  .dashboard {
    max-width: 1200px;
    margin: 0 auto;
    padding: 3rem 2rem;
    position: relative;
    z-index: 1;
  }

  .background-glow {
    position: absolute;
    top: -200px;
    left: 50%;
    transform: translateX(-50%);
    width: 800px;
    height: 400px;
    background: radial-gradient(ellipse at center, rgba(99, 102, 241, 0.15) 0%, rgba(0,0,0,0) 70%);
    z-index: -1;
    pointer-events: none;
  }

  h1, h2, h3, h4 { 
    margin-top: 0; 
    font-weight: 600;
    letter-spacing: -0.02em;
  }
  
  h1 { 
    font-size: 3rem; 
    font-weight: 800;
    background: linear-gradient(to right, #fff, #94a3b8);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    margin-bottom: 0.5rem; 
  }
  
  .subtitle { 
    color: #64748b; 
    font-size: 1.15rem; 
    font-weight: 300;
    margin-bottom: 3.5rem; 
  }

  h3 {
    color: #e2e8f0;
    font-size: 1.25rem;
    margin-bottom: 1.5rem;
  }

  .glass-panel {
    background: rgba(30, 41, 59, 0.4);
    backdrop-filter: blur(12px);
    -webkit-backdrop-filter: blur(12px);
    border: 1px solid rgba(255, 255, 255, 0.05);
    border-radius: 24px;
    padding: 2rem;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
    transition: transform 0.3s ease, border-color 0.3s ease;
  }
  
  .glass-panel:hover {
    border-color: rgba(255, 255, 255, 0.1);
  }

  .grid-layout {
    display: grid;
    grid-template-columns: 1fr 2.2fr;
    gap: 2rem;
    margin-bottom: 2rem;
  }
  
  .bottom-layout {
    display: grid;
    grid-template-columns: 1.5fr 1fr;
    gap: 2rem;
  }

  @media (max-width: 960px) {
    .grid-layout, .bottom-layout { grid-template-columns: 1fr; }
  }

  .input-group {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
    margin-bottom: 2rem;
  }
  
  input {
    width: 100%;
    box-sizing: border-box;
    background: rgba(15, 23, 42, 0.6);
    border: 1px solid rgba(255, 255, 255, 0.08);
    color: #f8fafc;
    padding: 1rem 1.25rem;
    border-radius: 16px;
    font-size: 0.95rem;
    font-family: inherit;
    transition: all 0.3s ease;
  }
  
  input::placeholder { color: #475569; }
  
  input:focus { 
    outline: none; 
    border-color: #6366f1; 
    background: rgba(15, 23, 42, 0.9);
    box-shadow: 0 0 0 4px rgba(99, 102, 241, 0.1); 
  }

  .actions { display: flex; gap: 1rem; }
  
  button {
    padding: 1rem 1.5rem;
    border-radius: 16px;
    font-weight: 600;
    font-size: 0.95rem;
    cursor: pointer;
    border: none;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    flex: 1;
    font-family: inherit;
  }
  
  .btn-primary { 
    background: linear-gradient(135deg, #6366f1, #4f46e5); 
    color: #fff; 
    box-shadow: 0 10px 20px rgba(79, 70, 229, 0.3);
  }
  
  .btn-primary:hover { 
    transform: translateY(-2px);
    box-shadow: 0 15px 25px rgba(79, 70, 229, 0.4);
  }
  
  .btn-danger { 
    background: rgba(239, 68, 68, 0.1); 
    color: #ef4444; 
    border: 1px solid rgba(239, 68, 68, 0.2); 
    flex: 0.5;
  }
  
  .btn-danger:hover { 
    background: rgba(239, 68, 68, 0.2); 
  }

  .stats-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1rem;
    margin-top: 2rem;
  }
  
  .stat-card {
    background: rgba(15, 23, 42, 0.4);
    border: 1px solid rgba(255, 255, 255, 0.03);
    padding: 1.5rem 1rem;
    border-radius: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    transition: transform 0.3s ease;
  }
  
  .stat-card:hover { transform: translateY(-3px); }
  
  .stat-card .label { 
    font-size: 0.75rem; 
    color: #64748b; 
    text-transform: uppercase; 
    letter-spacing: 0.05em;
    margin-bottom: 0.5rem; 
  }
  
  .stat-card .value { 
    font-size: 1.75rem; 
    font-weight: 800; 
    color: #f8fafc; 
  }
  
  .stat-card .unit {
    font-size: 1rem;
    color: #94a3b8;
    margin-left: 0.15rem;
  }

  .chart-container { display: flex; flex-direction: column; }
  .canvas-wrapper { position: relative; height: 350px; width: 100%; flex-grow: 1; }

.table-wrapper { 
  overflow-x: auto; 
  scrollbar-width: thin;
  scrollbar-color: rgba(148, 163, 184, 0.3) transparent;
}

.table-wrapper::-webkit-scrollbar {
  height: 8px;
}

.table-wrapper::-webkit-scrollbar-track {
  background: transparent;
  border-radius: 8px;
}

.table-wrapper::-webkit-scrollbar-thumb {
  background: rgba(148, 163, 184, 0.3);
  border-radius: 8px;
}

.table-wrapper::-webkit-scrollbar-thumb:hover {
  background: rgba(148, 163, 184, 0.5);
}
  table { width: 100%; border-collapse: separate; border-spacing: 0 0.75rem; }
  th { 
    text-align: left; 
    padding: 0 1.25rem 0.5rem; 
    color: #64748b; 
    font-size: 0.75rem; 
    text-transform: uppercase; 
    letter-spacing: 0.05em;
    font-weight: 600;
  }
  
  tr { transition: transform 0.2s ease; }
  tr:hover { transform: scale(1.01); }
  
 td { 
  padding: 1.25rem; 
  background: rgba(15, 23, 42, 0.4); 
  color: #cbd5e1;
}

td:first-child { border-top-left-radius: 16px; border-bottom-left-radius: 16px; }
td:last-child { border-top-right-radius: 16px; border-bottom-right-radius: 16px; }

.row-high-risk td { 
  background: rgba(239, 68, 68, 0.06); 
}

.row-high-risk td:first-child { 
  border-left: 4px solid rgba(239, 68, 68, 0.8); 
  box-shadow: inset 40px 0 40px -20px rgba(239, 68, 68, 0.15);
}

.row-low-risk td {
  background: rgba(16, 185, 129, 0.03);
}

.row-low-risk td:first-child { 
  border-left: 4px solid rgba(16, 185, 129, 0.6); 
  box-shadow: inset 40px 0 40px -20px rgba(16, 185, 129, 0.1);
}

  .badge {
    padding: 0.4rem 0.8rem;
    border-radius: 20px;
    font-size: 0.75rem;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }
  
  .badge-danger { background: rgba(239, 68, 68, 0.2); color: #fca5a5; }
  .badge-safe { background: rgba(16, 185, 129, 0.2); color: #6ee7b7; }

  .chem-grid { display: flex; flex-direction: column; gap: 1.25rem; }
  
  .chem-card { 
    background: rgba(15, 23, 42, 0.4); 
    padding: 1.5rem; 
    border-radius: 16px; 
    border-left: 4px solid #8b5cf6;
    transition: background 0.3s ease;
  }
  
  .chem-card:hover { background: rgba(15, 23, 42, 0.8); }
  
  .chem-card h4 { color: #c4b5fd; margin-bottom: 0.75rem; font-size: 1.05rem; }
  
  .chem-card p { 
    color: #94a3b8; 
    font-size: 0.9rem; 
    line-height: 1.6; 
    margin: 0 0 0.5rem 0; 
  }
  
  .chem-card p:last-child { margin-bottom: 0; }
  
  strong { color: #e2e8f0; }
</style>