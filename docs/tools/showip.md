## あなたのIPアドレスの情報を表示する
  
<button id="get-ip-btn">IPアドレス等を表示する</button>

<p id="ip-address">ボタンを押してください</p>
※このボタンを押すと、外部から認識できる、あなたのIPアドレス、UA、都市情報、郵便番号等の情報を表示します。
IPアドレス等の情報は表示のためだけに一時的に取得しています。なお、別途プラットフォーム側で保存されている可能性はあります。JSで取得できるブラウザフィンガープリントの様な情報も表示しています。

  <script>
    const btn = document.getElementById('get-ip-btn');
    const display = document.getElementById('ip-address');

    btn.addEventListener('click', () => {
      btn.disabled = true;
      btn.textContent = '取得中...';
      display.textContent = '取得中...';

      fetch('https://show-ip.hajimetwi3.workers.dev')
        .then(r => {
          if (r.status === 429) {
            throw new Error('RATE_LIMIT_EXCEEDED');
          }
          if (!r.ok) {
            throw new Error('FETCH_ERROR');
          }
          return r.json();
        })
        .then(data => {
        // === 接続情報===
        let connInfo = 'unknown';
        if (navigator.connection) {
          const c = navigator.connection;
          connInfo = `
            effectiveType: ${c.effectiveType || 'unknown'} 
            downlink: ${c.downlink || 'unknown'} Mbps 
            type: ${c.type || 'unknown'} 
            rtt: ${c.rtt || 'unknown'} ms`;
          }
          display.innerHTML = `
            IP: <strong>${data.ip}</strong><br>

            取得日時: ${data.timestamp}<br>
            事業者: ${data.isp}<br>
            AS番号: AS${data.asn}<br>
            都市: ${data.city} / ${data.region}<br>
            郵便番号: ${data.postalCode}<br>
            位置: ${data.latitude}, ${data.longitude}<br>
            タイムゾーン: ${data.timezone}<br>
            リファラ: <small>${data.referer}</small><br>
            ホスト名: ${data.hostname}<br>
            User-Agent: ${data.userAgent}<br><br>
            User-Agent(navigator): ${navigator.userAgent}<br>
            言語: ${data.acceptLanguage}<br>
            言語(navigator): ${navigator.language}<br>
            画面: ${screen.width}×${screen.height} (${window.devicePixelRatio}x)<br>
            CPUコア: ${navigator.hardwareConcurrency || 'unknown'}<br>
            メモリ(ブラウザ推定値）: ${navigator.deviceMemory ? navigator.deviceMemory + 'GB' : 'unknown'}<br>
            プラットフォーム: ${navigator.platform}<br>
            接続情報: ${connInfo}<br>
          `;
          btn.textContent = '再度取得する';
          btn.disabled = false;
        })
        .catch(err => {
          if (err.message === 'RATE_LIMIT_EXCEEDED') {
            display.textContent = 'アクセス制限中です。少し待ってから再度お試しください';
          } else {
            display.textContent = 'IP取得に失敗しました。しばらく待ってから再度お試しください';
          }
          btn.textContent = 'もう一度試す';
          btn.disabled = false;
        });
    });
  </script>
