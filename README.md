<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>現實人生副本模擬器 - Real-Life Life Simulator (AI Enhanced)</title>
  <style>
    :root {
      --bg-color: #000000;
      --card-bg: rgba(18, 22, 32, 0.75);
      --card-border: rgba(0, 240, 255, 0.2);
      --primary: #00f0ff;
      --secondary: #7000ff;
      --accent: #ff0055;
      --text: #e0e6ed;
      --text-muted: #8a99ad;
      --gold: #ffd700;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
    }

    body {
      background-color: var(--bg-color);
      color: var(--text);
      overflow-x: hidden;
      min-height: 100vh;
      position: relative;
    }

    /* 全螢幕 3D Canvas 背景 */
    #earth-canvas {
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      z-index: 1;
      pointer-events: none;
    }

    /* 頂部 API Key 工具列 */
    .api-key-bar {
      position: fixed;
      top: 10px;
      right: 15px;
      z-index: 1000;
      display: flex;
      align-items: center;
      gap: 8px;
      background: rgba(0, 0, 0, 0.8);
      border: 1px solid var(--primary);
      padding: 6px 12px;
      border-radius: 20px;
      backdrop-filter: blur(10px);
    }

    .api-key-bar label {
      font-size: 0.8rem;
      color: var(--primary);
      font-weight: bold;
    }

    .api-key-bar input {
      background: rgba(255, 255, 255, 0.1);
      border: 1px solid rgba(0, 240, 255, 0.3);
      color: #fff;
      padding: 4px 8px;
      border-radius: 12px;
      font-size: 0.8rem;
      width: 140px;
      outline: none;
    }

    .api-key-bar button {
      background: var(--primary);
      color: #000;
      border: none;
      padding: 4px 10px;
      border-radius: 12px;
      font-weight: bold;
      font-size: 0.8rem;
      cursor: pointer;
    }

    /* 應用程式主容器 */
    #app-container {
      position: relative;
      z-index: 10;
      max-width: 900px;
      margin: 0 auto;
      padding: 20px;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      justify-content: center;
    }

    .view-card {
      background: var(--card-bg);
      border: 1px solid var(--card-border);
      backdrop-filter: blur(16px);
      border-radius: 20px;
      padding: 30px;
      box-shadow: 0 10px 30px rgba(0, 240, 255, 0.1);
      display: none;
      animation: fadeIn 0.5s ease-in-out forwards;
    }

    .view-card.active {
      display: block;
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: translateY(0); }
    }

    h1, h2, h3 {
      text-align: center;
      background: linear-gradient(135deg, var(--primary), var(--secondary));
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      margin-bottom: 20px;
    }

    /* 職業彈幕牆區塊 */
    .danmaku-container {
      height: 180px;
      overflow: hidden;
      position: relative;
      margin: 20px 0;
      border-radius: 12px;
      background: rgba(0, 0, 0, 0.5);
      border: 1px dashed rgba(0, 240, 255, 0.2);
    }

    .danmaku-track {
      display: flex;
      white-space: nowrap;
      position: absolute;
      animation: marquee 25s linear infinite;
    }

    .danmaku-track.reverse {
      animation: marqueeReverse 30s linear infinite;
      top: 60px;
    }

    .danmaku-track.third {
      animation: marquee 22s linear infinite;
      top: 120px;
    }

    @keyframes marquee {
      0% { transform: translateX(0); }
      100% { transform: translateX(-50%); }
    }

    @keyframes marqueeReverse {
      0% { transform: translateX(-50%); }
      100% { transform: translateX(0); }
    }

    .danmaku-item {
      display: inline-block;
      background: rgba(0, 240, 255, 0.1);
      border: 1px solid var(--primary);
      color: #fff;
      padding: 6px 16px;
      margin: 0 8px;
      border-radius: 20px;
      cursor: pointer;
      transition: all 0.2s;
    }

    .danmaku-item:hover {
      background: var(--primary);
      color: #000;
      transform: scale(1.1);
      box-shadow: 0 0 15px var(--primary);
    }

    .input-group {
      display: flex;
      gap: 10px;
      margin-top: 15px;
    }

    input[type="text"] {
      flex: 1;
      background: rgba(0, 0, 0, 0.6);
      border: 1px solid var(--card-border);
      padding: 12px 16px;
      color: #fff;
      border-radius: 10px;
      font-size: 1rem;
      outline: none;
    }

    input[type="text"]:focus {
      border-color: var(--primary);
      box-shadow: 0 0 8px var(--primary);
    }

    .btn {
      background: linear-gradient(135deg, var(--primary), var(--secondary));
      color: #fff;
      border: none;
      padding: 12px 24px;
      border-radius: 10px;
      font-weight: bold;
      cursor: pointer;
      transition: all 0.3s;
      display: inline-block;
      text-align: center;
    }

    .btn:hover {
      opacity: 0.9;
      transform: translateY(-2px);
      box-shadow: 0 5px 15px rgba(0, 240, 255, 0.4);
    }

    /* 性格選擇標籤 */
    .trait-cloud {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      justify-content: center;
      margin: 20px 0;
    }

    .trait-tag {
      background: rgba(255, 255, 255, 0.05);
      border: 1px solid rgba(255, 255, 255, 0.2);
      padding: 8px 16px;
      border-radius: 15px;
      cursor: pointer;
      transition: all 0.2s;
    }

    .trait-tag.selected {
      background: var(--secondary);
      border-color: var(--primary);
      color: #fff;
      box-shadow: 0 0 10px var(--secondary);
    }

    /* 遊戲狀態頂部 Bar */
    .status-bar {
      display: flex;
      justify-content: space-between;
      background: rgba(0, 0, 0, 0.6);
      padding: 12px 20px;
      border-radius: 12px;
      margin-bottom: 20px;
      border: 1px solid var(--card-border);
    }

    .status-item {
      text-align: center;
    }

    .status-item .label {
      font-size: 0.75rem;
      color: var(--text-muted);
    }

    .status-item .value {
      font-size: 1.1rem;
      font-weight: bold;
      color: var(--primary);
    }

    /* AI 事件卡片區塊 */
    .event-card {
      background: rgba(0, 0, 0, 0.4);
      border-left: 4px solid var(--primary);
      padding: 20px;
      border-radius: 8px;
      margin-bottom: 20px;
      position: relative;
    }

    .event-card .ai-badge {
      position: absolute;
      top: 10px;
      right: 15px;
      font-size: 0.7rem;
      background: rgba(112, 0, 255, 0.3);
      border: 1px solid var(--secondary);
      color: #dfaaff;
      padding: 2px 8px;
      border-radius: 10px;
    }

    .choices-container {
      display: flex;
      flex-direction: column;
      gap: 12px;
    }

    .choice-btn {
      background: rgba(255, 255, 255, 0.05);
      border: 1px solid var(--card-border);
      color: #fff;
      padding: 14px 20px;
      border-radius: 10px;
      text-align: left;
      cursor: pointer;
      transition: all 0.2s;
    }

    .choice-btn:hover {
      background: rgba(0, 240, 255, 0.15);
      border-color: var(--primary);
      transform: translateX(5px);
    }

    /* AI Loading 載入動畫 */
    .ai-loading {
      display: none;
      text-align: center;
      padding: 30px;
    }

    .spinner {
      width: 40px;
      height: 40px;
      border: 4px solid rgba(0, 240, 255, 0.1);
      border-left-color: var(--primary);
      border-radius: 50%;
      animation: spin 1s linear infinite;
      margin: 0 auto 15px auto;
    }

    @keyframes spin {
      to { transform: rotate(360deg); }
    }
  </style>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

  <!-- 全螢幕 3D 背景 Canvas -->
  <canvas id="earth-canvas"></canvas>

  <!-- 頂部 API Key 輸入列 -->
  <div class="api-key-bar">
    <label for="gemini-key">🤖 Gemini API Key:</label>
    <input type="password" id="gemini-key" placeholder="貼上 API Key..." />
    <button onclick="saveApiKey()">儲存</button>
  </div>

  <div id="app-container">

    <!-- 階段一：選擇職業 -->
    <div id="view-career" class="view-card active">
      <h1>現實人生副本模擬器</h1>
      <p style="text-align: center; color: var(--text-muted); margin-bottom: 15px;">
        點擊下方動態劃過的現實職業彈幕，或輸入自訂職業：
      </p>

      <div class="danmaku-container">
        <div class="danmaku-track" id="track1"></div>
        <div class="danmaku-track reverse" id="track2"></div>
        <div class="danmaku-track third" id="track3"></div>
      </div>

      <div class="input-group">
        <input type="text" id="custom-career-input" placeholder="找不到想要的？手動輸入自訂職業 (如: 極地攝影師)..." />
        <button class="btn" onclick="selectCustomCareer()">確認職業</button>
      </div>
    </div>

    <!-- 階段二：選擇性格與特質 -->
    <div id="view-trait" class="view-card">
      <h2>選擇你的性格與處事特質</h2>
      <p style="text-align: center; color: var(--text-muted); margin-bottom: 15px;">
        選擇 1~3 個特質（這將影響 AI 為你生成的人生事件選項）：
      </p>

      <div class="trait-cloud" id="trait-cloud"></div>

      <div class="input-group" style="margin-bottom: 20px;">
        <input type="text" id="custom-trait-input" placeholder="手動輸入自訂性格 (如: 精神離職大師)..." />
        <button class="btn" onclick="addCustomTrait()">新增特質</button>
      </div>

      <button class="btn" style="width: 100%;" onclick="startSimulation()">開始體驗人生副本 🚀</button>
    </div>

    <!-- 階段三：人生副本模擬 -->
    <div id="view-simulation" class="view-card">
      <div class="status-bar">
        <div class="status-item">
          <div class="label">當前年齡</div>
          <div class="value" id="stat-age">22 歲</div>
        </div>
        <div class="status-item">
          <div class="label">職業身分</div>
          <div class="value" id="stat-career">-</div>
        </div>
        <div class="status-item">
          <div class="label">累積資產</div>
          <div class="value" id="stat-asset">$50 萬</div>
        </div>
        <div class="status-item">
          <div class="label">幸福指數</div>
          <div class="value" id="stat-happiness">80%</div>
        </div>
      </div>

      <!-- AI 思考中的 Loading -->
      <div class="ai-loading" id="ai-loading">
        <div class="spinner"></div>
        <div style="color: var(--primary);">Gemini AI 正在構思你當前年齡的現實人生抉擇...</div>
      </div>

      <!-- 事件與選項內容 -->
      <div id="simulation-content">
        <div class="event-card">
          <span class="ai-badge" id="ai-badge">AI 實時生成</span>
          <h3 id="event-title" style="text-align: left; margin-bottom: 10px;">職場初體驗</h3>
          <p id="event-desc">你剛踏入社會，面對第一個重大職涯考驗...</p>
        </div>

        <div class="choices-container" id="choices-container"></div>
      </div>
    </div>

    <!-- 階段四：結算與結語 -->
    <div id="view-summary" class="view-card" style="text-align: center;">
      <h2>人生通關結算</h2>
      <p style="color: var(--text-muted); margin-bottom: 20px;">這是一段獨一無二的現實人生旅程</p>
      
      <div style="background: rgba(0,0,0,0.5); padding: 20px; border-radius: 12px; margin-bottom: 20px; border: 1px solid var(--gold);">
        <h3 id="summary-title" style="color: var(--gold); -webkit-text-fill-color: initial;">稱號：傳奇人生大師</h3>
        <p id="summary-details" style="margin-top: 10px; line-height: 1.6;"></p>
      </div>

      <button class="btn" onclick="restartGame()">重新開啟新人生 🔄</button>
    </div>

  </div>

  <script>
    // --- 遊戲全域狀態 ---
    const gameState = {
      career: '',
      traits: [],
      age: 22,
      asset: 50,
      happiness: 80,
      fame: 50,
      history: []
    };

    const defaultCareers = [
      "AI演算法專家", "城市美食外送員", "百萬訂閱網紅", "獨立遊戲開發者",
      "跨國企業實習生", "傳統工藝傳承人", "自媒體創作者", "健身私人教練",
      "精釀啤酒師", "遠端數位遊民", "考研重考生", "新創公司創辦人"
    ];

    const defaultTraits = [
      "完美主義", "極度樂觀", "社恐天才", "精明利己",
      "絕對理智", "精神離職大師", "風險偏好", "隨遇而安"
    ];

    // --- 初始化頁面與彈幕 ---
    window.onload = () => {
      initEarth();
      initCareers();
      initTraits();
      loadApiKey();
    };

    function loadApiKey() {
      const savedKey = localStorage.getItem('user_gemini_key');
      if (savedKey) {
        document.getElementById('gemini-key').value = savedKey;
      }
    }

    function saveApiKey() {
      const key = document.getElementById('gemini-key').value.trim();
      localStorage.setItem('user_gemini_key', key);
      alert(key ? 'Gemini API Key 已成功儲存！' : '已清除 API Key，將切換為預設事件引擎。');
    }

    function initCareers() {
      const t1 = document.getElementById('track1');
      const t2 = document.getElementById('track2');
      const t3 = document.getElementById('track3');

      const shuffled = [...defaultCareers, ...defaultCareers, ...defaultCareers];
      shuffled.forEach((c, i) => {
        const item = document.createElement('div');
        item.className = 'danmaku-item';
        item.innerText = c;
        item.onclick = () => selectCareer(c);
        if (i % 3 === 0) t1.appendChild(item);
        else if (i % 3 === 1) t2.appendChild(item);
        else t3.appendChild(item);
      });
    }

    function selectCareer(careerName) {
      gameState.career = careerName;
      switchView('view-trait');
    }

    function selectCustomCareer() {
      const input = document.getElementById('custom-career-input').value.trim();
      if (!input) return alert('請輸入職業名稱！');
      selectCareer(input);
    }

    function initTraits() {
      const container = document.getElementById('trait-cloud');
      defaultTraits.forEach(t => {
        const tag = document.createElement('div');
        tag.className = 'trait-tag';
        tag.innerText = t;
        tag.onclick = () => toggleTrait(tag, t);
        container.appendChild(tag);
      });
    }

    function toggleTrait(el, trait) {
      if (gameState.traits.includes(trait)) {
        gameState.traits = gameState.traits.filter(t => t !== trait);
        el.classList.remove('selected');
      } else {
        if (gameState.traits.length >= 3) return alert('最多選擇 3 個性格特質！');
        gameState.traits.push(trait);
        el.classList.add('selected');
      }
    }

    function addCustomTrait() {
      const input = document.getElementById('custom-trait-input');
      const val = input.value.trim();
      if (!val) return;
      if (gameState.traits.length >= 3) return alert('最多選擇 3 個性格特質！');

      gameState.traits.push(val);
      const container = document.getElementById('trait-cloud');
      const tag = document.createElement('div');
      tag.className = 'trait-tag selected';
      tag.innerText = val;
      tag.onclick = () => toggleTrait(tag, val);
      container.appendChild(tag);
      input.value = '';
    }

    function switchView(viewId) {
      document.querySelectorAll('.view-card').forEach(v => v.classList.remove('active'));
      document.getElementById(viewId).classList.add('active');
    }

    // --- 啟動模擬遊戲 ---
    function startSimulation() {
      if (gameState.traits.length === 0) {
        gameState.traits.push('隨遇而安');
      }

      document.getElementById('stat-career').innerText = gameState.career;
      switchView('view-simulation');
      updateStatusUI();
      nextTurn();
    }

    function updateStatusUI() {
      document.getElementById('stat-age').innerText = `${gameState.age} 歲`;
      document.getElementById('stat-asset').innerText = `$${gameState.asset} 萬`;
      document.getElementById('stat-happiness').innerText = `${gameState.happiness}%`;
    }

    // --- 核心 AI 生成 / 推進邏輯 ---
    async function nextTurn() {
      if (gameState.age >= 60) {
        endGame();
        return;
      }

      const apiKey = localStorage.getItem('user_gemini_key');
      const loadingEl = document.getElementById('ai-loading');
      const contentEl = document.getElementById('simulation-content');

      loadingEl.style.display = 'block';
      contentEl.style.opacity = '0.3';

      let eventData = null;

      if (apiKey) {
        // 使用 Gemini API 生成動態事件
        eventData = await fetchGeminiEvent(apiKey);
      }

      // 若未提供 Key 或 API 呼叫失敗，降級使用內建演算法生成
      if (!eventData) {
        eventData = getFallbackEvent();
        document.getElementById('ai-badge').innerText = '系統預設事件引擎';
      } else {
        document.getElementById('ai-badge').innerText = 'Gemini AI 即時構思';
      }

      loadingEl.style.display = 'none';
      contentEl.style.opacity = '1';

      renderEvent(eventData);
    }

    async function fetchGeminiEvent(apiKey) {
      const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${apiKey}`;

      const promptText = `你是一個專業的人生副本模擬遊戲引擎。
玩家資訊：
- 職業：${gameState.career}
- 性格特質：${gameState.traits.join(', ')}
- 當前年齡：${gameState.age} 歲
- 目前資產：${gameState.asset} 萬元
- 幸福指數：${gameState.happiness}%

請為玩家生成一個符合現實生活發展的關鍵考驗事件與 3 個選項。
回傳格式必須嚴格為 JSON (不要有 Markdown 標記，直接輸出 JSON 內容)：
{
  "title": "事件標題",
  "description": "200字以內的現實情境描寫...",
  "choices": [
    {
      "text": "選項 A 的行動描述",
      "assetChange": 20,
      "happinessChange": -10
    },
    {
      "text": "選項 B 的行動描述",
      "assetChange": -10,
      "happinessChange": 15
    },
    {
      "text": "選項 C 的行動描述",
      "assetChange": 0,
      "happinessChange": 5
    }
  ]
}`;

      try {
        const response = await fetch(url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            contents: [{ parts: [{ text: promptText }] }]
          })
        });

        const data = await response.json();
        let rawText = data.candidates[0].content.parts[0].text;
        
        // 清理可能的 Markdown JSON 標記
        rawText = rawText.replace(/```json/g, '').replace(/```/g, '').trim();
        return JSON.parse(rawText);
      } catch (err) {
        console.warn('Gemini API 呼叫失敗或解析錯誤，切換至備用引擎:', err);
        return null;
      }
    }

    function getFallbackEvent() {
      return {
        title: `${gameState.age} 歲的現實考驗`,
        description: `作為一名【${gameState.career}】，你的性格【${gameState.traits.join('/')}】讓你在這個階段遇到了職涯與生活的抉擇...`,
        choices: [
          { text: "選擇積極進攻，承擔風險追求高報酬", assetChange: 30, happinessChange: -10 },
          { text: "穩紮穩打，優先提升個人生活品質與健康", assetChange: 5, happinessChange: 20 },
          { text: "嘗試斜槓探索，開啟全新的生活可能性", assetChange: 15, happinessChange: 10 }
        ]
      };
    }

    function renderEvent(eventData) {
      document.getElementById('event-title').innerText = eventData.title;
      document.getElementById('event-desc').innerText = eventData.description;

      const choicesContainer = document.getElementById('choices-container');
      choicesContainer.innerHTML = '';

      eventData.choices.forEach((choice, index) => {
        const btn = document.createElement('button');
        btn.className = 'choice-btn';
        btn.innerHTML = `<strong>${String.fromCharCode(65 + index)}. ${choice.text}</strong><br>
          <small style="color: var(--text-muted);">預期影響: 資產 ${choice.assetChange >= 0 ? '+' : ''}${choice.assetChange}萬 / 幸福 ${choice.happinessChange >= 0 ? '+' : ''}${choice.happinessChange}%</small>`;
        
        btn.onclick = () => makeChoice(choice);
        choicesContainer.appendChild(btn);
      });
    }

    function makeChoice(choice) {
      gameState.asset += choice.assetChange;
      gameState.happiness = Math.max(0, Math.min(100, gameState.happiness + choice.happinessChange));
      gameState.age += 5; // 每輪推進 5 歲

      updateStatusUI();
      nextTurn();
    }

    function endGame() {
      switchView('view-summary');
      let title = "平凡真摯的現實人生";
      if (gameState.asset > 200 && gameState.happiness > 70) title = "完美成就：白手起家傳奇";
      else if (gameState.asset > 300) title = "商業巨擘：金錢帝國締造者";
      else if (gameState.happiness > 80) title = "靈魂勝者：樂活人生大師";

      document.getElementById('summary-title').innerText = `專屬稱號：${title}`;
      document.getElementById('summary-details').innerHTML = `
        職業起點：${gameState.career}<br>
        性格標籤：${gameState.traits.join(', ')}<br>
        最終累積資產：$${gameState.asset} 萬元<br>
        最終幸福指數：${gameState.happiness}%
      `;
    }

    function restartGame() {
      gameState.age = 22;
      gameState.asset = 50;
      gameState.happiness = 80;
      gameState.traits = [];
      document.querySelectorAll('.trait-tag').forEach(t => t.classList.remove('selected'));
      switchView('view-career');
    }

    // --- Three.js 3D 背景地球繪製 ---
    function initEarth() {
      const canvas = document.getElementById('earth-canvas');
      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      const renderer = new THREE.WebGLRenderer({ canvas, alpha: true, antialias: true });
      
      renderer.setSize(window.innerWidth, window.innerHeight);

      // 粒子地球
      const geometry = new THREE.SphereGeometry(3.5, 48, 48);
      const material = new THREE.PointsMaterial({
        color: 0x00f0ff,
        size: 0.025,
        transparent: true,
        opacity: 0.7
      });
      const sphere = new THREE.Points(geometry, material);
      scene.add(sphere);

      camera.position.z = 7;

      function animate() {
        requestAnimationFrame(animate);
        sphere.rotation.y += 0.002;
        sphere.rotation.x += 0.0005;
        renderer.render(scene, camera);
      }
      animate();

      window.addEventListener('resize', () => {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
      });
    }
  </script>
</body>
</html>
