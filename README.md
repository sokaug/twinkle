<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>ベルハーモニー</title>

  <!-- フォント（任意） -->
  <link href="https://fonts.googleapis.com/css2?family=Rounded+Mplus+1c:wght@400;700&display=swap" rel="stylesheet">

  <style>
    body { box-sizing: border-box; }
    html, body {
      margin: 0;
      padding: 0;
      height: 100%;
      width: 100%;
      overflow: hidden;
    }
    * { box-sizing: border-box; }

    .app-container {
      width: 100%;
      height: 100%;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      font-family: 'Rounded Mplus 1c', sans-serif;
      padding: 20px;
    }

    .title {
      font-size: 48px;
      font-weight: 700;
      color: #ffffff;
      margin-bottom: 20px;
      text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
    }

    .controls {
      display: flex;
      gap: 15px;
      margin-bottom: 30px;
      flex-wrap: wrap;
      justify-content: center;
    }

    .mode-button {
      padding: 12px 24px;
      font-size: 16px;
      font-weight: 700;
      border: none;
      border-radius: 25px;
      cursor: pointer;
      transition: all 0.3s ease;
      font-family: 'Rounded Mplus 1c', sans-serif;
      box-shadow: 0 4px 10px rgba(0,0,0,0.2);
    }
    .mode-button.free { background: #ffffff; color: #667eea; }
    .mode-button.practice { background: #ffd700; color: #333333; }

    .mode-button:hover {
      transform: translateY(-2px);
      box-shadow: 0 6px 15px rgba(0,0,0,0.3);
    }
    .mode-button.active {
      transform: scale(1.05);
      box-shadow: 0 6px 20px rgba(255,255,255,0.4);
    }

    .practice-info {
      color: #ffffff;
      font-size: 18px;
      margin-bottom: 20px;
      text-shadow: 1px 1px 2px rgba(0,0,0,0.3);
    }

    .bells-container {
      display: flex;
      gap: 15px;
      flex-wrap: wrap;
      justify-content: center;
      max-width: 900px;
    }

    .bell {
      position: relative;
      width: 100px;
      height: 140px;
      cursor: pointer;
      transition: transform 0.1s ease;
      user-select: none;
    }
    .bell:active { transform: scale(0.95); }

    .bell-body {
      width: 100%;
      height: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }

    .bell-top {
      width: 30px;
      height: 15px;
      background: #8b7355;
      border-radius: 15px 15px 0 0;
      margin-bottom: -5px;
      z-index: 2;
    }

    .bell-shape {
      width: 80px;
      height: 100px;
      border-radius: 40px 40px 50px 50px;
      box-shadow:
        0 8px 20px rgba(0,0,0,0.3),
        inset -5px -5px 15px rgba(0,0,0,0.2),
        inset 5px 5px 15px rgba(255,255,255,0.3);
      display: flex;
      align-items: center;
      justify-content: center;
      position: relative;
    }

    .bell-shape.ringing { animation: ring 0.3s ease-in-out; }
    .bell-shape.highlight {
      animation: glow 0.8s ease-in-out infinite;
      box-shadow:
        0 0 30px rgba(255,255,255,0.9),
        0 8px 20px rgba(0,0,0,0.3),
        inset -5px -5px 15px rgba(0,0,0,0.2),
        inset 5px 5px 15px rgba(255,255,255,0.3);
    }

    @keyframes ring {
      0%, 100% { transform: rotate(0deg); }
      25% { transform: rotate(-10deg); }
      75% { transform: rotate(10deg); }
    }

    @keyframes glow {
      0%, 100% { filter: brightness(1.5); transform: scale(1.05); }
      50% { filter: brightness(1.8); transform: scale(1.1); }
    }

    .bell-note {
      font-size: 28px;
      font-weight: 700;
      color: #8b4513;
      text-shadow: 1px 1px 2px rgba(255,255,255,0.5);
    }

    .bell-label {
      margin-top: 8px;
      font-size: 14px;
      font-weight: 700;
      color: #ffffff;
      text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
    }

    @media (max-width: 768px) {
      .title { font-size: 32px; margin-bottom: 30px; }
      .bell { width: 80px; height: 120px; }
      .bell-shape { width: 65px; height: 85px; }
      .bells-container { gap: 10px; }
    }
  </style>
</head>

<body>
  <div class="app-container">
    <h1 class="title" id="title">ベルハーモニー</h1>

    <div class="controls">
      <button class="mode-button free active" id="freeMode">自由演奏</button>
      <button class="mode-button practice" id="practiceMode">きらきら星を練習</button>
    </div>

    <div class="practice-info" id="practiceInfo" style="display:none;"></div>
    <div class="bells-container" id="bellsContainer"></div>
  </div>

  <script>
    const notes = [
      { note: 'ド', frequency: 261.63, label: 'C', color: '#ff4444' },
      { note: 'レ', frequency: 293.66, label: 'D', color: '#ff8833' },
      { note: 'ミ', frequency: 329.63, label: 'E', color: '#ffdd33' },
      { note: 'ファ', frequency: 349.23, label: 'F', color: '#44dd44' },
      { note: 'ソ', frequency: 392.00, label: 'G', color: '#3399ff' },
      { note: 'ラ', frequency: 440.00, label: 'A', color: '#4488ff' },
      { note: 'シ', frequency: 493.88, label: 'B', color: '#ff66cc' }
    ];

    const twinkleSong = [
      'ド', 'ド', 'ソ', 'ソ', 'ラ', 'ラ', 'ソ',
      'ファ', 'ファ', 'ミ', 'ミ', 'レ', 'レ', 'ド',
      'ソ', 'ソ', 'ファ', 'ファ', 'ミ', 'ミ', 'レ',
      'ソ', 'ソ', 'ファ', 'ファ', 'ミ', 'ミ', 'レ',
      'ド', 'ド', 'ソ', 'ソ', 'ラ', 'ラ', 'ソ',
      'ファ', 'ファ', 'ミ', 'ミ', 'レ', 'レ', 'ド'
    ];

    let practiceMode = false;
    let currentNoteIndex = 0;
    let bellElements = {};
    let audioContext;

    function initAudio() {
      if (!audioContext) audioContext = new (window.AudioContext || window.webkitAudioContext)();
      // iOS対策：ユーザー操作後にresumeされていない場合がある
      if (audioContext.state === 'suspended') audioContext.resume();
    }

    function playBellSound(frequency) {
      initAudio();

      const oscillator = audioContext.createOscillator();
      const gainNode = audioContext.createGain();

      oscillator.connect(gainNode);
      gainNode.connect(audioContext.destination);

      oscillator.type = 'sine';
      oscillator.frequency.setValueAtTime(frequency, audioContext.currentTime);

      gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);
      gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 1.5);

      oscillator.start(audioContext.currentTime);
      oscillator.stop(audioContext.currentTime + 1.5);
    }

    function adjustBrightness(hex, percent) {
      const num = parseInt(hex.replace('#', ''), 16);
      const r = Math.min(255, Math.floor((num >> 16) * percent));
      const g = Math.min(255, Math.floor(((num >> 8) & 0x00FF) * percent));
      const b = Math.min(255, Math.floor((num & 0x0000FF) * percent));
      return '#' + ((r << 16) | (g << 8) | b).toString(16).padStart(6, '0');
    }

    function updatePracticeInfo() {
      const info = document.getElementById('practiceInfo');
      info.textContent = `♪ ${currentNoteIndex + 1} / ${twinkleSong.length} - 次: ${twinkleSong[currentNoteIndex]}`;
    }

    function highlightNextNote() {
      Object.values(bellElements).forEach(el => el.classList.remove('highlight'));

      if (currentNoteIndex < twinkleSong.length) {
        const nextNote = twinkleSong[currentNoteIndex];
        if (bellElements[nextNote]) bellElements[nextNote].classList.add('highlight');
        updatePracticeInfo();
      } else {
        document.getElementById('practiceInfo').textContent =
          '🎉 完璧！きらきら星をマスターしました！もう一度練習しますか？';
        setTimeout(() => {
          currentNoteIndex = 0;
          highlightNextNote();
        }, 3000);
      }
    }

    function checkNote(playedNote) {
      const expectedNote = twinkleSong[currentNoteIndex];
      if (playedNote === expectedNote) {
        if (bellElements[playedNote]) bellElements[playedNote].classList.remove('highlight');
        currentNoteIndex++;
        setTimeout(() => highlightNextNote(), 400);
      }
    }

    function setFreeMode() {
      practiceMode = false;
      currentNoteIndex = 0;
      document.getElementById('practiceInfo').style.display = 'none';
      document.getElementById('freeMode').classList.add('active');
      document.getElementById('practiceMode').classList.remove('active');
      Object.values(bellElements).forEach(el => el.classList.remove('highlight'));
    }

    function setPracticeMode() {
      practiceMode = true;
      currentNoteIndex = 0;
      document.getElementById('practiceInfo').style.display = 'block';
      document.getElementById('freeMode').classList.remove('active');
      document.getElementById('practiceMode').classList.add('active');
      highlightNextNote();
    }

    function createBells() {
      const container = document.getElementById('bellsContainer');
      container.innerHTML = '';
      bellElements = {};

      notes.forEach(({ note, frequency, label, color }) => {
        const bell = document.createElement('div');
        bell.className = 'bell';
        bell.dataset.note = note;

        const bright = adjustBrightness(color, 1.2);

        bell.innerHTML = `
          <div class="bell-body">
            <div class="bell-top"></div>
            <div class="bell-shape" style="background: linear-gradient(180deg, ${color} 0%, ${bright} 50%, ${color} 100%);">
              <div class="bell-note">${note}</div>
            </div>
            <div class="bell-label">${label}</div>
          </div>
        `;

        const bellShape = bell.querySelector('.bell-shape');
        bellElements[note] = bellShape;

        bell.addEventListener('click', () => {
          bellShape.classList.add('ringing');
          playBellSound(frequency);

          if (practiceMode) checkNote(note);

          setTimeout(() => bellShape.classList.remove('ringing'), 300);
        });

        container.appendChild(bell);
      });

      if (practiceMode) highlightNextNote();
    }

    document.getElementById('freeMode').addEventListener('click', setFreeMode);
    document.getElementById('practiceMode').addEventListener('click', setPracticeMode);

    createBells();
  </script>
</body>
</html>
