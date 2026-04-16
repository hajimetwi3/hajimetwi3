## あなたのIPアドレス
  
<button id="get-ip-btn">IPアドレスを表示する</button>

  <p id="ip-address">ボタンを押してください</p>
  ※このボタンを押すと、外部から認識できる、あなたのIPアドレス、UA、都市情報等の情報を表示します。
  　IPアドレス等の情報は表示のためだけに一時的に取得しています。なお、別途プラットフォーム側で保存されている可能性はあります。

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
          display.innerHTML = `
            IP: <strong>${data.ip}</strong><br>
            UA: ${data.userAgent}<br>
            国: ${data.country}<br>
            事業者: ${data.isp}<br>
            都市: ${data.city} / ${data.region}
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
