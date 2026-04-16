## あなたのIPアドレス
  
  <p id="ip-address">取得中...</p>

  <script>
    const display = document.getElementById('ip-address');

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
          国: ${data.country}
        `;
      })
      .catch(err => {
        if (err.message === 'RATE_LIMIT_EXCEEDED') {
          display.textContent = 'アクセス制限中です（1分に20回まで）。少し待ってから再読み込みしてください';
        } else {
          display.textContent = 'IP取得に失敗しました。しばらく待ってから再度お試しください';
        }
      });
  </script>
