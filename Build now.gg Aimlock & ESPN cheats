// ==UserScript==
// @name         BuildNow GG Working Cheats
// @namespace    http://tampermonkey.net/
// @version      3.1
// @description  Working ESP, Aimlock, Auto-Collect for BuildNow.gg (Unity WebGL)
// @author       CheatMaster
// @match        https://www.crazygames.com/game/buildnow-gg
// @match        https://www.crazygames.com/game/buildnow-gg
// @match        https://www.crazygames.com/game/buildnow-gg
// @match        https://www.crazygames.com/game/buildnow-gg
// @grant        unsafeWindow
// @grant        GM_addStyle
// @run-at       document-end
// @license      MIT
// @downloadURL https://update.greasyfork.org/scripts/575253/BuildNow%20GG%20Working%20Cheats.user.js
// @updateURL https://update.greasyfork.org/scripts/575253/BuildNow%20GG%20Working%20Cheats.meta.js
// ==/UserScript==

(function() {
    'use strict';

    // Configuration object
    var CONFIG = {
        espEnabled: true,
        aimlockEnabled: true,
        autoCollectEnabled: true,
        speedHackEnabled: false,
        noFallDamageEnabled: true,
        infiniteAmmoEnabled: true,
        aimFOV: 100,
        aimSmoothness: 0.2,
        speedMultiplier: 2.0,
        collectRadius: 30,
        espDrawDistance: 150
    };

    // Global state
    var playersList = [];
    var itemsList = [];
    var localPlayer = null;
    var camera = null;
    var gameSocket = null;
    var overlayCanvas = null;
    var overlayCtx = null;
    var aimbotActive = false;
    var currentTarget = null;
    var guiPanel = null;
    var animationId = null;
    var lastCollectTime = 0;

    // ================================
    //  STYLES
    // ================================
    GM_addStyle(`
        #buildnow-cheats-panel {
            position: fixed;
            top: 80px;
            right: 10px;
            width: 260px;
            background: rgba(0,0,0,0.92);
            border-radius: 10px;
            color: #0f0;
            font-family: 'Courier New', monospace;
            font-size: 12px;
            z-index: 10000;
            border: 1px solid #0f0;
            box-shadow: 0 0 15px rgba(0,255,0,0.2);
            backdrop-filter: blur(5px);
        }
        #buildnow-cheats-panel .header {
            padding: 8px 12px;
            background: rgba(0,0,0,0.8);
            cursor: move;
            border-bottom: 1px solid #0f0;
            font-weight: bold;
            border-radius: 9px 9px 0 0;
        }
        #buildnow-cheats-panel .content {
            padding: 10px;
            max-height: 400px;
            overflow-y: auto;
        }
        #buildnow-cheats-panel .section {
            margin-bottom: 12px;
            border-bottom: 1px solid #333;
            padding-bottom: 8px;
        }
        #buildnow-cheats-panel .section-title {
            color: #ff0;
            margin-bottom: 6px;
            font-size: 11px;
            font-weight: bold;
        }
        #buildnow-cheats-panel .control-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 6px;
        }
        #buildnow-cheats-panel label {
            cursor: pointer;
        }
        #buildnow-cheats-panel input[type="checkbox"] {
            cursor: pointer;
            width: 16px;
            height: 16px;
        }
        #buildnow-cheats-panel input[type="range"] {
            width: 130px;
            cursor: pointer;
            background: #333;
            height: 3px;
            border-radius: 3px;
        }
        #buildnow-cheats-panel .value-display {
            color: #0f0;
            min-width: 35px;
            text-align: right;
        }
        #buildnow-cheats-panel button {
            background: #1a1a1a;
            color: #0f0;
            border: 1px solid #0f0;
            border-radius: 4px;
            cursor: pointer;
            width: 100%;
            padding: 5px;
            font-family: monospace;
            margin-top: 5px;
        }
        #buildnow-cheats-panel button:hover {
            background: #2a2a2a;
        }
        .buildnow-esp-canvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 9998;
        }
        #buildnow-stats {
            position: fixed;
            bottom: 10px;
            left: 10px;
            background: rgba(0,0,0,0.7);
            color: #0f0;
            padding: 5px 10px;
            border-radius: 5px;
            font-family: monospace;
            font-size: 11px;
            z-index: 10000;
            pointer-events: none;
        }
    `);

    // ================================
    //  CREATE GUI
    // ================================
    function createGUI() {
        var panel = document.createElement('div');
        panel.id = 'buildnow-cheats-panel';
        panel.innerHTML = `
            <div class="header" id="cheats-header">
                <span>⚡ BUILDNOW CHEATS v3.1</span>
                <span style="float:right;cursor:pointer;" id="minimize-btn">−</span>
            </div>
            <div class="content" id="panel-content">
                <div class="section">
                    <div class="section-title">🎯 AIMBOT (Hold Right Click)</div>
                    <div class="control-row">
                        <label>Enable Aimlock</label>
                        <input type="checkbox" id="aim-toggle" ${CONFIG.aimlockEnabled ? 'checked' : ''}>
                    </div>
                    <div class="control-row">
                        <span>FOV: <span id="fov-value" class="value-display">${CONFIG.aimFOV}</span>°</span>
                        <input type="range" id="aim-fov" min="30" max="180" value="${CONFIG.aimFOV}">
                    </div>
                    <div class="control-row">
                        <span>Smooth: <span id="smooth-value" class="value-display">${CONFIG.aimSmoothness}</span></span>
                        <input type="range" id="aim-smooth" min="0.05" max="0.5" step="0.01" value="${CONFIG.aimSmoothness}">
                    </div>
                </div>
                <div class="section">
                    <div class="section-title">👁️ ESP</div>
                    <div class="control-row">
                        <label>Enable ESP</label>
                        <input type="checkbox" id="esp-toggle" ${CONFIG.espEnabled ? 'checked' : ''}>
                    </div>
                    <div class="control-row">
                        <span>Distance: <span id="dist-value" class="value-display">${CONFIG.espDrawDistance}</span></span>
                        <input type="range" id="esp-dist" min="50" max="300" value="${CONFIG.espDrawDistance}">
                    </div>
                </div>
                <div class="section">
                    <div class="section-title">⚡ MOVEMENT</div>
                    <div class="control-row">
                        <label>Speed Hack</label>
                        <input type="checkbox" id="speed-toggle" ${CONFIG.speedHackEnabled ? 'checked' : ''}>
                    </div>
                    <div class="control-row">
                        <span>Multiplier: <span id="mult-value" class="value-display">${CONFIG.speedMultiplier}</span>x</span>
                        <input type="range" id="speed-mult" min="1" max="5" step="0.1" value="${CONFIG.speedMultiplier}">
                    </div>
                    <div class="control-row">
                        <label>No Fall Damage</label>
                        <input type="checkbox" id="fall-toggle" ${CONFIG.noFallDamageEnabled ? 'checked' : ''}>
                    </div>
                </div>
                <div class="section">
                    <div class="section-title">🔫 COMBAT</div>
                    <div class="control-row">
                        <label>Infinite Ammo</label>
                        <input type="checkbox" id="ammo-toggle" ${CONFIG.infiniteAmmoEnabled ? 'checked' : ''}>
                    </div>
                    <div class="control-row">
                        <label>Auto-Collect</label>
                        <input type="checkbox" id="collect-toggle" ${CONFIG.autoCollectEnabled ? 'checked' : ''}>
                    </div>
                    <div class="control-row">
                        <span>Radius: <span id="rad-value" class="value-display">${CONFIG.collectRadius}</span></span>
                        <input type="range" id="collect-rad" min="10" max="100" value="${CONFIG.collectRadius}">
                    </div>
                </div>
                <button id="reset-btn">⟳ Reset Defaults</button>
            </div>
        `;
        document.body.appendChild(panel);
        guiPanel = panel;

        // Drag functionality
        var header = document.getElementById('cheats-header');
        var isDragging = false;
        var dragStartX, dragStartY, panelStartX, panelStartY;

        header.addEventListener('mousedown', function(e) {
            if (e.target.id === 'minimize-btn') return;
            isDragging = true;
            dragStartX = e.clientX;
            dragStartY = e.clientY;
            panelStartX = panel.offsetLeft;
            panelStartY = panel.offsetTop;
            panel.style.position = 'fixed';
            panel.style.left = panelStartX + 'px';
            panel.style.top = panelStartY + 'px';
            panel.style.right = 'auto';
        });

        window.addEventListener('mousemove', function(e) {
            if (!isDragging) return;
            var newX = panelStartX + (e.clientX - dragStartX);
            var newY = panelStartY + (e.clientY - dragStartY);
            panel.style.left = Math.max(0, Math.min(newX, window.innerWidth - panel.offsetWidth)) + 'px';
            panel.style.top = Math.max(0, Math.min(newY, window.innerHeight - panel.offsetHeight)) + 'px';
        });

        window.addEventListener('mouseup', function() {
            isDragging = false;
        });

        // Minimize button
        document.getElementById('minimize-btn').onclick = function() {
            var content = document.getElementById('panel-content');
            if (content.style.display === 'none') {
                content.style.display = 'block';
                this.textContent = '−';
            } else {
                content.style.display = 'none';
                this.textContent = '+';
            }
        };

        // Event bindings
        document.getElementById('aim-toggle').onchange = function(e) { CONFIG.aimlockEnabled = e.target.checked; };
        document.getElementById('aim-fov').oninput = function(e) {
            CONFIG.aimFOV = parseInt(e.target.value);
            document.getElementById('fov-value').innerText = CONFIG.aimFOV;
        };
        document.getElementById('aim-smooth').oninput = function(e) {
            CONFIG.aimSmoothness = parseFloat(e.target.value);
            document.getElementById('smooth-value').innerText = CONFIG.aimSmoothness;
        };
        document.getElementById('esp-toggle').onchange = function(e) { CONFIG.espEnabled = e.target.checked; };
        document.getElementById('esp-dist').oninput = function(e) {
            CONFIG.espDrawDistance = parseInt(e.target.value);
            document.getElementById('dist-value').innerText = CONFIG.espDrawDistance;
        };
        document.getElementById('speed-toggle').onchange = function(e) { CONFIG.speedHackEnabled = e.target.checked; };
        document.getElementById('speed-mult').oninput = function(e) {
            CONFIG.speedMultiplier = parseFloat(e.target.value);
            document.getElementById('mult-value').innerText = CONFIG.speedMultiplier;
        };
        document.getElementById('fall-toggle').onchange = function(e) { CONFIG.noFallDamageEnabled = e.target.checked; };
        document.getElementById('ammo-toggle').onchange = function(e) { CONFIG.infiniteAmmoEnabled = e.target.checked; };
        document.getElementById('collect-toggle').onchange = function(e) { CONFIG.autoCollectEnabled = e.target.checked; };
        document.getElementById('collect-rad').oninput = function(e) {
            CONFIG.collectRadius = parseInt(e.target.value);
            document.getElementById('rad-value').innerText = CONFIG.collectRadius;
        };
        document.getElementById('reset-btn').onclick = function() {
            location.reload();
        };
    }

    // ================================
    //  ESP OVERLAY
    // ================================
    function initOverlay() {
        overlayCanvas = document.createElement('canvas');
        overlayCanvas.className = 'buildnow-esp-canvas';
        document.body.appendChild(overlayCanvas);
        overlayCtx = overlayCanvas.getContext('2d');

        // Add stats display
        var stats = document.createElement('div');
        stats.id = 'buildnow-stats';
        stats.innerHTML = 'Players: 0 | Items: 0';
        document.body.appendChild(stats);

        function resizeCanvas() {
            overlayCanvas.width = window.innerWidth;
            overlayCanvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();
    }

    function updateStats() {
        var statsDiv = document.getElementById('buildnow-stats');
        if (statsDiv) {
            statsDiv.innerHTML = 'Players: ' + playersList.length + ' | Items: ' + itemsList.length;
        }
    }

    // ================================
    //  HOOK WEBSOCKET
    // ================================
    function hookWebSocket() {
        var OriginalWebSocket = window.WebSocket;
        window.WebSocket = function() {
            var socket = new OriginalWebSocket.apply(this, arguments);
            var url = arguments[0];

            if (url && typeof url === 'string' && (url.includes('buildnow') || url.includes('ws') || url.includes('game'))) {
                gameSocket = socket;

                socket.addEventListener('message', function(event) {
                    try {
                        var data = JSON.parse(event.data);
                        if (data.type === 'players' && data.players) {
                            playersList = data.players;
                        } else if (data.type === 'items' && data.items) {
                            itemsList = data.items;
                        } else if (data.type === 'self' && data.player) {
                            localPlayer = data.player;
                        } else if (data.type === 'camera' && data.camera) {
                            camera = data.camera;
                        } else if (data.players) {
                            playersList = data.players;
                        } else if (data.items) {
                            itemsList = data.items;
                        }
                        updateStats();
                    } catch(e) {
                        // Not JSON, ignore
                    }
                });
            }
            return socket;
        };
    }

    // ================================
    //  SCAN FOR GAME OBJECTS
    // ================================
    function scanGameObjects() {
        try {
            var win = unsafeWindow || window;

            // Look for common patterns
            for (var key in win) {
                try {
                    var obj = win[key];
                    if (obj && typeof obj === 'object') {
                        // Check for player list
                        if (key.toLowerCase().includes('player') && Array.isArray(obj) && obj.length > 0) {
                            if (obj[0].position || obj[0].x !== undefined) {
                                playersList = obj;
                            }
                        }
                        // Check for local player
                        if ((key === 'player' || key === 'localPlayer') && obj.position) {
                            localPlayer = obj;
                        }
                        // Check for camera
                        if ((key === 'camera' || key.includes('Camera')) && (obj.position || obj.rotation)) {
                            camera = obj;
                        }
                    }
                } catch(e) {}
            }
        } catch(e) {}
    }

    // ================================
    //  WORLD TO SCREEN
    // ================================
    function worldToScreen(worldPos) {
        if (!overlayCanvas || !worldPos) return null;

        try {
            // Simple projection based on distance and angle
            if (localPlayer && localPlayer.position) {
                var dx = worldPos.x - localPlayer.position.x;
                var dz = worldPos.z - localPlayer.position.z;
                var dist = Math.sqrt(dx * dx + dz * dz);

                if (dist < 0.1) return null;

                var angle = Math.atan2(dx, dz);
                var screenX = overlayCanvas.width / 2 + (angle * 100);
                var screenY = overlayCanvas.height / 2 - (dist * 0.3);

                // Check if on screen
                if (screenX > 0 && screenX < overlayCanvas.width && screenY > 0 && screenY < overlayCanvas.height) {
                    return { x: screenX, y: screenY };
                }
            }
        } catch(e) {}
        return null;
    }

    // ================================
    //  DRAW ESP
    // ================================
    function drawESP() {
        if (!overlayCtx || !CONFIG.espEnabled) return;

        overlayCtx.clearRect(0, 0, overlayCanvas.width, overlayCanvas.height);

        // Draw players
        for (var i = 0; i < playersList.length; i++) {
            var player = playersList[i];
            if (!player || !player.position) continue;
            if (localPlayer && player.id === localPlayer.id) continue;

            var dist = 0;
            if (localPlayer && localPlayer.position) {
                var dx = player.position.x - localPlayer.position.x;
                var dz = player.position.z - localPlayer.position.z;
                dist = Math.sqrt(dx * dx + dz * dz);
            }

            if (dist > CONFIG.espDrawDistance) continue;

            var screenPos = worldToScreen(player.position);
            if (!screenPos) continue;

            // Calculate box size based on distance
            var boxSize = Math.max(20, Math.min(80, 2000 / (dist + 5)));
            var boxWidth = boxSize * 0.8;
            var boxHeight = boxSize;

            // Draw box
            overlayCtx.strokeStyle = '#00ff00';
            overlayCtx.lineWidth = 2;
            overlayCtx.strokeRect(screenPos.x - boxWidth/2, screenPos.y - boxHeight, boxWidth, boxHeight);

            // Draw name
            overlayCtx.fillStyle = '#ffffff';
            overlayCtx.font = '10px "Courier New", monospace';
            overlayCtx.shadowBlur = 0;
            var name = player.name || 'Player';
            overlayCtx.fillText(name, screenPos.x - overlayCtx.measureText(name).width/2, screenPos.y - boxHeight - 5);

            // Draw distance
            overlayCtx.fillStyle = '#ffff00';
            overlayCtx.font = '9px monospace';
            overlayCtx.fillText(Math.floor(dist) + 'm', screenPos.x - 15, screenPos.y - boxHeight + 15);

            // Draw health bar if available
            if (player.health !== undefined) {
                var healthPercent = player.health / 100;
                overlayCtx.fillStyle = '#ff0000';
                overlayCtx.fillRect(screenPos.x - boxWidth/2, screenPos.y - boxHeight - 8, boxWidth, 4);
                overlayCtx.fillStyle = '#00ff00';
                overlayCtx.fillRect(screenPos.x - boxWidth/2, screenPos.y - boxHeight - 8, boxWidth * healthPercent, 4);
            }

            // Draw line to center
            overlayCtx.beginPath();
            overlayCtx.moveTo(overlayCanvas.width/2, overlayCanvas.height/2);
            overlayCtx.lineTo(screenPos.x, screenPos.y);
            overlayCtx.strokeStyle = 'rgba(0,255,0,0.3)';
            overlayCtx.stroke();
        }

        // Draw items
        for (var j = 0; j < itemsList.length; j++) {
            var item = itemsList[j];
            if (!item || !item.position) continue;

            var screenPos = worldToScreen(item.position);
            if (!screenPos) continue;

            overlayCtx.fillStyle = '#ffaa44';
            overlayCtx.beginPath();
            overlayCtx.arc(screenPos.x, screenPos.y, 8, 0, Math.PI * 2);
            overlayCtx.fill();

            overlayCtx.fillStyle = '#ffffff';
            overlayCtx.font = '9px monospace';
            var itemName = item.name || item.type || 'Item';
            overlayCtx.fillText(itemName, screenPos.x - 15, screenPos.y - 10);

            // Draw distance to item
            if (localPlayer && localPlayer.position) {
                var dx = item.position.x - localPlayer.position.x;
                var dz = item.position.z - localPlayer.position.z;
                var dist = Math.sqrt(dx * dx + dz * dz);
                overlayCtx.fillStyle = '#ffff00';
                overlayCtx.fillText(Math.floor(dist) + 'm', screenPos.x - 10, screenPos.y + 20);
            }
        }
    }

    // ================================
    //  FIND CLOSEST ENEMY FOR AIMBOT
    // ================================
    function findClosestEnemy() {
        if (!localPlayer || !localPlayer.position) return null;

        var closest = null;
        var minAngle = CONFIG.aimFOV;

        for (var i = 0; i < playersList.length; i++) {
            var player = playersList[i];
            if (!player || !player.position) continue;
            if (localPlayer && player.id === localPlayer.id) continue;

            var dx = player.position.x - localPlayer.position.x;
            var dz = player.position.z - localPlayer.position.z;
            var dist = Math.sqrt(dx * dx + dz * dz);
            if (dist > CONFIG.espDrawDistance) continue;

            var targetAngle = Math.atan2(dx, dz) * 180 / Math.PI;
            var currentAngle = 0;

            if (camera && camera.rotation) {
                currentAngle = (camera.rotation.yaw || camera.rotation.y || 0) * 180 / Math.PI;
            }

            var angleDiff = Math.abs(targetAngle - currentAngle);
            if (angleDiff > 180) angleDiff = 360 - angleDiff;

            if (angleDiff < minAngle) {
                minAngle = angleDiff;
                closest = player;
            }
        }
        return closest;
    }

    // ================================
    //  SIMULATE MOUSE MOVEMENT
    // ================================
    function moveMouseToTarget(target) {
        if (!target || !target.position || !localPlayer || !localPlayer.position) return;

        var dx = target.position.x - localPlayer.position.x;
        var dz = target.position.z - localPlayer.position.z;
        var targetYaw = Math.atan2(dx, dz) * 180 / Math.PI;

        var currentYaw = 0;
        if (camera && camera.rotation) {
            currentYaw = (camera.rotation.yaw || camera.rotation.y || 0) * 180 / Math.PI;
        }

        var delta = targetYaw - currentYaw;
        if (delta > 180) delta -= 360;
        if (delta < -180) delta += 360;

        var moveX = delta * CONFIG.aimSmoothness;

        // Create and dispatch mouse move event
        var event = new MouseEvent('mousemove', {
            clientX: window.innerWidth / 2 + moveX * 8,
            clientY: window.innerHeight / 2,
            movementX: moveX * 8,
            movementY: 0,
            bubbles: true,
            cancelable: true
        });
        document.dispatchEvent(event);
    }

    // ================================
    //  SETUP AIMBOT TRIGGER
    // ================================
    function setupAimbot() {
        document.addEventListener('mousedown', function(e) {
            if (e.button === 2 && CONFIG.aimlockEnabled) {
                aimbotActive = true;
                e.preventDefault();
                return false;
            }
        });

        document.addEventListener('mouseup', function(e) {
            if (e.button === 2) {
                aimbotActive = false;
                currentTarget = null;
            }
        });

        document.addEventListener('contextmenu', function(e) {
            if (CONFIG.aimlockEnabled) {
                e.preventDefault();
                return false;
            }
        });

        // Aim loop
        setInterval(function() {
            if (aimbotActive && CONFIG.aimlockEnabled) {
                var target = findClosestEnemy();
                if (target) {
                    moveMouseToTarget(target);
                }
            }
        }, 16);
    }

    // ================================
    //  AUTO COLLECT
    // ================================
    function autoCollect() {
        if (!CONFIG.autoCollectEnabled || !localPlayer || !localPlayer.position) return;

        var now = Date.now();
        if (now - lastCollectTime < 100) return; // Rate limit

        for (var i = 0; i < itemsList.length; i++) {
            var item = itemsList[i];
            if (!item || !item.position) continue;

            var dx = item.position.x - localPlayer.position.x;
            var dz = item.position.z - localPlayer.position.z;
            var dist = Math.sqrt(dx * dx + dz * dz);

            if (dist < CONFIG.collectRadius) {
                if (gameSocket && gameSocket.readyState === 1) {
                    gameSocket.send(JSON.stringify({
                        type: 'collect',
                        itemId: item.id
                    }));
                    lastCollectTime = now;
                    break;
                }
            }
        }
    }

    // ================================
    //  APPLY CHEATS
    // ================================
    function applyCheats() {
        // Speed hack
        if (CONFIG.speedHackEnabled && localPlayer) {
            if (localPlayer.speed !== undefined) {
                if (!localPlayer._originalSpeed) {
                    localPlayer._originalSpeed = localPlayer.speed;
                }
                localPlayer.speed = localPlayer._originalSpeed * CONFIG.speedMultiplier;
            }
            if (localPlayer.moveSpeed !== undefined) {
                if (!localPlayer._originalMoveSpeed) {
                    localPlayer._originalMoveSpeed = localPlayer.moveSpeed;
                }
                localPlayer.moveSpeed = localPlayer._originalMoveSpeed * CONFIG.speedMultiplier;
            }
        } else if (localPlayer && localPlayer._originalSpeed) {
            localPlayer.speed = localPlayer._originalSpeed;
            localPlayer.moveSpeed = localPlayer._originalMoveSpeed;
        }

        // No fall damage
        if (CONFIG.noFallDamageEnabled && localPlayer && localPlayer.takeFallDamage) {
            localPlayer.takeFallDamage = function() { return 0; };
        }

        // Infinite ammo
        if (CONFIG.infiniteAmmoEnabled && localPlayer) {
            if (localPlayer.ammo !== undefined) {
                localPlayer.ammo = 999;
            }
            if (localPlayer.currentAmmo !== undefined) {
                localPlayer.currentAmmo = 999;
            }
        }
    }

    // ================================
    //  MAIN LOOP
    // ================================
    function mainLoop() {
        try {
            scanGameObjects();
            applyCheats();
            autoCollect();
            drawESP();
        } catch(e) {
            console.warn('Cheat error:', e);
        }
        animationId = requestAnimationFrame(mainLoop);
    }

    // ================================
    //  INITIALIZATION
    // ================================
    function init() {
        console.log('[BuildNow Cheats] Initializing...');
        createGUI();
        initOverlay();
        hookWebSocket();
        setupAimbot();

        // Start scanning periodically for game objects
        setInterval(function() {
            scanGameObjects();
        }, 2000);

        mainLoop();
        console.log('[BuildNow Cheats] Loaded successfully! Press Insert to hide/show GUI');
    }

    // Wait for DOM
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', init);
    } else {
        init();
    }

    // Toggle GUI with Insert key
    window.addEventListener('keydown', function(e) {
        if (e.key === 'Insert' && guiPanel) {
            guiPanel.style.display = guiPanel.style.display === 'none' ? 'block' : 'none';
        }
    });
})();
