<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulador de Ataque SYN Flood</title>
    <style>
        :root {
            --bg-color: #f8fafc;
            --text-color: #1e293b;
            --primary: #2563eb;
            --danger: #dc2626;
            --success: #16a34a;
            --warning: #f97316;
            --border: #e2e8f0;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
        }

        .container {
            width: 100%;
            max-width: 900px;
            background: white;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.05);
            padding: 30px;
            box-sizing: border-box;
        }

        h1 {
            text-align: center;
            color: var(--primary);
            margin-top: 0;
            border-bottom: 2px solid var(--border);
            padding-bottom: 10px;
        }

        /* --- ZONA DE ANIMACIÓN --- */
        .animation-area {
            position: relative;
            height: 250px;
            background-color: #f1f5f9;
            border: 1px dashed #cbd5e1;
            border-radius: 8px;
            margin-bottom: 30px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 40px;
            overflow: hidden;
        }

        .node {
            display: flex;
            flex-direction: column;
            align-items: center;
            z-index: 10;
            background: white;
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }

        .attacker {
            border: 2px solid var(--danger);
        }

        .server {
            border: 2px solid var(--success);
            transition: all 0.3s ease;
            width: 140px;
            text-align: center;
        }

        .node-icon { font-size: 3rem; margin-bottom: 10px; }
        .node-title { font-weight: bold; font-size: 1.1rem; }

        /* Efecto de Caída (Servidor) */
        .server.flooded {
            background-color: #fef2f2;
            border-color: var(--danger);
            color: var(--danger);
        }
        
        .server.mitigated {
            background-color: #f0fdf4;
            border-color: var(--primary);
        }

        @keyframes shake {
            0% { transform: translate(1px, 1px) rotate(0deg); }
            20% { transform: translate(-2px, 0px) rotate(-1deg); }
            40% { transform: translate(1px, -1px) rotate(1deg); }
            60% { transform: translate(-1px, 2px) rotate(0deg); }
            80% { transform: translate(2px, -1px) rotate(1deg); }
            100% { transform: translate(0px, 0px) rotate(0deg); }
        }
        .shake { animation: shake 0.4s infinite; }

        /* Paquetes SYN */
        .packet {
            position: absolute;
            background-color: var(--danger);
            color: white;
            font-size: 0.7rem;
            font-weight: bold;
            padding: 4px 8px;
            border-radius: 12px;
            box-shadow: 0 2px 4px rgba(220, 38, 38, 0.4);
            animation: moveRight 1s linear forwards;
            pointer-events: none; /* Que no interfiera el click */
        }

        @keyframes moveRight {
            0% { left: 150px; top: calc(50% + var(--offset)); opacity: 0; transform: scale(0.5); }
            10% { opacity: 1; transform: scale(1); }
            90% { opacity: 1; transform: scale(1); }
            100% { left: calc(100% - 200px); top: calc(50% + var(--offset)); opacity: 0; transform: scale(0.5); }
        }

        /* --- ZONA DE CONTROLES --- */
        .controls-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin-bottom: 20px;
        }

        .control-group {
            background: #f8fafc;
            padding: 15px;
            border-radius: 8px;
            border: 1px solid var(--border);
        }

        .control-group label {
            display: block;
            font-weight: bold;
            margin-bottom: 10px;
            color: #475569;
        }

        input[type="range"] {
            width: 100%;
            cursor: pointer;
        }

        .buttons {
            display: flex;
            gap: 15px;
            margin-bottom: 30px;
        }

        button {
            flex: 1;
            padding: 12px;
            font-size: 1rem;
            font-weight: bold;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            transition: opacity 0.2s;
        }

        button:hover { opacity: 0.9; }

        .btn-attack { background-color: var(--danger); color: white; }
        .btn-attack.active { background-color: #991b1b; }
        .btn-mitigate { background-color: var(--primary); color: white; }

        /* --- ZONA DE ESTADO --- */
        .status-area {
            background: #f1f5f9;
            padding: 20px;
            border-radius: 8px;
            border: 1px solid var(--border);
        }

        .stats-header {
            display: flex;
            justify-content: space-between;
            font-weight: bold;
            margin-bottom: 10px;
            font-size: 1.1rem;
        }

        .progress-bg {
            background: #cbd5e1;
            height: 24px;
            border-radius: 12px;
            overflow: hidden;
            position: relative;
        }

        .progress-bar {
            height: 100%;
            width: 0%;
            background-color: var(--success);
            transition: width 0.2s ease, background-color 0.3s ease;
        }

        .status-text {
            text-align: center;
            font-size: 1.2rem;
            font-weight: bold;
            margin-top: 15px;
            color: var(--success);
        }

        @media (max-width: 600px) {
            .controls-grid { grid-template-columns: 1fr; }
            .buttons { flex-direction: column; }
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Simulador de Ataque TCP SYN Flood</h1>
    
    <!-- ZONA DE ANIMACIÓN VISUAL -->
    <div class="animation-area" id="animationArea">
        <div class="node attacker">
            <div class="node-icon">🦹‍♂️</div>
            <div class="node-title">Botnet Atacante</div>
        </div>
        
        <div class="node server" id="serverIcon">
            <div class="node-icon" id="serverEmoji">🖥️</div>
            <div class="node-title" id="serverTitle">Servidor OK</div>
        </div>
    </div>

    <!-- CONTROLES DESLIZANTES -->
    <div class="controls-grid">
        <div class="control-group">
            <label>Tasa de Ataque: <span id="rateVal">10</span> pps (paquetes/seg)</label>
            <input type="range" id="rateSlider" min="1" max="100" value="10">
        </div>
        <div class="control-group">
            <label>Tamaño Cola Backlog: <span id="backlogVal">100</span> espacios</label>
            <input type="range" id="backlogSlider" min="50" max="500" step="10" value="100">
        </div>
    </div>

    <!-- BOTONES DE ACCIÓN -->
    <div class="buttons">
        <button id="toggleBtn" class="btn-attack" onclick="toggleAttack()">Iniciar Ataque SYN Flood</button>
        <button class="btn-mitigate" onclick="mitigateAttack()">Mitigar Ataque (SYN Cookies)</button>
    </div>

    <!-- MÉTRICAS Y ESTADO DEL SERVIDOR -->
    <div class="status-area">
        <div class="stats-header">
            <span>Conexiones Pendientes (Cola):</span>
            <span><span id="currentBacklog">0</span> / <span id="maxBacklogDisplay">100</span></span>
        </div>
        <div class="progress-bg">
            <div class="progress-bar" id="progressBar"></div>
        </div>
        <div class="status-text" id="statusText">Normal (Operativo)</div>
    </div>
</div>

<script>
    // Variables de Estado
    let attackInterval = null;
    let recoveryInterval = null;
    let backlog = 0;
    let maxBacklog = 100;
    let isMitigating = false;

    // Elementos del DOM
    const rateSlider = document.getElementById('rateSlider');
    const backlogSlider = document.getElementById('backlogSlider');
    const rateVal = document.getElementById('rateVal');
    const backlogVal = document.getElementById('backlogVal');
    
    const animationArea = document.getElementById('animationArea');
    const serverIcon = document.getElementById('serverIcon');
    const serverEmoji = document.getElementById('serverEmoji');
    const serverTitle = document.getElementById('serverTitle');
    
    const currentBacklogEl = document.getElementById('currentBacklog');
    const maxBacklogDisplay = document.getElementById('maxBacklogDisplay');
    const progressBar = document.getElementById('progressBar');
    const statusText = document.getElementById('statusText');
    const toggleBtn = document.getElementById('toggleBtn');

    // Inicializar Valores
    maxBacklogDisplay.innerText = maxBacklog;

    // Listeners de los Sliders
    rateSlider.addEventListener('input', function() {
        rateVal.innerText = this.value;
        if (attackInterval) {
            // Reiniciar el intervalo con la nueva velocidad si ya está atacando
            clearInterval(attackInterval);
            attackInterval = setInterval(createPacket, 1000 / parseInt(this.value));
        }
    });

    backlogSlider.addEventListener('input', function() {
        maxBacklog = parseInt(this.value);
        backlogVal.innerText = maxBacklog;
        maxBacklogDisplay.innerText = maxBacklog;
        updateDashboard();
    });

    // Control de Inicio/Detención
    function toggleAttack() {
        if (attackInterval) {
            // Detener Ataque
            clearInterval(attackInterval);
            attackInterval = null;
            toggleBtn.innerText = "Iniciar Ataque SYN Flood";
            toggleBtn.classList.remove('active');
        } else {
            // Iniciar Ataque
            const rate = parseInt(rateSlider.value);
            attackInterval = setInterval(createPacket, 1000 / rate);
            toggleBtn.innerText = "Detener Ataque";
            toggleBtn.classList.add('active');
        }
    }

    // Crear y Animar un Paquete SYN
    function createPacket() {
        const packet = document.createElement('div');
        packet.classList.add('packet');
        packet.innerText = 'SYN';
        
        // Desviación vertical aleatoria para que no vayan todos en fila india
        const randomOffset = (Math.random() - 0.5) * 80; // +/- 40px
        packet.style.setProperty('--offset', randomOffset + 'px');

        animationArea.appendChild(packet);

        // Cuando la animación termine (1 seg), el paquete impacta el servidor
        setTimeout(() => {
            if (packet.parentNode) {
                packet.remove();
                if (!isMitigating) {
                    processHit();
                }
            }
        }, 1000);
    }

    // Procesar el impacto del paquete en el servidor
    function processHit() {
        if (backlog < maxBacklog) {
            backlog++;
        }
        updateDashboard();
    }

    // Botón de Mitigación (SYN Cookies / Flush)
    function mitigateAttack() {
        isMitigating = true;
        backlog = 0; // Vaciar la cola inmediatamente
        
        // Efecto visual de escudo protector
        serverIcon.classList.add('mitigated');
        serverEmoji.innerText = '🛡️';
        serverTitle.innerText = 'Mitigando...';
        
        updateDashboard();

        // El escudo dura 2.5 segundos simulando el filtrado activo
        setTimeout(() => {
            isMitigating = false;
            serverIcon.classList.remove('mitigated');
            updateDashboard(); // Restaura el icono normal
        }, 2500);
    }

    // Recuperación Natural: El servidor descarta conexiones que no se completan (Time-out)
    recoveryInterval = setInterval(() => {
        if (backlog > 0 && !isMitigating) {
            // Se limpian unas pocas conexiones cada medio segundo
            backlog = Math.max(0, backlog - Math.ceil(maxBacklog * 0.02)); 
            updateDashboard();
        }
    }, 500);

    // Actualizar Barras y Textos Visuales
    function updateDashboard() {
        currentBacklogEl.innerText = backlog;
        
        let percent = (backlog / maxBacklog) * 100;
        if (percent > 100) percent = 100;
        
        progressBar.style.width = percent + '%';

        if (percent >= 100) {
            // ESTADO CAÍDO
            progressBar.style.backgroundColor = 'var(--danger)';
            statusText.innerText = '⚠️ INUNDADO / DENEGACIÓN DE SERVICIO';
            statusText.style.color = 'var(--danger)';
            
            if (!isMitigating) {
                serverIcon.classList.add('shake', 'flooded');
                serverEmoji.innerText = '🔥';
                serverTitle.innerText = 'Servidor Caído';
            }
        } else if (percent >= 75) {
            // ESTADO BAJO ESTRÉS
            progressBar.style.backgroundColor = 'var(--warning)';
            statusText.innerText = 'Bajo Estrés (Saturación de Red)';
            statusText.style.color = 'var(--warning)';
            
            if (!isMitigating) {
                serverIcon.classList.remove('shake', 'flooded');
                serverEmoji.innerText = '⚠️';
                serverTitle.innerText = 'Servidor Lento';
            }
        } else {
            // ESTADO NORMAL
            progressBar.style.backgroundColor = 'var(--success)';
            statusText.innerText = 'Normal (Operativo)';
            statusText.style.color = 'var(--success)';
            
            if (!isMitigating) {
                serverIcon.classList.remove('shake', 'flooded');
                serverEmoji.innerText = '🖥️';
                serverTitle.innerText = 'Servidor OK';
            }
        }
    }
</script>
</body>
</html>
