## あなたのIPアドレス等の情報を表示する
  
<button id="get-ip-btn">IPアドレス等の情報を表示する</button>

<p id="ip-address"> </p>
このボタンを押すと、アクセス先が認識できるあなたのIPアドレス、UA、都市情報、郵便番号等の情報を表示します。
IPアドレス等の情報は表示のためだけに一時的に取得しています。
なお、別途プラットフォーム側で保存されている可能性はあります。
JSで取得できるブラウザフィンガープリントの様な情報も表示しています。  
CloudflareのWorkerを使ってるだけなので、変な所に情報を送っている訳ではありません。

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
            X-Forwarded-For: ${data.xForwardedFor}<br>
            Via: ${data.via}<br>
            Forwarded: ${data.forwarded}<br>
            User-Agent: ${data.userAgent}<br><br>
            User-Agent(navigator): ${navigator.userAgent}<br>
            tlsVersion: ${data.tlsVersion}<br>
            httpProtocol: ${data.httpProtocol}<br>
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

  ---

## スピードテスト（ダウンロード速度）

<button id="speedtest-btn">スピードテストを実行（約10秒）</button>

<div id="speed-result"></div>
※cloudflareの機能を使って実装しています。
<script>
// === スピードテスト機能（ページ末尾に追加）===
const speedBtn = document.getElementById('speedtest-btn');
const speedResult = document.getElementById('speed-result');

speedBtn.addEventListener('click', async () => {
  speedBtn.disabled = true;
  speedBtn.textContent = '測定中...（約10秒）';
  speedResult.style.display = 'block';
  speedResult.innerHTML = '<strong>測定開始...</strong>';

  // 10MBテストファイル（軽め・実用的）
  const testUrl = 'https://speed.cloudflare.com/__down?bytes=10000000';
  const startTime = Date.now();

  try {
    const response = await fetch(testUrl, { cache: 'no-store' });
    const blob = await response.blob();
    const endTime = Date.now();

    const durationSec = (endTime - startTime) / 1000;
    const sizeMB = blob.size / (1024 * 1024);
    const speedMbps = (sizeMB / durationSec) * 8;   // Mbps換算

    speedResult.innerHTML = `
      <strong>ダウンロード速度: ${speedMbps.toFixed(1)} Mbps</strong><br>
      ファイルサイズ: ${sizeMB.toFixed(1)} MB<br>
      所要時間: ${durationSec.toFixed(2)} 秒<br>
      <small>※目安値です。ネットワーク状況やブラウザによって変動します。</small>
    `;
  } catch (e) {
    speedResult.innerHTML = '測定に失敗しました。<br>もう一度試してください。';
  }

  speedBtn.disabled = false;
  speedBtn.textContent = 'もう一度スピードテストを実行';
});
</script>

