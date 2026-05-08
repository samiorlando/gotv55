# gotv55   <!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <meta name="theme-color" content="#0a0b0f">
  <title>NEXUS BROADCAST ENGINE | Pro Audio Processor</title>
  <style>
    :root {
      --bg-dark: #0a0b0f;
      --bg-panel: rgba(20, 22, 30, 0.85);
      --neon-green: #00ff9d;
      --neon-blue: #00d2ff;
      --neon-purple: #b026ff;
      --text-main: #e0e6f0;
      --text-muted: #8892b0;
      --glass: rgba(255, 255, 255, 0.05);
      --border: 1px solid rgba(255, 255, 255, 0.1);
      --radius: 12px;
      --danger: #ff4757;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Inter', system-ui, -apple-system, sans-serif; }
    body {
      background: radial-gradient(circle at 50% 0%, #1a1d2a 0%, var(--bg-dark) 70%);
      color: var(--text-main);
      min-height: 100vh;
      overflow-x: hidden;
    }

    #app-wrapper { display: flex; flex-direction: column; height: 100vh; }
    .header { display: flex; justify-content: space-between; align-items: center; padding: 1rem 1.5rem; border-bottom: var(--border); background: rgba(10,11,15,0.8); backdrop-filter: blur(10px); flex-shrink: 0; }
    .logo { font-size: 1.5rem; font-weight: 800; background: linear-gradient(90deg, var(--neon-green), var(--neon-blue)); -webkit-background-clip: text; background-clip: text; color: transparent; }
    .status-panel span { margin-left: 1rem; font-family: 'JetBrains Mono', monospace; font-size: 0.85rem; color: var(--text-muted); }
    #stream-status.active { color: var(--danger); animation: pulse 1.5s infinite; }
    @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:0.4} }

    .main-container { display: grid; grid-template-columns: 280px 1fr; gap: 1.5rem; padding: 1.5rem; max-width: 1800px; margin: 0 auto; width: 100%; overflow-y: auto; }
    @media (max-width: 1024px) { .main-container { grid-template-columns: 1fr; } }

    /* I/O ROUTING PANEL */
    .connection-list {
      background: rgba(15, 17, 23, 0.95);
      border: 1px solid rgba(0, 210, 255, 0.15);
      border-radius: 10px;
      padding: 14px;
      display: flex;
      flex-direction: column;
      gap: 12px;
      height: fit-content;
    }
    .io-section { background: rgba(255,255,255,0.03); border-radius: 8px; padding: 10px; border: 1px solid rgba(255,255,255,0.05); }
    .io-header { font-size: 11px; font-weight: 700; letter-spacing: 1px; color: #fff; margin-bottom: 8px; display: flex; align-items: center; gap: 6px; }
    .io-dot { width: 8px; height: 8px; border-radius: 50%; box-shadow: 0 0 6px currentColor; }
    .io-select { width: 100%; background: #111; color: #e0e0e0; border: 1px solid #444; padding: 6px; border-radius: 5px; font-size: 11px; cursor: pointer; }
    .io-meters-row { display: flex; gap: 10px; align-items: center; margin: 8px 0; }
    .vu-container { display: flex; gap: 4px; flex: 1; }
    .vu-track { flex: 1; height: 70px; background: #0a0a0a; border-radius: 3px; position: relative; overflow: hidden; border: 1px solid #222; }
    .vu-bar { position: absolute; bottom: 0; width: 100%; height: 0%; background: linear-gradient(to top, var(--danger), #ffa502, var(--neon-green)); transition: height 0.04s linear; }
    .vu-labels { font-size: 9px; color: #666; text-align: center; width: 30px; }
    .peak-display { display: flex; flex-direction: column; gap: 4px; font-family: monospace; font-size: 10px; color: var(--neon-blue); background: #0a0a0a; padding: 6px; border-radius: 5px; border: 1px solid #222; min-width: 70px; }
    .fft-visualizer { width: 100%; height: 50px; background: #050608; border-radius: 5px; margin: 6px 0; border: 1px solid #222; display: block; }
    .io-controls-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin: 8px 0; }
    .io-control { display: flex; flex-direction: column; gap: 4px; font-size: 10px; color: #888; }
    .io-control input { width: 100%; height: 4px; background: #222; border-radius: 2px; outline: none; -webkit-appearance: none; }
    .io-control input::-webkit-slider-thumb { -webkit-appearance: none; width: 12px; height: 12px; background: var(--neon-blue); border-radius: 50%; cursor: pointer; }
    .io-val { color: var(--neon-green); font-weight: bold; text-align: right; }
    .io-toggles-row { display: flex; gap: 6px; align-items: center; margin-top: 6px; flex-wrap: wrap; }
    .io-btn { flex: 1; min-width: 60px; padding: 6px; background: #1a1a1a; border: 1px solid #333; color: #666; border-radius: 4px; cursor: pointer; font-size: 10px; transition: all 0.2s; }
    .io-btn.active { background: rgba(0,210,255,0.15); border-color: var(--neon-blue); color: var(--neon-blue); }
    .mute-btn.active { background: rgba(255,71,87,0.15); border-color: var(--danger); color: var(--danger); }
    .io-status-badge { font-size: 9px; padding: 4px 8px; background: rgba(0,255,157,0.15); color: var(--neon-green); border-radius: 3px; font-weight: bold; }
    .clip-indicator { font-size: 9px; padding: 4px 8px; background: #333; color: #666; border-radius: 3px; font-weight: bold; transition: all 0.1s; }
    .clip-indicator.active { background: var(--danger); color: #fff; box-shadow: 0 0 10px var(--danger); animation: clipFlash 0.15s infinite; }
    @keyframes clipFlash { 0%,100%{opacity:1} 50%{opacity:0.4} }
    .io-divider { height: 1px; background: linear-gradient(90deg, transparent, #444, transparent); margin: 4px 0; }

    /* PROCESSOR PANEL */
    .processor-content { display: flex; flex-direction: column; gap: 1rem; }
    .preset-selector { background: linear-gradient(90deg, #333, #2a2a2a); border: 1px solid #444; border-radius: 10px; padding: 15px; }
    .preset-title { font-size: 11px; color: #888; margin-bottom: 10px; text-transform: uppercase; letter-spacing: 1px; }
    .preset-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 10px; }
    .preset-btn { background: linear-gradient(135deg, #2a2a2a, #1a1a1a); border: 1px solid #444; border-radius: 6px; padding: 12px; cursor: pointer; transition: all 0.3s; text-align: left; position: relative; overflow: hidden; }
    .preset-btn::before { content: ''; position: absolute; top: 0; left: 0; width: 3px; height: 100%; background: linear-gradient(to bottom, var(--neon-blue), var(--neon-green)); opacity: 0; transition: opacity 0.3s; }
    .preset-btn:hover { background: linear-gradient(135deg, #3a3a3a, #2a2a2a); transform: translateY(-2px); box-shadow: 0 5px 15px rgba(0,0,0,0.3); }
    .preset-btn.active { background: linear-gradient(135deg, rgba(0,210,255,0.15), rgba(0,255,157,0.15)); border-color: var(--neon-blue); box-shadow: 0 0 20px rgba(0,210,255,0.2); }
    .preset-btn.active::before { opacity: 1; }
    .preset-name { font-size: 12px; font-weight: bold; color: var(--neon-green); margin-bottom: 3px; }
    .preset-desc { font-size: 10px; color: #666; }
    .preset-badge { position: absolute; top: 5px; right: 5px; background: linear-gradient(135deg, var(--neon-purple), var(--neon-blue)); color: white; font-size: 8px; padding: 2px 6px; border-radius: 3px; font-weight: bold; }
    .preset-controls { display: flex; gap: 10px; margin-top: 12px; align-items: center; }
    .preset-select { flex: 1; background: #1a1a1a; border: 1px solid #444; color: #e0e0e0; padding: 8px; border-radius: 5px; font-size: 12px; cursor: pointer; }
    .btn-small { padding: 8px 15px; background: linear-gradient(135deg, var(--neon-blue), var(--neon-green)); border: none; border-radius: 5px; color: #000; font-size: 11px; font-weight: bold; cursor: pointer; transition: all 0.2s; }
    .btn-small:hover { transform: translateY(-1px); box-shadow: 0 3px 10px rgba(0,210,255,0.4); }
    .btn-small.secondary { background: #444; color: #fff; }

    .preset-bar { background: #333; padding: 10px; border-radius: 8px; display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 10px; }
    .preset-info { font-size: 13px; color: #aaa; }
    .preset-name-display { color: var(--neon-green); font-weight: bold; }

    .tabs { display: flex; gap: 5px; margin-bottom: 15px; overflow-x: auto; flex-wrap: wrap; padding-bottom: 5px; }
    .tab { padding: 8px 15px; background: #333; border: 1px solid #444; border-radius: 6px; cursor: pointer; font-size: 11px; white-space: nowrap; transition: all 0.2s; }
    .tab.active { background: var(--neon-blue); color: #000; border-color: var(--neon-blue); font-weight: bold; }
    .tab:hover:not(.active) { background: #3a3a3a; }

    .tab-panels { position: relative; }
    .tab-panel { display: none; animation: fadeIn 0.3s ease; }
    .tab-panel.active { display: block; }
    @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

    .controls-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 15px; }
    .control-panel { background: #333; border: 1px solid #444; border-radius: 8px; padding: 15px; }
    .panel-title { font-size: 12px; color: var(--neon-blue); margin-bottom: 15px; text-transform: uppercase; border-bottom: 1px solid #444; padding-bottom: 5px; }
    .control-group { margin: 12px 0; }
    .control-label { font-size: 11px; color: #888; margin-bottom: 5px; display: flex; justify-content: space-between; }
    .control-value { color: var(--neon-green); font-weight: bold; }
    input[type="range"] { width: 100%; height: 6px; background: #1a1a1a; border-radius: 3px; outline: none; -webkit-appearance: none; }
    input[type="range"]::-webkit-slider-thumb { -webkit-appearance: none; width: 16px; height: 16px; background: linear-gradient(135deg, var(--neon-purple), var(--neon-blue)); border-radius: 50%; cursor: pointer; box-shadow: 0 0 10px rgba(0,210,255,0.5); }
    .toggle-group { display: flex; gap: 10px; margin-top: 10px; }
    .toggle-btn { flex: 1; padding: 8px; background: #2a2a2a; border: 1px solid #444; color: #888; border-radius: 4px; cursor: pointer; font-size: 11px; transition: all 0.2s; }
    .toggle-btn.active { background: var(--neon-blue); color: #000; border-color: var(--neon-blue); }

    .meters-container { display: grid; grid-template-columns: repeat(10, 1fr); gap: 10px; margin-bottom: 20px; background: #1a1a1a; padding: 15px; border-radius: 8px; }
    .meter-group { display: flex; flex-direction: column; align-items: center; }
    .meter-label { font-size: 10px; color: #888; margin-bottom: 5px; text-align: center; }
    .meter { width: 36px; height: 120px; background: #0a0a0a; border: 1px solid #333; border-radius: 4px; position: relative; overflow: hidden; }
    .meter-bar { position: absolute; bottom: 0; width: 100%; background: linear-gradient(to top, #ff9500 0%, #ffeb3b 60%, var(--neon-green) 100%); transition: height 0.05s linear; }
    .meter-scale { position: absolute; right: 2px; top: 0; bottom: 0; display: flex; flex-direction: column; justify-content: space-between; padding: 2px; font-size: 8px; color: #666; }

    .audio-controls { margin-top: 20px; display: flex; gap: 10px; justify-content: center; flex-wrap: wrap; }
    .btn { padding: 12px 30px; background: linear-gradient(135deg, var(--neon-blue), var(--neon-green)); border: none; border-radius: 6px; color: #000; font-weight: bold; cursor: pointer; transition: transform 0.1s, box-shadow 0.2s; }
    .btn:hover { transform: translateY(-2px); box-shadow: 0 5px 20px rgba(0,210,255,0.4); }
    .btn:active { transform: translateY(0); }
    .btn-secondary { background: #444; color: #fff; }
    .btn.active { background: var(--danger); color: white; }

    .branding { text-align: right; padding: 10px; color: #666; font-size: 11px; margin-top: auto; }
    .brand-name { font-size: 24px; font-weight: bold; background: linear-gradient(90deg, var(--neon-blue), var(--neon-green)); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
  </style>
</head>
<body>

  <div id="app-wrapper">
    <header class="header">
      <div class="logo">⚡ NEXUS BROADCAST</div>
      <div class="status-panel">
        <span id="latency-indicator">LAT: --ms</span>
        <span id="cpu-meter">CPU: --%</span>
        <span id="stream-status">OFFLINE</span>
      </div>
    </header>

    <div class="main-container">
      <!-- I/O ROUTING PANEL -->
      <div class="connection-list">
        <div class="io-section">
          <div class="io-header"><span class="io-dot" style="background:var(--neon-blue);"></span>INPUT</div>
          <div class="io-device-selector">
            <select id="input-device-select" class="io-select"><option value="">Cargando dispositivos...</option></select>
          </div>
          <div class="io-meters-row">
            <div class="vu-container">
              <div class="vu-track"><div class="vu-bar" id="in-vu-l"></div></div>
              <div class="vu-track"><div class="vu-bar" id="in-vu-r"></div></div>
              <div class="vu-labels">L / R</div>
            </div>
            <div class="peak-display" id="in-peak">-∞ dB</div>
          </div>
          <div class="io-controls-grid">
            <label class="io-control">
              <span>Input Gain</span>
              <input type="range" id="input-gain" min="-20" max="20" step="0.5" value="0">
              <span class="io-val" id="val-input-gain">0 dB</span>
            </label>
            <label class="io-control">
              <span>Mic Level</span>
              <input type="range" id="mic-level" min="0" max="100" step="1" value="80">
              <span class="io-val" id="val-mic-level">80%</span>
            </label>
          </div>
          <div class="io-toggles-row">
            <button class="io-btn" id="btn-input-mono">🔀 MONO</button>
            <button class="io-btn active" id="btn-noise-reduction">🔇 NR ON</button>
            <button class="io-btn mute-btn" id="btn-input-mute">🔇 MUTE</button>
            <div class="io-status-badge" id="input-mode-indicator">STEREO</div>
          </div>
        </div>

        <div class="io-divider"></div>

        <div class="io-section">
          <div class="io-header"><span class="io-dot" style="background:var(--neon-green);"></span>OUTPUT</div>
          <div class="io-device-selector">
            <select id="output-device-select" class="io-select"><option value="">Cargando dispositivos...</option></select>
          </div>
          <div class="io-meters-row">
            <div class="vu-container">
              <div class="vu-track"><div class="vu-bar" id="out-vu-l"></div></div>
              <div class="vu-track"><div class="vu-bar" id="out-vu-r"></div></div>
              <div class="vu-labels">L / R</div>
            </div>
            <div class="peak-display">
              <div id="out-rms">RMS: -</div>
              <div id="out-peak">PEAK: -</div>
            </div>
          </div>
          <canvas id="fft-canvas" class="fft-visualizer"></canvas>
          <div class="io-controls-grid">
            <label class="io-control">
              <span>Master Volume</span>
              <input type="range" id="master-volume" min="-30" max="6" step="0.1" value="0">
              <span class="io-val" id="val-master-vol">0 dB</span>
            </label>
            <label class="io-control">
              <span>L/R Balance</span>
              <input type="range" id="output-balance" min="-100" max="100" step="1" value="0">
              <span class="io-val" id="val-balance">C</span>
            </label>
          </div>
          <div class="io-toggles-row">
            <button class="io-btn active" id="btn-output-limiter"> LIM ON</button>
            <button class="io-btn" id="btn-headphone-monitor"> MON</button>
            <div class="clip-indicator" id="output-clip">️ CLIP</div>
          </div>
        </div>
      </div>

      <!-- PROCESSOR PANEL -->
      <div class="processor-content">
        <div class="preset-selector">
          <div class="preset-title">️ Premium Presets Library</div>
          <div class="preset-grid">
            <div class="preset-btn active" data-preset="broadcast-hd"><span class="preset-badge">PRO</span><div class="preset-name">Broadcast HD</div><div class="preset-desc">Radio profesional -14 LUFS</div></div>
            <div class="preset-btn" data-preset="voice-over"><span class="preset-badge">PRO</span><div class="preset-name">Voice Over Pro</div><div class="preset-desc">Voz clara y presente</div></div>
            <div class="preset-btn" data-preset="music-master"><span class="preset-badge">PRO</span><div class="preset-name">Music Master</div><div class="preset-desc">Música con punch</div></div>
            <div class="preset-btn" data-preset="podcast-studio"><span class="preset-badge">PRO</span><div class="preset-name">Podcast Studio</div><div class="preset-desc">Conversación natural</div></div>
            <div class="preset-btn" data-preset="radio-fm"><span class="preset-badge">PRO</span><div class="preset-name">Radio FM</div><div class="preset-desc">Sonido clásico FM</div></div>
            <div class="preset-btn" data-preset="streaming-live"><span class="preset-badge">PRO</span><div class="preset-name">Streaming Live</div><div class="preset-desc">Twitch/YouTube Live</div></div>
            <div class="preset-btn" data-preset="audiobook"><span class="preset-badge">PRO</span><div class="preset-name">Audiobook</div><div class="preset-desc">Narración larga</div></div>
            <div class="preset-btn" data-preset="club-dj"><span class="preset-badge">PRO</span><div class="preset-name">Club/DJ</div><div class="preset-desc">Bass pesado -9 LUFS</div></div>
          </div>
          <div class="preset-controls">
            <select class="preset-select" id="custom-preset-select"><option value="">Cargar preset personalizado...</option></select>
            <button class="btn-small" id="btn-save-preset">💾 Guardar</button>
            <button class="btn-small secondary" id="btn-del-preset">️ Borrar</button>
          </div>
        </div>

        <div class="preset-bar">
          <div class="preset-info">Active Preset: <span class="preset-name-display" id="active-preset-name">Broadcast HD</span></div>
          <div class="preset-info">Mode: <span style="color: var(--neon-green);" id="display-mode">Music</span> | Core: <span style="color: var(--neon-green);">1</span> | Target Loudness: <span style="color: var(--neon-green);" id="display-lufs">-14 LUFS</span></div>
        </div>

        <div class="tabs">
          <div class="tab active" data-tab="tab-lessmore">2.0 Less-More</div>
          <div class="tab" data-tab="tab-agc">2.0 AGC</div>
          <div class="tab" data-tab="tab-eq">2.0 EQ</div>
          <div class="tab" data-tab="tab-stereo">Stereo Enhancer</div>
          <div class="tab" data-tab="tab-multiband">2.0 Multiband</div>
          <div class="tab" data-tab="tab-compressors">2.0 Compressors</div>
          <div class="tab" data-tab="tab-speech">2.0 Speech Mode</div>
          <div class="tab" data-tab="tab-bandmix">2.0 Bandmix</div>
          <div class="tab" data-tab="tab-limiters">2.0 Limiters</div>
          <div class="tab" data-tab="tab-synthesizer">Stereo Synthesizer</div>
        </div>

        <div class="meters-container">
          <div class="meter-group"><div class="meter-label">Input</div><div class="meter"><div class="meter-bar" id="meter-input"></div><div class="meter-scale"><span>0</span><span>-6</span><span>-12</span><span>-18</span><span>-24</span><span>-36</span></div></div></div>
          <div class="meter-group"><div class="meter-label">2.0 AGC</div><div class="meter"><div class="meter-bar" id="meter-agc"></div></div></div>
          <div class="meter-group"><div class="meter-label">HF Enhancer</div><div class="meter"><div class="meter-bar" id="meter-hf"></div></div></div>
          <div class="meter-group"><div class="meter-label">Stereo</div><div class="meter"><div class="meter-bar" id="meter-stereo"></div></div></div>
          <div class="meter-group"><div class="meter-label">Gain Red</div><div class="meter"><div class="meter-bar" id="meter-gain" style="background: linear-gradient(to top, var(--neon-blue), var(--neon-green));"></div></div></div>
          <div class="meter-group"><div class="meter-label">Loudness GR</div><div class="meter"><div class="meter-bar" id="meter-loudness-gr"></div></div></div>
          <div class="meter-group"><div class="meter-label">Limiter</div><div class="meter"><div class="meter-bar" id="meter-limiter"></div></div></div>
          <div class="meter-group"><div class="meter-label">Bass Lim</div><div class="meter"><div class="meter-bar" id="meter-bass"></div></div></div>
          <div class="meter-group"><div class="meter-label">Loudness</div><div class="meter"><div class="meter-bar" id="meter-loudness" style="background: linear-gradient(to top, var(--neon-green), var(--neon-blue));"></div></div></div>
          <div class="meter-group"><div class="meter-label">Output</div><div class="meter"><div class="meter-bar" id="meter-output" style="background: linear-gradient(to top, #ff9500, #ffeb3b);"></div></div></div>
        </div>

        <div class="tab-panels">
          <div class="tab-panel active" id="tab-lessmore">
            <div class="controls-grid">
              <div class="control-panel">
                <div class="panel-title">Less-More Processing</div>
                <div class="control-group"><div class="control-label">Less-More Amount <span class="control-value" id="val-lessmore">9.5</span></div><input type="range" id="lessmore" min="0" max="10" step="0.1" value="9.5"></div>
                <div class="control-group"><div class="control-label" style="color: var(--neon-green);">Less-More Active</div><div class="toggle-group"><div class="toggle-btn active" data-param="lessmore-state" data-value="1">On</div><div class="toggle-btn" data-param="lessmore-state" data-value="0">Off</div></div></div>
              </div>
              <div class="control-panel">
                <div class="panel-title">Pass-through</div>
                <div class="control-group"><div class="control-label">Pass-through Gain <span class="control-value" id="val-passthrough">0 dB</span></div><input type="range" id="passthrough" min="-12" max="12" step="0.1" value="0"></div>
                <div class="toggle-group"><div class="toggle-btn" data-param="passthrough-switch" data-value="in">In</div><div class="toggle-btn active" data-param="passthrough-switch" data-value="out">Out</div></div>
              </div>
            </div>
          </div>
          <div class="tab-panel" id="tab-agc">
            <div class="controls-grid">
              <div class="control-panel">
                <div class="panel-title">Automatic Gain Control</div>
                <div class="control-group"><div class="control-label">AGC Threshold <span class="control-value" id="val-agc">-18 dB</span></div><input type="range" id="agc-gate" min="-40" max="0" step="1" value="-18"></div>
                <div class="control-group"><div class="control-label">AGC Attack <span class="control-value" id="val-agc-attack">10 ms</span></div><input type="range" id="agc-attack" min="1" max="100" step="1" value="10"></div>
                <div class="control-group"><div class="control-label">AGC Release <span class="control-value" id="val-agc-release">250 ms</span></div><input type="range" id="agc-release" min="10" max="1000" step="10" value="250"></div>
                <div class="control-group"><div class="control-label">AGC Ratio <span class="control-value" id="val-agc-ratio">3:1</span></div><input type="range" id="agc-ratio" min="1" max="20" step="0.5" value="3"></div>
              </div>
              <div class="control-panel">
                <div class="panel-title">AGC Gate</div>
                <div class="control-group"><div class="control-label">Gate Threshold <span class="control-value" id="val-gate">-24 dB</span></div><input type="range" id="gate-threshold" min="-60" max="0" step="1" value="-24"></div>
                <div class="toggle-group"><div class="toggle-btn active" data-param="agc-enable" data-value="1">Enable AGC</div><div class="toggle-btn" data-param="agc-enable" data-value="0">Bypass</div></div>
              </div>
            </div>
          </div>
          <div class="tab-panel" id="tab-eq">
            <div class="controls-grid">
              <div class="control-panel">
                <div class="panel-title">Equalizer</div>
                <div style="height: 100px; background: #1a1a1a; border-radius: 5px; display: flex; align-items: flex-end; justify-content: space-around; padding: 10px; gap: 5px; margin: 10px 0;">
                  <div style="flex:1; background: linear-gradient(to top, var(--neon-blue), var(--neon-green)); height: 60%; border-radius: 3px 3px 0 0;"></div>
                  <div style="flex:1; background: linear-gradient(to top, var(--neon-blue), var(--neon-green)); height: 80%; border-radius: 3px 3px 0 0;"></div>
                  <div style="flex:1; background: linear-gradient(to top, var(--neon-blue), var(--neon-green)); height: 45%; border-radius: 3px 3px 0 0;"></div>
                  <div style="flex:1; background: linear-gradient(to top, var(--neon-blue), var(--neon-green)); height: 70%; border-radius: 3px 3px 0 0;"></div>
                  <div style="flex:1; background: linear-gradient(to top, var(--neon-blue), var(--neon-green)); height: 90%; border-radius: 3px 3px 0 0;"></div>
                </div>
              </div>
              <div class="control-panel">
                <div class="panel-title">EQ Bands</div>
                <div class="control-group"><div class="control-label">Low Shelf (32Hz) <span class="control-value" id="val-eq-low">0 dB</span></div><input type="range" id="eq-low" min="-12" max="12" step="0.5" value="0"></div>
                <div class="control-group"><div class="control-label">Mid (1kHz) <span class="control-value" id="val-eq-mid">0 dB</span></div><input type="range" id="eq-mid" min="-12" max="12" step="0.5" value="0"></div>
                <div class="control-group"><div class="control-label">High Shelf (10kHz) <span class="control-value" id="val-eq-high">0 dB</span></div><input type="range" id="eq-high" min="-12" max="12" step="0.5" value="0"></div>
                <div class="toggle-group"><div class="toggle-btn active" data-param="eq-enable" data-value="1">EQ On</div><div class="toggle-btn" data-param="eq-enable" data-value="0">EQ Off</div></div>
              </div>
            </div>
          </div>
          <div class="tab-panel" id="tab-stereo">
            <div class="controls-grid">
              <div class="control-panel">
                <div class="panel-title">Stereo Enhancement</div>
                <div class="control-group"><div class="control-label">Stereo Width <span class="control-value" id="val-stereo-width">50%</span></div><input type="range" id="stereo-width" min="0" max="100" step="1" value="50"></div>
                <div class="control-group"><div class="control-label">Mid/Side Balance <span class="control-value" id="val-ms-balance">0</span></div><input type="range" id="ms-balance" min="-100" max="100" step="1" value="0"></div>
                <div class="toggle-group"><div class="toggle-btn active" data-param="stereo-enable" data-value="1">Enhance On</div><div class="toggle-btn" data-param="stereo-enable" data-value="0">Mono</div></div>
              </div>
              <div class="control-panel">
                <div class="panel-title">Stereo Meter</div>
                <div style="height: 100px; background: #1a1a1a; border-radius: 5px; display: flex; align-items: center; justify-content: center; margin: 10px 0;">
                  <div style="text-align: center;"><div style="font-size: 32px; color: var(--neon-green);" id="stereo-meter-value">50%</div><div style="font-size: 11px; color: #666;">Width</div></div>
                </div>
              </div>
            </div>
          </div>
          <div class="tab-panel" id="tab-multiband">
            <div class="controls-grid">
              <div class="control-panel">
                <div class="panel-title">Multiband Compressor</div>
                <div class="control-group"><div class="control-label">Low Band (20-250Hz) <span class="control-value" id="val-mb-low">-3 dB</span></div><input type="range" id="mb-low" min="-20" max="0" step="0.5" value="-3"></div>
                <div class="control-group"><div class="control-label">Low-Mid (250-1kHz) <span class="control-value" id="val-mb-lowmid">-2 dB</span></div><input type="range" id="mb-lowmid" min="-20" max="0" step="0.5" value="-2"></div>
                <div class="control-group"><div class="control-label">Mid (1k-4kHz) <span class="control-value" id="val-mb-mid">-1 dB</span></div><input type="range" id="mb-mid" min="-20" max="0" step="0.5" value="-1"></div>
                <div class="control-group"><div class="control-label">High-Mid (4k-8kHz) <span class="control-value" id="val-mb-highmid">-2 dB</span></div><input type="range" id="mb-highmid" min="-20" max="0" step="0.5" value="-2"></div>
                <div class="control-group"><div class="control-label">High (8k-20kHz) <span class="control-value" id="val-mb-high">-4 dB</span></div><input type="range" id="mb-high" min="-20" max="0" step="0.5" value="-4"></div>
              </div>
              <div class="control-panel">
                <div class="panel-title">Crossover Frequencies</div>
                <div class="control-group"><div class="control-label">Low/Mid Crossover <span class="control-value" id="val-cross-1">250 Hz</span></div><input type="range" id="cross-1" min="100" max="500" step="10" value="250"></div>
                <div class="control-group"><div class="control-label">Mid/High Crossover <span class="control-value" id="val-cross-2">4000 Hz</span></div><input type="range" id="cross-2" min="2000" max="8000" step="100" value="4000"></div>
              </div>
            </div>
          </div>
          <div class="tab-panel" id="tab-compressors">
            <div class="controls-grid">
              <div class="control-panel">
                <div class="panel-title">Main Compressor</div>
                <div class="control-group"><div class="control-label">Threshold <span class="control-value" id="val-comp-threshold">-14 dB</span></div><input type="range" id="comp-threshold" min="-40" max="0" step="0.5" value="-14"></div>
                <div class="control-group"><div class="control-label">Ratio <span class="control-value" id="val-comp-ratio">4:1</span></div><input type="range" id="comp-ratio" min="1" max="20" step="0.5" value="4"></div>
                <div class="control-group"><div class="control-label">Attack <span class="control-value" id="val-comp-attack">10 ms</span></div><input type="range" id="comp-attack" min="1" max="100" step="1" value="10"></div>
                <div class="control-group"><div class="control-label">Release <span class="control-value" id="val-comp-release">100 ms</span></div><input type="range" id="comp-release" min="10" max="500" step="5" value="100"></div>
                <div class="control-group"><div class="control-label">Makeup Gain <span class="control-value" id="val-comp-makeup">3 dB</span></div><input type="range" id="comp-makeup" min="0" max="12" step="0.5" value="3"></div>
              </div>
              <div class="control-panel">
                <div class="panel-title">Compressor Meters</div>
                <div style="display: flex; gap: 20px; justify-content: center; margin: 20px 0;">
                  <div style="text-align: center;"><div style="height: 100px; width: 30px; background: #1a1a1a; border-radius: 3px; position: relative; overflow: hidden;"><div style="position: absolute; bottom: 0; width: 100%; height: 70%; background: linear-gradient(to top, var(--danger), #ffa502);"></div></div><div style="font-size: 10px; color: #888; margin-top: 5px;">Input</div></div>
                  <div style="text-align: center;"><div style="height: 100px; width: 30px; background: #1a1a1a; border-radius: 3px; position: relative; overflow: hidden;"><div style="position: absolute; bottom: 0; width: 100%; height: 85%; background: linear-gradient(to top, var(--neon-blue), var(--neon-green));"></div></div><div style="font-size: 10px; color: #888; margin-top: 5px;">Output</div></div>
                  <div style="text-align: center;"><div style="height: 100px; width: 30px; background: #1a1a1a; border-radius: 3px; position: relative; overflow: hidden;"><div style="position: absolute; bottom: 0; width: 100%; height: 40%; background: linear-gradient(to top, var(--neon-purple), var(--neon-blue));"></div></div><div style="font-size: 10px; color: #888; margin-top: 5px;">GR</div></div>
                </div>
                <div class="toggle-group"><div class="toggle-btn active" data-param="comp-enable" data-value="1">Compress On</div><div class="toggle-btn" data-param="comp-enable" data-value="0">Bypass</div></div>
              </div>
            </div>
          </div>
          <div class="tab-panel" id="tab-speech">
            <div class="controls-grid">
              <div class="control-panel">
                <div class="panel-title">Voice Processor</div>
                <div class="control-group"><div class="control-label">Voice Presence <span class="control-value" id="val-voice-presence">70%</span></div><input type="range" id="voice-presence" min="0" max="100" step="1" value="70"></div>
                <div class="control-group"><div class="control-label">De-Esser <span class="control-value" id="val-deesser">-6 dB</span></div><input type="range" id="deesser" min="-20" max="0" step="0.5" value="-6"></div>
                <div class="control-group"><div class="control-label">Vocal Clarity <span class="control-value" id="val-clarity">80%</span></div><input type="range" id="clarity" min="0" max="100" step="1" value="80"></div>
                <div class="toggle-group"><div class="toggle-btn active" data-param="speech-enable" data-value="1">Voice Mode</div><div class="toggle-btn" data-param="speech-enable" data-value="0">Music Mode</div></div>
              </div>
              <div class="control-panel">
                <div class="panel-title">Noise Gate</div>
                <div class="control-group"><div class="control-label">Gate Threshold <span class="control-value" id="val-speech-gate">-30 dB</span></div><input type="range" id="speech-gate" min="-60" max="0" step="1" value="-30"></div>
                <div class="control-group"><div class="control-label">Gate Attack <span class="control-value" id="val-gate-attack">5 ms</span></div><input type="range" id="gate-attack" min="1" max="50" step="1" value="5"></div>
                <div class="control-group"><div class="control-label">Gate Release <span class="control-value" id="val-gate-release">100 ms</span></div><input type="range" id="gate-release" min="10" max="500" step="10" value="100"></div>
              </div>
            </div>
          </div>
          <div class="tab-panel" id="tab-bandmix">
            <div class="controls-grid">
              <div class="control-panel">
                <div class="panel-title">Band Mixing</div>
                <div class="control-group"><div class="control-label">Bass Mix <span class="control-value" id="val-bass-mix">80%</span></div><input type="range" id="bass-mix" min="0" max="100" step="1" value="80"></div>
                <div class="control-group"><div class="control-label">Mid Mix <span class="control-value" id="val-mid-mix">90%</span></div><input type="range" id="mid-mix" min="0" max="100" step="1" value="90"></div>
                <div class="control-group"><div class="control-label">Treble Mix <span class="control-value" id="val-treble-mix">70%</span></div><input type="range" id="treble-mix" min="0" max="100" step="1" value="70"></div>
                <div class="control-group"><div class="control-label">Overall Balance <span class="control-value" id="val-overall">0</span></div><input type="range" id="overall" min="-50" max="50" step="1" value="0"></div>
              </div>
              <div class="control-panel">
                <div class="panel-title">Band Visualizer</div>
                <div style="display: flex; height: 120px; gap: 10px; align-items: flex-end; justify-content: center; margin: 10px 0;">
                  <div style="flex: 1; background: linear-gradient(to top, var(--danger), #ffa502); height: 80%; border-radius: 3px 3px 0 0;"></div>
                  <div style="flex: 1; background: linear-gradient(to top, var(--neon-blue), var(--neon-green)); height: 90%; border-radius: 3px 3px 0 0;"></div>
                  <div style="flex: 1; background: linear-gradient(to top, var(--neon-purple), var(--neon-blue)); height: 70%; border-radius: 3px 3px 0 0;"></div>
                </div>
                <div style="display: flex; justify-content: space-around; font-size: 10px; color: #666;"><span>Bass</span><span>Mid</span><span>Treble</span></div>
              </div>
            </div>
          </div>
          <div class="tab-panel" id="tab-limiters">
            <div class="controls-grid">
              <div class="control-panel">
                <div class="panel-title">True Peak Limiter</div>
                <div class="control-group"><div class="control-label">Ceiling <span class="control-value" id="val-ceiling">-1 dB</span></div><input type="range" id="ceiling" min="-6" max="0" step="0.1" value="-1"></div>
                <div class="control-group"><div class="control-label">Lookahead <span class="control-value" id="val-lookahead">5 ms</span></div><input type="range" id="lookahead" min="1" max="20" step="0.5" value="5"></div>
                <div class="control-group"><div class="control-label">Release <span class="control-value" id="val-lim-release">50 ms</span></div><input type="range" id="lim-release" min="10" max="200" step="5" value="50"></div>
                <div class="toggle-group"><div class="toggle-btn active" data-param="limiter-enable" data-value="1">Limiter On</div><div class="toggle-btn" data-param="limiter-enable" data-value="0">Bypass</div></div>
              </div>
              <div class="control-panel">
                <div class="panel-title">Bass Limiter</div>
                <div class="control-group"><div class="control-label">Bass Threshold <span class="control-value" id="val-bass-threshold">-6 dB</span></div><input type="range" id="bass-threshold" min="-20" max="0" step="0.5" value="-6"></div>
                <div class="control-group"><div class="control-label">Bass Ratio <span class="control-value" id="val-bass-ratio">20:1</span></div><input type="range" id="bass-ratio" min="1" max="50" step="0.5" value="20"></div>
              </div>
            </div>
          </div>
          <div class="tab-panel" id="tab-synthesizer">
            <div class="controls-grid">
              <div class="control-panel">
                <div class="panel-title">Stereo Synthesizer</div>
                <div class="control-group"><div class="control-label">Harmonic Width <span class="control-value" id="val-harmonic-width">60%</span></div><input type="range" id="harmonic-width" min="0" max="100" step="1" value="60"></div>
                <div class="control-group"><div class="control-label">Phase Spread <span class="control-value" id="val-phase-spread">30°</span></div><input type="range" id="phase-spread" min="0" max="180" step="1" value="30"></div>
                <div class="control-group"><div class="control-label">Stereo Delay <span class="control-value" id="val-stereo-delay">10 ms</span></div><input type="range" id="stereo-delay" min="0" max="50" step="1" value="10"></div>
                <div class="toggle-group"><div class="toggle-btn active" data-param="synth-enable" data-value="1">Synth On</div><div class="toggle-btn" data-param="synth-enable" data-value="0">Bypass</div></div>
              </div>
              <div class="control-panel">
                <div class="panel-title">Stereo Field</div>
                <div style="height: 150px; background: #1a1a1a; border-radius: 5px; position: relative; margin: 10px 0;">
                  <div style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 100px; height: 100px; border: 2px solid var(--neon-blue); border-radius: 50%;"></div>
                  <div style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 50px; height: 50px; border: 2px solid var(--neon-green); border-radius: 50%;"></div>
                  <div style="position: absolute; top: 20%; left: 30%; width: 10px; height: 10px; background: var(--neon-blue); border-radius: 50%;"></div>
                  <div style="position: absolute; top: 70%; right: 25%; width: 10px; height: 10px; background: var(--neon-green); border-radius: 50%;"></div>
                  <div style="position: absolute; bottom: 10px; left: 10px; font-size: 10px; color: #666;">L</div>
                  <div style="position: absolute; bottom: 10px; right: 10px; font-size: 10px; color: #666;">R</div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="audio-controls">
          <button class="btn" id="btn-start">🎵 Start Processing</button>
          <button class="btn btn-secondary" id="btn-stop">⏹ Stop</button>
          <button class="btn btn-secondary" id="btn-load">📂 Load Audio</button>
          <button class="btn btn-secondary" id="btn-test">🔊 Test Tone</button>
          <input type="file" id="audio-file" accept="audio/*" style="display: none;">
        </div>

        <div class="branding">
          <div class="brand-name">WebAudio Pro</div>
          <div>Enterprise Class Broadcast Audio Processor</div>
        </div>
      </div>
    </div>
  </div>

  <script type="module">
    // ==========================================
    // CORE APP CONTROLLER
    // ==========================================
    const App = {
      ctx: null,
      engine: null,
      io: null,
      init() {
        // Inicialización diferida para cumplir con políticas de autoplay
        const initAudio = async () => {
          this.ctx = new (window.AudioContext || window.webkitAudioContext)({ sampleRate: 48000, latencyHint: 'interactive' });
          if(this.ctx.state === 'suspended') await this.ctx.resume();
          
          // Inicializar motor de procesamiento
          this.engine = new AudioProcessor(this.ctx);
          // Inicializar I/O Routing
          this.io = new IORouting(this.ctx, this.engine);
          
          this.bindControls();
          Presets.init();
          Tabs.init();
          Meters.startLoop();
          document.removeEventListener('click', initAudio);
          console.log('[Nexus] Sistema de audio inicializado y listo.');
        };
        
        // Escuchar primer clic para iniciar audio
        document.addEventListener('click', initAudio, { once: true });
      },
      bindControls() {
        document.getElementById('btn-start').addEventListener('click', () => {
          if(!this.engine.source) {
            this.engine.play(this.engine.generateTestTone());
            document.getElementById('btn-start').textContent = '⏸ Pause';
            document.getElementById('btn-start').classList.add('active');
          } else {
            this.engine.stop();
            document.getElementById('btn-start').textContent = '🎵 Start Processing';
            document.getElementById('btn-start').classList.remove('active');
          }
        });
        document.getElementById('btn-stop').addEventListener('click', () => {
          this.engine.stop();
          document.getElementById('btn-start').textContent = '🎵 Start Processing';
          document.getElementById('btn-start').classList.remove('active');
        });
        document.getElementById('btn-load').addEventListener('click', () => document.getElementById('audio-file').click());
        document.getElementById('audio-file').addEventListener('change', async e => {
          if(e.target.files[0] && this.ctx) {
            const buf = await this.ctx.decodeAudioData(await e.target.files[0].arrayBuffer());
            this.engine.play(buf);
            document.getElementById('btn-start').click();
          }
        });
        document.getElementById('btn-test').addEventListener('click', () => {
          this.engine.play(this.engine.generateTestTone());
          document.getElementById('btn-start').click();
        });
      }
    };

    // ==========================================
    // AUDIO PROCESSOR ENGINE
    // ==========================================
    class AudioProcessor {
      constructor(ctx) {
        this.ctx = ctx;
        this.source = null;
        this.params = { 
          lessmore:9.5, passthrough:0, agcThreshold:-18, agcAttack:10, agcRatio:3,
          hfBoost:3, stereoWidth:50, eqLow:0, eqMid:0, eqHigh:0, compThreshold:-14, compRatio:4 
        };
        this.initChain();
      }
      initChain() {
        // Analyzers
        this.inputAnalyser = this.ctx.createAnalyser(); this.inputAnalyser.fftSize = 2048;
        this.outputAnalyser = this.ctx.createAnalyser(); this.outputAnalyser.fftSize = 2048;

        // Nodes
        this.agc = this.ctx.createDynamicsCompressor(); 
        this.agc.threshold.value = this.params.agcThreshold; 
        this.agc.attack.value = this.params.agcAttack/1000; 
        this.agc.ratio.value = this.params.agcRatio;

        this.eqLow = this.ctx.createBiquadFilter(); this.eqLow.type = 'lowshelf'; this.eqLow.frequency.value = 100; this.eqLow.gain.value = this.params.eqLow;
        this.eqMid = this.ctx.createBiquadFilter(); this.eqMid.type = 'peaking'; this.eqMid.frequency.value = 1000; this.eqMid.Q.value = 1; this.eqMid.gain.value = this.params.eqMid;
        this.eqHigh = this.ctx.createBiquadFilter(); this.eqHigh.type = 'highshelf'; this.eqHigh.frequency.value = 10000; this.eqHigh.gain.value = this.params.eqHigh;
        
        this.hfEnhancer = this.ctx.createBiquadFilter(); this.hfEnhancer.type = 'highshelf'; this.hfEnhancer.frequency.value = 3000; this.hfEnhancer.gain.value = this.params.hfBoost;
        
        this.compressor = this.ctx.createDynamicsCompressor(); 
        this.compressor.threshold.value = this.params.compThreshold; 
        this.compressor.ratio.value = this.params.compRatio;
        this.compressor.knee.value = 10;

        // Chain: Input -> Analyser -> AGC -> EQ Low -> EQ Mid -> EQ High -> HF -> Compressor -> Analyser -> Output
        this.inputAnalyser.connect(this.agc)
          .connect(this.eqLow).connect(this.eqMid).connect(this.eqHigh)
          .connect(this.hfEnhancer)
          .connect(this.compressor)
          .connect(this.outputAnalyser);
      }
      play(buffer) {
        if(this.source) { this.source.stop(); this.source.disconnect(); }
        this.source = this.ctx.createBufferSource();
        this.source.buffer = buffer; this.source.loop = true;
        this.source.connect(this.inputAnalyser);
        this.source.start(0);
      }
      stop() { if(this.source) { this.source.stop(); this.source.disconnect(); this.source = null; } }
      generateTestTone() {
        // Genera ruido rosa con un tono de prueba audible (1kHz sweep)
        const sr = this.ctx.sampleRate, dur = 10, len = sr * dur;
        const buf = this.ctx.createBuffer(2, len, sr);
        let lastOut = 0, phase = 0;
        for(let ch=0; ch<2; ch++) { 
          const d = buf.getChannelData(ch); 
          for(let i=0; i<len; i++) { 
            const white = Math.random()*2-1;
            // Pink noise approximation
            d[i] = (lastOut + (0.02 * white)) / 1.02;
            lastOut = d[i];
            // Add sine wave sweep for testing EQ
            const t = i / sr;
            const freq = 200 + (t * 50); // 200Hz to 700Hz sweep
            d[i] += 0.2 * Math.sin(2 * Math.PI * freq * t + phase);
            d[i] *= 0.8; 
          } 
        }
        return buf;
      }
      updateParam(p, v) {
        this.params[p] = parseFloat(v);
        const val = this.params[p];
        switch(p) {
          case 'agcThreshold': this.agc.threshold.value = val; break;
          case 'agcAttack': this.agc.attack.value = val/1000; break;
          case 'agcRatio': this.agc.ratio.value = val; break;
          case 'hfBoost': this.hfEnhancer.gain.value = val; break;
          case 'eqLow': this.eqLow.gain.value = val; break;
          case 'eqMid': this.eqMid.gain.value = val; break;
          case 'eqHigh': this.eqHigh.gain.value = val; break;
          case 'compThreshold': this.compressor.threshold.value = val; break;
          case 'compRatio': this.compressor.ratio.value = val; break;
        }
      }
    }

    // ==========================================
    // I/O ROUTING MODULE
    // ==========================================
    class IORouting {
      constructor(ctx, engine) {
        this.ctx = ctx; this.engine = engine;
        
        // I/O Nodes
        this.inputGain = ctx.createGain(); this.inputGain.gain.value = 1;
        this.micGain = ctx.createGain(); this.micGain.gain.value = 0.8;
        this.monoSplit = ctx.createChannelSplitter(2); 
        this.monoMerge = ctx.createChannelMerger(2);
        this.stereoPanner = ctx.createStereoPanner(); this.stereoPanner.pan.value = 0;
        this.outputLimiter = ctx.createDynamicsCompressor(); 
        this.outputLimiter.threshold.value = -0.5; this.outputLimiter.ratio.value = 20; this.outputLimiter.knee.value = 0;
        this.masterGain = ctx.createGain(); this.masterGain.gain.value = 1;
        this.headphoneMix = ctx.createGain(); this.headphoneMix.gain.value = 0;
        
        // State
        this.isMono = false; this.isMuted = false; this.isNR = true; 
        this.isLimiter = true; this.isMonitor = false; 
        this.clipTimer = null;
        
        this.initGraph(); 
        this.bindControls(); 
        this.loadDevices();
      }
      initGraph() {
        // Input Chain: InputGain -> MicGain -> MonoSplit -> MonoMerge -> Engine Input
        this.monoSplit.connect(this.monoMerge, 0, 0); 
        this.monoSplit.connect(this.monoMerge, 1, 1);
        this.inputGain.connect(this.micGain).connect(this.monoSplit);
        this.monoMerge.connect(this.engine.inputAnalyser);
        
        // Output Chain: Engine Output -> StereoPanner -> Limiter -> MasterGain -> HeadphoneMix -> Destination
        this.engine.outputAnalyser.connect(this.stereoPanner)
          .connect(this.outputLimiter)
          .connect(this.masterGain)
          .connect(this.headphoneMix)
          .connect(this.ctx.destination);
        
        // Headphone monitor direct path
        this.masterGain.connect(this.ctx.destination);
      }
      async loadDevices() {
        try {
          const devs = await navigator.mediaDevices.enumerateDevices();
          const inSel = document.getElementById('input-device-select');
          const outSel = document.getElementById('output-device-select');
          inSel.innerHTML = ''; 
          outSel.innerHTML = '<option value="">Default System Output</option>';
          
          devs.forEach(d => {
            const opt = document.createElement('option'); 
            opt.value = d.deviceId; 
            opt.text = d.label || `${d.kind} ${d.deviceId.slice(0,8)}...`;
            if(d.kind === 'audioinput') inSel.appendChild(opt);
            if(d.kind === 'audiooutput') outSel.appendChild(opt);
          });
          if(inSel.options.length > 0) { 
            inSel.value = inSel.options[0].value; 
            this.startInput(inSel.value); 
          }
        } catch(e) { console.warn('Devices error:', e); }
      }
      async startInput(id) {
        if(this.stream) this.stream.getTracks().forEach(t => t.stop());
        try {
          this.stream = await navigator.mediaDevices.getUserMedia({ 
            audio: { 
              deviceId: id ? {exact:id} : undefined, 
              echoCancellation: this.isNR, 
              noiseSuppression: this.isNR, 
              autoGainControl: false 
            } 
          });
          const src = this.ctx.createMediaStreamSource(this.stream);
          src.connect(this.inputGain);
        } catch(e) { console.error('Mic error:', e); }
      }
      bindControls() {
        const bind = (id, node, param, dispId, suffix='', tx=v=>v) => {
          const el = document.getElementById(id);
          if(!el) return;
          el.addEventListener('input', e => {
            const v = tx(parseFloat(e.target.value));
            if(node[param]) node[param].value = v;
            const dispEl = document.getElementById(dispId);
            if(dispEl) dispEl.textContent = e.target.value + suffix;
          });
        };
        
        // Bind I/O Controls
        bind('input-gain', this.inputGain, 'gain', 'val-input-gain', ' dB', v=>Math.pow(10,v/20));
        bind('mic-level', this.micGain, 'gain', 'val-mic-level', '%', v=>v/100);
        bind('master-volume', this.masterGain, 'gain', 'val-master-vol', ' dB', v=>Math.pow(10,v/20));
        
        // Balance
        const balEl = document.getElementById('output-balance');
        if(balEl) {
          balEl.addEventListener('input', e => {
            this.stereoPanner.pan.value = e.target.value/100;
            document.getElementById('val-balance').textContent = e.target.value==0 ? 'C' : (e.target.value<0 ? `L${Math.abs(e.target.value)}` : `R${e.target.value}`);
          });
        }

        // Device Selectors
        const inSel = document.getElementById('input-device-select');
        if(inSel) inSel.addEventListener('change', e => this.startInput(e.target.value));
        
        const outSel = document.getElementById('output-device-select');
        if(outSel) outSel.addEventListener('change', async e => {
          if(e.target.value && this.ctx.destination.setSinkId) {
            try { await this.ctx.destination.setSinkId(e.target.value); } catch(err){}
          }
        });

        // Toggles
        const bindToggle = (id, prop, activeCb, inactiveCb) => {
          const btn = document.getElementById(id);
          if(!btn) return;
          btn.addEventListener('click', e => {
            this[prop] = !this[prop];
            btn.classList.toggle('active');
            if(this[prop]) activeCb(e); else inactiveCb(e);
          });
        };

        bindToggle('btn-input-mono', 'isMono', 
          () => {
            document.getElementById('input-mode-indicator').textContent = 'MONO';
            this.monoMerge.disconnect(); this.monoSplit.disconnect();
            this.inputGain.connect(this.micGain).connect(this.monoSplit);
            this.monoSplit.connect(this.monoMerge, 0, 0); 
            this.monoSplit.connect(this.monoMerge, 0, 1);
            this.monoMerge.connect(this.engine.inputAnalyser);
          },
          () => {
            document.getElementById('input-mode-indicator').textContent = 'STEREO';
            this.monoMerge.disconnect(); this.monoSplit.disconnect();
            this.inputGain.connect(this.micGain).connect(this.monoSplit);
            this.monoSplit.connect(this.monoMerge, 0, 0); 
            this.monoSplit.connect(this.monoMerge, 1, 1);
            this.monoMerge.connect(this.engine.inputAnalyser);
          }
        );

        bindToggle('btn-noise-reduction', 'isNR',
          (e) => { e.target.textContent = '🔇 NR ON'; if(this.stream) this.startInput(document.getElementById('input-device-select').value); },
          (e) => { e.target.textContent = '🔇 NR OFF'; if(this.stream) this.startInput(document.getElementById('input-device-select').value); }
        );

        bindToggle('btn-input-mute', 'isMuted',
          (e) => { this.inputGain.gain.value = 0; },
          (e) => { const v = parseFloat(document.getElementById('input-gain').value); this.inputGain.gain.value = Math.pow(10,v/20); }
        );

        bindToggle('btn-output-limiter', 'isLimiter',
          (e) => {
            this.stereoPanner.disconnect(); this.outputLimiter.disconnect();
            this.stereoPanner.connect(this.outputLimiter);
            this.outputLimiter.connect(this.masterGain);
          },
          (e) => {
            this.stereoPanner.disconnect(); this.outputLimiter.disconnect();
            this.stereoPanner.connect(this.masterGain);
          }
        );

        bindToggle('btn-headphone-monitor', 'isMonitor',
          (e) => { this.headphoneMix.gain.value = 0.7; },
          (e) => { this.headphoneMix.gain.value = 0; }
        );
      }
    }

    // ==========================================
    // PRESETS SYSTEM (FIXED)
    // ==========================================
    const Presets = {
       {
        'broadcast-hd': { name:'Broadcast HD', mode:'Music', lufs:-14, params:{lessmore:9.5,passthrough:0,agcThreshold:-18,agcAttack:10,agcRatio:3,hfBoost:3,stereoWidth:50,eqLow:0,eqMid:0,eqHigh:0,compThreshold:-14,compRatio:4,masterVolume:0} },
        'voice-over': { name:'Voice Over Pro', mode:'Voice', lufs:-16, params:{lessmore:7,passthrough:2,agcThreshold:-20,agcAttack:5,agcRatio:3,hfBoost:6,stereoWidth:30,eqLow:-2,eqMid:3,eqHigh:4,compThreshold:-16,compRatio:3,masterVolume:2} },
        'music-master': { name:'Music Master', mode:'Music', lufs:-10, params:{lessmore:8.5,passthrough:-1,agcThreshold:-16,agcAttack:15,agcRatio:4,hfBoost:4,stereoWidth:75,eqLow:2,eqMid:1,eqHigh:2,compThreshold:-10,compRatio:6,masterVolume:-1} },
        'podcast-studio': { name:'Podcast Studio', mode:'Voice', lufs:-16, params:{lessmore:6,passthrough:1,agcThreshold:-22,agcAttack:8,agcRatio:2.5,hfBoost:5,stereoWidth:20,eqLow:-1,eqMid:2,eqHigh:3,compThreshold:-16,compRatio:2.5,masterVolume:1} },
        'radio-fm': { name:'Radio FM', mode:'Music', lufs:-12, params:{lessmore:9,passthrough:0,agcThreshold:-15,agcAttack:12,agcRatio:5,hfBoost:5,stereoWidth:60,eqLow:1,eqMid:2,eqHigh:3,compThreshold:-12,compRatio:5,masterVolume:0} },
        'streaming-live': { name:'Streaming Live', mode:'Mixed', lufs:-14, params:{lessmore:7.5,passthrough:0,agcThreshold:-18,agcAttack:10,agcRatio:4,hfBoost:4,stereoWidth:45,eqLow:0,eqMid:1,eqHigh:2,compThreshold:-14,compRatio:4,masterVolume:0} },
        'audiobook': { name:'Audiobook', mode:'Voice', lufs:-18, params:{lessmore:5,passthrough:2,agcThreshold:-24,agcAttack:6,agcRatio:2,hfBoost:3,stereoWidth:15,eqLow:-3,eqMid:1,eqHigh:2,compThreshold:-18,compRatio:2,masterVolume:2} },
        'club-dj': { name:'Club/DJ', mode:'Music', lufs:-9, params:{lessmore:10,passthrough:-2,agcThreshold:-12,agcAttack:20,agcRatio:8,hfBoost:6,stereoWidth:85,eqLow:3,eqMid:0,eqHigh:4,compThreshold:-9,compRatio:8,masterVolume:-2} }
      },
      custom: JSON.parse(localStorage.getItem('customPresets')||'{}'),
      init() {
        document.querySelectorAll('.preset-btn').forEach(btn => {
          btn.addEventListener('click', () => this.load(btn.dataset.preset));
        });
        
        document.getElementById('btn-save-preset').addEventListener('click', () => {
          const n = prompt('Nombre del preset:'); if(!n) return;
          const k = 'custom-'+Date.now();
          // Guardar parámetros actuales del Engine y del I/O
          this.custom[k] = { 
            name:n, 
            params:{
              ...App.engine.params,
              masterVolume: parseFloat(document.getElementById('master-volume').value),
              stereoWidth: parseFloat(document.getElementById('stereo-width').value)
            } 
          };
          localStorage.setItem('customPresets', JSON.stringify(this.custom));
          this.updateCustomSelect(); alert('Preset guardado: '+n);
        });
        
        document.getElementById('btn-del-preset').addEventListener('click', () => {
          const k = document.getElementById('custom-preset-select').value;
          if(k && k.startsWith('custom-') && confirm('¿Borrar '+this.custom[k].name+'?')) {
            delete this.custom[k]; localStorage.setItem('customPresets', JSON.stringify(this.custom));
            this.updateCustomSelect();
          }
        });
        this.updateCustomSelect();
      },
      load(key) {
        const p = this.data[key] || this.custom[key];
        if(!p) return;
        
        // UI Updates
        document.querySelectorAll('.preset-btn').forEach(b => b.classList.remove('active'));
        document.querySelector(`.preset-btn[data-preset="${key}"]`)?.classList.add('active');
        document.getElementById('active-preset-name').textContent = p.name + (this.custom[key]?' (Custom)':'');
        document.getElementById('display-mode').textContent = p.mode;
        document.getElementById('display-lufs').textContent = p.lufs + ' LUFS';
        
        // Aplicar parámetros a Engine y I/O
        Object.entries(p.params).forEach(([k,v]) => {
          const sliderId = k.replace(/([A-Z])/g,'-$1').toLowerCase();
          const slider = document.getElementById(sliderId);
          if(slider) { 
            slider.value = v; 
            slider.dispatchEvent(new Event('input')); 
          }
          
          // Actualizar Engine
          if(App.engine && ['agcThreshold','agcAttack','agcRatio','hfBoost','eqLow','eqMid','eqHigh','compThreshold','compRatio'].includes(k)) {
            App.engine.updateParam(k,v);
          }
          
          // Actualizar I/O directamente si es necesario
          if(App.io) {
            if(k === 'masterVolume') {
               document.getElementById('master-volume').value = v;
               document.getElementById('master-volume').dispatchEvent(new Event('input'));
            }
            if(k === 'stereoWidth') {
               document.getElementById('stereo-width').value = v;
               document.getElementById('stereo-width').dispatchEvent(new Event('input'));
            }
          }
        });
      },
      updateCustomSelect() {
        const sel = document.getElementById('custom-preset-select');
        sel.innerHTML = '<option value="">Cargar preset personalizado...</option>';
        Object.keys(this.custom).forEach(k => { const o=document.createElement('option'); o.value=k; o.text=this.custom[k].name; sel.appendChild(o); });
        sel.onchange = () => sel.value && this.load(sel.value);
      }
    };

    // ==========================================
    // TABS & CONTROLS SYSTEM
    // ==========================================
    const Tabs = {
      init() {
        document.querySelectorAll('.tab').forEach(tab => {
          tab.addEventListener('click', () => {
            document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
            document.querySelectorAll('.tab-panel').forEach(p => p.classList.remove('active'));
            tab.classList.add('active');
            document.getElementById(tab.dataset.tab).classList.add('active');
          });
        });
        
        const sliderMap = {
          'lessmore': 'val-lessmore|',
          'passthrough': 'val-passthrough| dB',
          'agc-gate': 'val-agc| dB',
          'agc-attack': 'val-agc-attack| ms',
          'agc-ratio': 'val-agc-ratio|:1',
          'hf-boost': 'val-hf| dB',
          'eq-low': 'val-eq-low| dB',
          'eq-mid': 'val-eq-mid| dB',
          'eq-high': 'val-eq-high| dB',
          'stereo-width': 'val-stereo-width|%',
          'comp-threshold': 'val-comp-threshold| dB',
          'comp-ratio': 'val-comp-ratio|:1',
          'voice-presence': 'val-voice-presence|%',
          'deesser': 'val-deesser| dB',
          'clarity': 'val-clarity|%',
          'speech-gate': 'val-speech-gate| dB',
          'gate-attack': 'val-gate-attack| ms',
          'gate-release': 'val-gate-release| ms',
          'mb-low': 'val-mb-low| dB',
          'mb-lowmid': 'val-mb-lowmid| dB',
          'mb-mid': 'val-mb-mid| dB',
          'mb-highmid': 'val-mb-highmid| dB',
          'mb-high': 'val-mb-high| dB',
          'cross-1': 'val-cross-1| Hz',
          'cross-2': 'val-cross-2| Hz',
          'comp-attack': 'val-comp-attack| ms',
          'comp-release': 'val-comp-release| ms',
          'comp-makeup': 'val-comp-makeup| dB',
          'ceiling': 'val-ceiling| dB',
          'lookahead': 'val-lookahead| ms',
          'lim-release': 'val-lim-release| ms',
          'bass-threshold': 'val-bass-threshold| dB',
          'bass-ratio': 'val-bass-ratio|:1',
          'harmonic-width': 'val-harmonic-width|%',
          'phase-spread': 'val-phase-spread|°',
          'stereo-delay': 'val-stereo-delay| ms',
          'bass-mix': 'val-bass-mix|%',
          'mid-mix': 'val-mid-mix|%',
          'treble-mix': 'val-treble-mix|%',
          'overall': 'val-overall|',
          'ms-balance': 'val-ms-balance|',
          'phase-corrector': 'val-phase| Hz'
        };

        Object.entries(sliderMap).forEach(([id, cfg]) => {
          const [disp, suf] = cfg.split('|');
          const el = document.getElementById(id); 
          if(!el) return;
          el.addEventListener('input', e => {
            document.getElementById(disp).textContent = e.target.value + suf;
            const paramKey = id.replace(/-/g, '');
            if(App.engine) App.engine.updateParam(paramKey, parseFloat(e.target.value));
          });
        });

        document.querySelectorAll('.toggle-btn').forEach(btn => {
          btn.addEventListener('click', function() {
            document.querySelectorAll(`[data-param="${this.dataset.param}"]`).forEach(b => b.classList.remove('active'));
            this.classList.add('active');
          });
        });
      }
    };

    // ==========================================
    // METERS LOOP
    // ==========================================
    const Meters = {
      startLoop() {
        const draw = () => {
          requestAnimationFrame(draw);
          if(!App.engine?.inputAnalyser || !App.io?.engine?.outputAnalyser) return;

          const inData = new Uint8Array(App.engine.inputAnalyser.frequencyBinCount);
          App.engine.inputAnalyser.getByteFrequencyData(inData);
          const inRMS = Math.sqrt(inData.reduce((a,b)=>a+b*b,0)/inData.length);
          const inDB = 20 * Math.log10(inRMS/255 || 0.0001);
          document.getElementById('in-vu-l').style.height = document.getElementById('in-vu-r').style.height = Math.min(100, Math.max(0, (inDB+60)*1.67)) + '%';
          document.getElementById('in-peak').textContent = inDB > -100 ? inDB.toFixed(1)+' dB' : '-∞ dB';

          const outTime = new Float32Array(App.io.engine.outputAnalyser.fftSize);
          const outFreq = new Uint8Array(App.io.engine.outputAnalyser.frequencyBinCount);
          App.io.engine.outputAnalyser.getFloatTimeDomainData(outTime);
          App.io.engine.outputAnalyser.getByteFrequencyData(outFreq);

          let rms=0, peak=0;
          for(let i=0; i<outTime.length; i++) { rms+=outTime[i]*outTime[i]; if(Math.abs(outTime[i])>peak) peak=Math.abs(outTime[i]); }
          rms=Math.sqrt(rms/outTime.length);
          const rmsDB=20*Math.log10(rms||0.0001), peakDB=20*Math.log10(peak||0.0001);

          document.getElementById('out-vu-l').style.height = document.getElementById('out-vu-r').style.height = Math.min(100, Math.max(0, (rmsDB+60)*1.67)) + '%';
          document.getElementById('out-rms').textContent = `RMS: ${rmsDB.toFixed(1)} dB`;
          document.getElementById('out-peak').textContent = `PEAK: ${peakDB.toFixed(1)} dB`;

          const clipEl = document.getElementById('output-clip');
          if(peakDB > -0.5) { clipEl.classList.add('active'); clearTimeout(App.io.clipTimer); App.io.clipTimer=setTimeout(()=>clipEl.classList.remove('active'), 300); }

          const canvas = document.getElementById('fft-canvas'); const ctx = canvas.getContext('2d');
          canvas.width = canvas.clientWidth; canvas.height = canvas.clientHeight; ctx.clearRect(0,0,canvas.width,canvas.height);
          const bw = (canvas.width/outFreq.length)*2.5; let x=0;
          for(let i=0; i<outFreq.length; i++) {
            const h = (outFreq[i]/255)*canvas.height;
            ctx.fillStyle = `hsl(${180+(outFreq[i]/255)*60}, 100%, 50%)`;
            ctx.fillRect(x, canvas.height-h, bw-1, h); x+=bw;
          }

          const data = { input: Math.min(100,Math.max(0,(inDB+36)*2.78)), output: Math.min(100,Math.max(0,(rmsDB+36)*2.78)), gainRed: Math.min(100,Math.max(0,(App.engine.compressor.threshold.value - rmsDB)*5)) };
          document.getElementById('meter-input').style.height = data.input+'%';
          document.getElementById('meter-output').style.height = data.output+'%';
          document.getElementById('meter-gain').style.height = data.gainRed+'%';
          document.getElementById('meter-agc').style.height = (data.input*0.8)+'%';
          document.getElementById('meter-hf').style.height = (30+App.engine.params.hfBoost*2)+'%';
          document.getElementById('meter-stereo').style.height = App.engine.params.stereoWidth+'%';
          document.getElementById('meter-loudness-gr').style.height = data.gainRed+'%';
          document.getElementById('meter-limiter').style.height = (data.output>80?data.output-80:0)+'%';
          document.getElementById('meter-bass').style.height = (data.input*0.3)+'%';
          document.getElementById('meter-loudness').style.height = Math.max(0,Math.min(100,data.output-20))+'%';
          document.getElementById('latency-indicator').textContent = `LAT: ${(App.ctx.baseLatency*1000).toFixed(1)}ms`;
        };
        draw();
      }
    };

    // INIT
    document.addEventListener('DOMContentLoaded', () => App.init());
  </script>
</body>
</html>
