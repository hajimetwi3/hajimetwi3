## Show your IP address and browser info

<button id="get-ip-btn">Show my IP and browser info</button>

<p id="ip-address"> </p>
Click the button above to display information that servers can see about your connection - your IP address, user agent, approximate city, postal code, and so on.
This information is only fetched for display; it is not stored by this page (though the underlying platform may log requests separately).
Browser fingerprinting information available via JavaScript is also shown.
This page only uses a Cloudflare Worker - your data is not sent anywhere suspicious.

### Displayed fields
IP, Timestamp, ISP, AS number, City / Region, Postal code, Coordinates, Timezone, Referer, X-Forwarded-For, Via, Forwarded, User-Agent, User-Agent (navigator), tlsVersion, httpProtocol, Language, Language (navigator), screen size, window size, devicePixelRatio, CPU cores, Memory (browser estimate), Platform, Connection info
*Actual fields may vary slightly.*

  <script>
    const btn = document.getElementById('get-ip-btn');
    const display = document.getElementById('ip-address');

    btn.addEventListener('click', () => {
      btn.disabled = true;
      btn.textContent = 'Loading...';
      display.textContent = 'Loading...';

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
          let connInfo = 'unknown';
          if (navigator.connection) {
            const c = navigator.connection;
            connInfo = `effectiveType: ${c.effectiveType || 'unknown'}, downlink: ${c.downlink || 'unknown'} Mbps, type: ${c.type || 'unknown'}, rtt: ${c.rtt || 'unknown'} ms`;
          }

          const rows = [
            ['IP', data.ip],
            ['Timestamp', data.timestamp],
            ['ISP', data.isp],
            ['AS number', 'AS' + data.asn],
            ['City / Region', `${data.city} / ${data.region}`],
            ['Postal code', data.postalCode],
            ['Coordinates', `${data.latitude}, ${data.longitude}`],
            ['Timezone', data.timezone],
            ['Referer', data.referer],
            ['X-Forwarded-For', data.xForwardedFor],
            ['Via', data.via],
            ['Forwarded', data.forwarded],
            ['User-Agent', data.userAgent],
            ['User-Agent (navigator)', navigator.userAgent],
            ['tlsVersion', data.tlsVersion],
            ['httpProtocol', data.httpProtocol],
            ['Language', data.acceptLanguage],
            ['Language (navigator)', navigator.language],
            ['screen.width × screen.height', `${screen.width}×${screen.height}`],
            ['window.outerWidth × outerHeight', `${window.outerWidth}×${window.outerHeight}`],
            ['window.innerWidth × innerHeight', `${window.innerWidth}×${window.innerHeight}`],
            ['window.devicePixelRatio', window.devicePixelRatio],
            ['CPU cores', navigator.hardwareConcurrency || 'unknown'],
            ['Memory (browser estimate)', navigator.deviceMemory ? navigator.deviceMemory + 'GB' : 'unknown'],
            ['Platform', navigator.platform],
            ['Connection info', connInfo],
          ];

          display.replaceChildren();
          for (const [label, value] of rows) {
            const span = document.createElement('span');
            span.textContent = `${label}: ${String(value ?? '')}`;
            display.appendChild(span);
            display.appendChild(document.createElement('br'));
          }

          btn.textContent = 'Refresh';
          btn.disabled = false;
        })
        .catch(err => {
          if (err.message === 'RATE_LIMIT_EXCEEDED') {
            display.textContent = 'Rate limit exceeded. Please wait a moment and try again.';
          } else {
            display.textContent = 'Failed to fetch. Please try again in a moment.';
          }
          btn.textContent = 'Try again';
          btn.disabled = false;
        });
    });
  </script>

---

## Speed test (download)

<button id="speedtest-btn">Run speed test (~10s)</button>

<div id="speed-result"></div>
*Powered by Cloudflare's speed test endpoint.*

<script>
const speedBtn = document.getElementById('speedtest-btn');
const speedResult = document.getElementById('speed-result');

speedBtn.addEventListener('click', async () => {
  speedBtn.disabled = true;
  speedBtn.textContent = 'Measuring... (~10s)';
  speedResult.style.display = 'block';
  speedResult.innerHTML = '<strong>Starting...</strong>';

  // 10MB test file (lightweight, realistic)
  const testUrl = 'https://speed.cloudflare.com/__down?bytes=10000000';
  const startTime = Date.now();

  try {
    const response = await fetch(testUrl, { cache: 'no-store' });
    const blob = await response.blob();
    const endTime = Date.now();

    const durationSec = (endTime - startTime) / 1000;
    const sizeMB = blob.size / (1024 * 1024);
    const speedMbps = (sizeMB / durationSec) * 8;   // convert to Mbps

    speedResult.innerHTML = `
      <strong>Download speed: ${speedMbps.toFixed(1)} Mbps</strong><br>
      File size: ${sizeMB.toFixed(1)} MB<br>
      Elapsed: ${durationSec.toFixed(2)} s<br>
      <small>* Approximate. Actual speed varies by network conditions and browser.</small>
    `;
  } catch (e) {
    speedResult.innerHTML = 'Measurement failed.<br>Please try again.';
  }

  speedBtn.disabled = false;
  speedBtn.textContent = 'Run speed test again';
});
</script>

---
## Details
The source of this page is available here:
[https://github.com/hajimetwi3/hajimetwi3/blob/main/docs/tools/showip.en.md?plain=1](https://github.com/hajimetwi3/hajimetwi3/blob/main/docs/tools/showip.en.md?plain=1)
