<!DOCTYPE html>
<html lang="pt">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Avaliação da Praia</title>
    <style>
      body {
        font-family: sans-serif;
        text-align: center;
        padding: 4rem;
        margin: 0;
      }
      h1 {
        font-size: 4rem;
        color: #007BFF;
        margin-bottom: 1rem;
      }
      .vote-btn {
        font-size: 9rem;
        margin: 2rem;
        cursor: pointer;
      }
      .label {
        display: block;
        margin-top: 0.5rem;
        font-size: 1.2rem;
      }
      #thanks {
        display: none;
        margin-top: 2rem;
        font-size: 1.5rem;
      }
      #thanks-message {
        margin-bottom: 1rem;
      }
      #qr-code {
        width: 150px;
        height: 150px;
        margin: 1rem auto;
      }
      #logo-fundo {
        position: fixed;
        bottom: 1px;
        left: 50%;
        transform: translateX(-50%);
        width: 350px;
        height: 350px;
        background-image: url('https://cdn.glitch.global/0e725a93-508c-4ec5-8b4f-959ef088f183/logo.png?v=1748093603467');
        background-repeat: no-repeat;
        background-size: contain;
        background-position: center;
        pointer-events: none;
        z-index: -1;
        background-color: white;
      }
      #status {
        margin-top: 1rem;
        font-size: 1rem;
        color: #333;
      }
    </style>
  </head>
  <body>
    <h1>Esta praia é limpa?</h1>
    <div id="buttons">
      <div onclick="enviarVoto(2)" class="vote-btn">😃<span class="label">Muito limpa</span></div>
      <div onclick="enviarVoto(1)" class="vote-btn">🙂<span class="label">Limpa</span></div>
      <div onclick="enviarVoto(0)" class="vote-btn">😞<span class="label">Nada limpa</span></div>
    </div>
    <div id="status"></div>

    <div id="thanks">
      <div id="thanks-message">Obrigado pelo teu contributo!</div>
      <div id="Inquerito">Preenche o inquérito completo e ajuda-nos a saber mais sobre o estado desta praia!</div>
      <img id="qr-code" src="https://cdn.glitch.global/0e725a93-508c-4ec5-8b4f-959ef088f183/QR%20percepcao%20lixo%20praias.png?v=1748440390107" alt="QR Code para o inquérito" />
    </div>

    <div id="logo-fundo"></div>

    <script>
      let cachedLocation = null;

      function getCachedLocation() {
        const data = localStorage.getItem('cachedLocation');
        if (!data) return null;
        try {
          const obj = JSON.parse(data);
          const today = new Date().toISOString().slice(0, 10);
          if (obj.date === today) {
            return { lat: obj.lat, lon: obj.lon };
          }
        } catch {
          return null;
        }
        return null;
      }

      function cacheLocation(lat, lon) {
        const today = new Date().toISOString().slice(0, 10);
        localStorage.setItem('cachedLocation', JSON.stringify({ date: today, lat, lon }));
      }

      window.onload = () => {
        cachedLocation = getCachedLocation();
        if (!cachedLocation) {
          if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(
              pos => {
                cachedLocation = {
                  lat: pos.coords.latitude,
                  lon: pos.coords.longitude,
                };
                cacheLocation(cachedLocation.lat, cachedLocation.lon);
              },
              err => {
                console.warn('Não foi possível obter localização:', err.message);
              },
              { enableHighAccuracy: true, timeout: 10000 }
            );
          }
        }
      };

      function enviarVoto(voto) {
        const statusEl = document.getElementById('status');
        statusEl.textContent = 'A obter localização…';

        if (cachedLocation) {
          statusEl.textContent = `GPS (cache): ${cachedLocation.lat.toFixed(6)}, ${cachedLocation.lon.toFixed(6)}`;
          enviarDados(voto, cachedLocation.lat, cachedLocation.lon);
          return;
        }

        if (!navigator.geolocation) {
          alert('GPS não suportado neste navegador.');
          enviarDados(voto, '', '');
          return;
        }

        navigator.geolocation.getCurrentPosition(
          position => {
            const lat = position.coords.latitude;
            const lon = position.coords.longitude;
            cacheLocation(lat, lon);
            cachedLocation = { lat, lon };
            statusEl.textContent = `GPS: ${lat.toFixed(6)}, ${lon.toFixed(6)}`;
            enviarDados(voto, lat, lon);
          },
          error => {
            alert('Não foi possível obter GPS: ' + error.message);
            statusEl.textContent = 'Erro GPS: ' + error.message;
            enviarDados(voto, '', '');
          },
          { enableHighAccuracy: true, timeout: 10000, maximumAge: 0 }
        );
      }

      function enviarDados(voto, lat, lon) {
        let body = 'voto=' + encodeURIComponent(voto);
        if (lat !== '' && lon !== '') {
          body += '&lat=' + encodeURIComponent(lat) + '&lon=' + encodeURIComponent(lon);
        }
        fetch(
          'https://script.google.com/macros/s/AKfycbz9F3JdODG3O0tXohr_Fq8wMr3qeiBE2OKSK_FNYAUp4M7ynunzZC1WWiknExnzGLzO/exec',
          {
            method: 'POST',
            mode: 'no-cors',
            headers: {
              'Content-Type': 'application/x-www-form-urlencoded'
            },
            body: body
          }
        )
          .then(() => {
            document.getElementById('thanks').style.display = 'block';
            document.getElementById('buttons').style.display = 'none';
            document.getElementById('status').textContent = '';
            setTimeout(() => {
              document.getElementById('thanks').style.display = 'none';
              document.getElementById('buttons').style.display = 'block';
            }, 5000);
          })
          .catch(err => {
            console.error('Erro no fetch:', err);
            alert('Ocorreu um erro no envio do voto.');
          });
      }
    </script>
  </body>
</html>
