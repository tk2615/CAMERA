<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>SURVEILLANCE GRID // V17 TRANSITION NOISE</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=VT323&family=Share+Tech+Mono&display=swap');

        :root {
            --bg-main: #000;
            --accent-color: #ff2222; 
            --bar-height: 50px; 
        }

        html, body {
            width: 100%; height: 100%; 
            margin: 0; padding: 0;
            background-color: var(--bg-main); color: #ddd;
            font-family: 'Share Tech Mono', monospace;
            overflow: hidden;
        }

        #monitor-wall {
            position: absolute; top: 0; left: 0;
            width: 100vw; height: calc(100vh - var(--bar-height));
            background: #000; 
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            grid-template-rows: repeat(3, 1fr);
            gap: 2px; box-sizing: border-box; z-index: 1;
        }

        .monitor {
            position: relative; background: #000;
            overflow: hidden; width: 100%; height: 100%;
        }

        @media (max-width: 768px) {
            #monitor-wall {
                position: relative; 
                height: calc(100% - var(--bar-height));
                grid-template-columns: repeat(2, 1fr);
                grid-template-rows: repeat(5, 1fr);
                overflow-y: auto; gap: 2px;
            }
        }

        /* --- 砂嵐 (noise.gif) --- */
        .static-noise {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            z-index: 0;
            background-image: url('noise.gif');
            background-size: cover; background-position: center; background-repeat: no-repeat;
            opacity: 0.6; 
            transition: opacity 0.1s; /* 切り替わりを少しキビキビさせる */
        }
        .monitor.is-error .static-noise { opacity: 1 !important; filter: contrast(1.2) brightness(1.1); }

        /* iframe制御 */
        .monitor iframe {
            width: 100%; height: 100%; border: none;
            transform: scale(1.6); transform-origin: center center;
            pointer-events: none;
            opacity: 0; /* デフォルトは透明（GIFが見える） */
            transition: opacity 0.5s ease-in; /* 映像が出る時はフワッと */
            position: relative; z-index: 2;
            filter: contrast(1.1) sepia(0.1) saturate(1.2);
        }
        /* クラスがついている時だけ表示 */
        .monitor iframe.is-playing { opacity: 1; }

        .monitor::before {
            content: ""; display: block; position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(rgba(0,0,0,0) 50%, rgba(0,0,0,0.3) 50%);
            z-index: 5; background-size: 100% 4px; pointer-events: none;
        }

        /* UIパーツ */
        .overlay-top-left {
            position: absolute; top: 8px; left: 8px; z-index: 10;
            font-family: 'VT323', monospace; font-size: 14px;
            text-shadow: 1px 1px 2px #000; pointer-events: none;
            background: rgba(0,0,0,0.5); padding: 2px 4px;
        }
        .overlay-bottom {
            position: absolute; bottom: 0; left: 0; width: 100%; z-index: 10;
            background: linear-gradient(to top, rgba(0,0,0,0.9), transparent);
            padding: 20px 8px 5px 8px; box-sizing: border-box;
            display: flex; justify-content: space-between; align-items: flex-end;
            pointer-events: none;
        }
        .video-title {
            font-family: 'Share Tech Mono', monospace; font-size: 11px;
            color: #ddd; text-transform: uppercase;
            white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
            max-width: 60%; opacity: 0.9; text-shadow: 1px 1px 1px #000;
        }
        .monitor.is-error .video-title { color: var(--accent-color); animation: blink 0.5s infinite alternate; }

        .status-badge { font-family: 'VT323', monospace; font-size: 16px; text-shadow: 1px 1px 2px #000; }
        .rec-indicator { color: var(--accent-color); font-weight: bold; }
        .rec-dot {
            display: inline-block; width: 8px; height: 8px; 
            background: var(--accent-color); border-radius: 50%; 
            animation: blink 1s steps(2, start) infinite; margin-right: 5px;
            box-shadow: 0 0 5px var(--accent-color);
        }

        #control-bar {
            position: absolute; bottom: 0; left: 0; width: 100%; height: var(--bar-height);
            background: #050505; border-top: 1px solid #333;
            display: flex; align-items: center; padding: 0 10px; gap: 10px; 
            font-family: 'VT323', monospace; z-index: 100;
            white-space: nowrap; overflow-x: auto; box-sizing: border-box;
        }
        
        #playlist-input { display: none; } 
        .playlist-label { font-size: 14px; color: var(--accent-color); border: 1px solid var(--accent-color); padding: 2px 6px; }

        button {
            background: #222; color: #ddd; border: 1px solid #444;
            padding: 4px 12px; cursor: pointer; font-family: inherit; text-transform: uppercase; font-size: 16px;
        }
        button:hover { border-color: var(--accent-color); color: var(--accent-color); background: #333; }
        .system-status { font-size: 14px; color: #666; margin-left: auto; text-align: right; display: none; }
        @media (min-width: 769px) { .system-status { display: block; } }

        @keyframes blink { 0% { opacity: 1; } 100% { opacity: 0.3; } }
        
        h1:first-of-type { display: none !important; }
    </style>
</head>
<body>

    <div id="monitor-wall"></div>

    <div id="control-bar">
        <textarea id="playlist-input">
PLxtg5zfgORZr8KB1VglBvI6czMJpPL-rx, 
https://www.youtube.com/watch?v=tujkoXI8rWM, 
https://www.youtube.com/watch?v=mCBV2OpKKXQ, 
https://www.youtube.com/watch?v=LnrmP3Z1M-s, 
https://www.youtube.com/watch?v=VM18f-IIUTw
        </textarea>
        
        <div class="playlist-label">NOISE_LINK: ON</div>
        <button onclick="forceCycle()">SKIP</button>
        <button onclick="location.reload()">REBOOT</button>
        <div class="system-status" id="status-display">STANDBY</div>
    </div>

    <script>
        var tag = document.createElement('script');
        tag.src = "https://www.youtube.com/iframe_api";
        var firstScriptTag = document.getElementsByTagName('script')[0];
        firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);

        let players = [];
        const gridCount = 9;
        let channelPool = [];
        let activeSources = new Array(gridCount).fill(null);

        function onYouTubeIframeAPIReady() {
            setTimeout(createGrid, 1000);
        }

        function buildChannelPool() {
            const raw = document.getElementById('playlist-input').value;
            const parts = raw.split(/[\n, ]+/);
            channelPool = [];
            let playlistId = "";

            parts.forEach(part => {
                const cleanPart = part.trim();
                if(cleanPart.length < 5) return;
                if (cleanPart.includes("list=") || cleanPart.startsWith("PL")) {
                    const match = cleanPart.match(/[?&]list=([^#\&\?]+)/);
                    if (match) playlistId = match[1];
                    else if (cleanPart.startsWith("PL")) playlistId = cleanPart;
                } else {
                    let id = "";
                    const matchV = cleanPart.match(/[?&]v=([^#\&\?]+)/);
                    if (matchV) id = matchV[1];
                    else if (cleanPart.includes("youtu.be/")) id = cleanPart.split("youtu.be/")[1];
                    else id = cleanPart;
                    if(id) channelPool.push({ type: 'video', id: id, uid: `vid_${id}` });
                }
            });

            if (playlistId) {
                for (let i = 0; i < 50; i++) {
                    channelPool.push({ type: 'playlist_slice', id: playlistId, index: i, uid: `pl_${playlistId}_${i}` });
                }
            }
        }

        function pickRandomSource() {
            const currentUIDs = activeSources.map(s => s ? s.uid : null);
            const available = channelPool.filter(ch => !currentUIDs.includes(ch.uid));
            const candidates = available.length > 0 ? available : channelPool;
            return candidates[Math.floor(Math.random() * candidates.length)];
        }

        function createGrid() {
            buildChannelPool();
            const wall = document.getElementById('monitor-wall');
            wall.innerHTML = '';
            players = [];

            for (let i = 0; i < gridCount; i++) {
                const source = pickRandomSource();
                activeSources[i] = source;

                const camId = (i + 1).toString().padStart(2, '0');
                const div = document.createElement('div');
                div.className = 'monitor';
                div.id = `monitor-${i}`;
                div.innerHTML = `
                    <div class="static-noise"></div>
                    <div class="overlay-top-left">
                        <span class="rec-indicator"><span class="rec-dot"></span>REC</span> CAM-${camId}
                    </div>
                    <div class="overlay-bottom">
                        <div class="video-title" id="title-${i}">CONNECTING...</div>
                        <div class="status-badge" id="status-${i}">[WAIT]</div>
                    </div>
                    <div id="player-${i}"></div>
                `;
                wall.appendChild(div);

                let videoConfig = {
                    height: '100%', width: '100%',
                    playerVars: { 'playsinline': 1, 'controls': 0, 'showinfo': 0, 'mute': 1, 'autoplay': 1, 'iv_load_policy': 3 },
                    events: {
                        'onReady': (event) => { 
                            event.target.playVideo(); 
                            startAutoSwitch(i);
                        },
                        'onStateChange': (event) => onPlayerStateChange(event, i),
                        'onError': (event) => onPlayerError(event, i)
                    }
                };

                if (source.type === 'playlist_slice') {
                    videoConfig.playerVars.listType = 'playlist';
                    videoConfig.playerVars.list = source.id;
                    videoConfig.playerVars.index = source.index;
                    videoConfig.playerVars.loop = 1;
                } else {
                    videoConfig.videoId = source.id;
                    videoConfig.playerVars.loop = 1;
                    videoConfig.playerVars.playlist = source.id;
                }

                players[i] = new YT.Player(`player-${i}`, videoConfig);
            }
            updateSysStatus();
        }

        function startAutoSwitch(index) {
            const delay = (Math.floor(Math.random() * 60) + 30) * 1000;
            setTimeout(() => {
                const monitor = document.getElementById(`monitor-${index}`);
                if (!monitor.classList.contains('is-error')) {
                    changeChannel(index);
                }
                startAutoSwitch(index);
            }, delay);
        }

        function changeChannel(index) {
            const player = players[index];
            const iframe = document.getElementById(`player-${index}`);
            const statusLabel = document.getElementById(`status-${index}`);
            const titleLabel = document.getElementById(`title-${index}`);

            if (!player || typeof player.loadPlaylist !== 'function') return;

            // 1. まず現在の映像を消す（これで後ろのGIFが見える）
            iframe.classList.remove('is-playing');
            statusLabel.innerText = "[TUNE]"; // チューニング中
            statusLabel.style.color = "";
            titleLabel.innerText = "SCANNING...";

            // 2. 新しいソースを選ぶ
            const newSource = pickRandomSource();
            activeSources[index] = newSource;

            // 3. プレイヤーにロード命令（ロードには1〜2秒かかるので、その間GIFが見え続ける）
            if (newSource.type === 'playlist_slice') {
                player.loadPlaylist({ list: newSource.id, listType: 'playlist', index: newSource.index, startSeconds: 0 });
            } else {
                player.loadVideoById(newSource.id);
            }
        }

        function onPlayerStateChange(event, index) {
            const monitor = document.getElementById(`monitor-${index}`);
            const iframe = document.getElementById(`player-${index}`);
            const statusLabel = document.getElementById(`status-${index}`);
            const titleLabel = document.getElementById(`title-${index}`);
            
            if (event.data === YT.PlayerState.PLAYING) {
                // 再生開始：エラー状態解除＆映像表示（GIFを隠す）
                monitor.classList.remove('is-error'); 
                iframe.classList.add('is-playing');
                statusLabel.innerText = "[LIVE]";
                statusLabel.style.color = "var(--accent-color)";
                statusLabel.style.textShadow = "0 0 5px var(--accent-color)";
                let videoData = event.target.getVideoData();
                titleLabel.innerText = (videoData && videoData.title) ? videoData.title : "SIGNAL LOCKED";
            } else if (event.data === YT.PlayerState.ENDED) {
                changeChannel(index);
            }
        }

        function onPlayerError(event, index) {
            const monitor = document.getElementById(`monitor-${index}`);
            const iframe = document.getElementById(`player-${index}`);
            const statusLabel = document.getElementById(`status-${index}`);
            const titleLabel = document.getElementById(`title-${index}`);
            
            iframe.classList.remove('is-playing'); // 映像消去（GIF表示）
            monitor.classList.add('is-error');     // エラー強調
            statusLabel.innerText = "[ERR]";
            statusLabel.style.color = "var(--accent-color)";
            titleLabel.innerText = "SIGNAL LOST";
        }

        function forceCycle() {
            for(let i=0; i<gridCount; i++) {
                const monitor = document.getElementById(`monitor-${i}`);
                if (!monitor.classList.contains('is-error')) {
                    changeChannel(i);
                }
            }
        }

        function updateSysStatus() {
            const now = new Date().toLocaleTimeString();
            document.getElementById('status-display').innerHTML = `TIME: ${now} | POOL: ${channelPool.length} CH`;
        }
        setInterval(updateSysStatus, 1000);
    </script>
</body>
</html>
