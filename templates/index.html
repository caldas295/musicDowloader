<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Spool — downloader de áudio</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;700&family=Inter:wght@400;500;600&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #14161a;
    --panel: #1c1f26;
    --panel-2: #22262e;
    --line: #2d323c;
    --text: #ece9e2;
    --muted: #8b909c;
    --accent: #6ee7b0;
    --amber: #f2b750;
    --red: #ff6b6b;
  }

  * { box-sizing: border-box; }

  body {
    margin: 0;
    background: var(--bg);
    background-image:
      radial-gradient(circle at 15% 0%, rgba(110,231,176,0.06), transparent 40%),
      radial-gradient(circle at 85% 100%, rgba(242,183,80,0.05), transparent 40%);
    color: var(--text);
    font-family: 'Inter', system-ui, sans-serif;
    min-height: 100vh;
    display: flex;
    justify-content: center;
    padding: 48px 20px 80px;
  }

  .wrap { width: 100%; max-width: 620px; }

  .eyebrow {
    font-family: 'JetBrains Mono', monospace;
    font-size: 12px;
    letter-spacing: 0.18em;
    color: var(--muted);
    text-transform: uppercase;
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 10px;
  }

  .eyebrow .dot { width: 6px; height: 6px; border-radius: 50%; background: var(--accent); }

  h1 {
    font-family: 'JetBrains Mono', monospace;
    font-size: 30px;
    font-weight: 700;
    margin: 0 0 6px;
    letter-spacing: -0.01em;
  }

  .sub { color: var(--muted); font-size: 14.5px; margin-bottom: 28px; line-height: 1.5; }

  .meter {
    display: flex;
    align-items: flex-end;
    gap: 3px;
    height: 22px;
    margin-bottom: 22px;
  }
  .meter span {
    width: 4px;
    background: var(--line);
    border-radius: 1px;
    height: 20%;
    transition: background 0.2s;
  }
  .meter.ativo span {
    background: var(--accent);
    animation: bounce 0.9s ease-in-out infinite;
  }
  .meter.ativo span:nth-child(odd) { animation-duration: 0.7s; }
  .meter.ativo span:nth-child(3n) { animation-duration: 1.1s; }
  @keyframes bounce {
    0%, 100% { height: 15%; }
    50% { height: 90%; }
  }

  .card {
    background: var(--panel);
    border: 1px solid var(--line);
    border-radius: 10px;
    padding: 22px;
  }

  label {
    display: block;
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    color: var(--muted);
    margin-bottom: 8px;
  }

  input[type=text] {
    width: 100%;
    background: var(--panel-2);
    border: 1px solid var(--line);
    border-radius: 6px;
    color: var(--text);
    font-size: 14.5px;
    padding: 13px 14px;
    font-family: 'Inter', sans-serif;
  }
  input[type=text]:focus { outline: none; border-color: var(--accent); }
  input[type=text]::placeholder { color: #565b66; }

  .row { display: flex; gap: 12px; margin-top: 18px; }
  .col { flex: 1; }

  .segmented {
    display: flex;
    background: var(--panel-2);
    border: 1px solid var(--line);
    border-radius: 6px;
    overflow: hidden;
  }
  .segmented button {
    flex: 1;
    background: transparent;
    border: none;
    color: var(--muted);
    font-family: 'JetBrains Mono', monospace;
    font-size: 12.5px;
    padding: 11px 0;
    cursor: pointer;
    transition: 0.15s;
  }
  .segmented button.ativo { background: var(--accent); color: #0d1410; font-weight: 700; }

  select {
    width: 100%;
    background: var(--panel-2);
    border: 1px solid var(--line);
    border-radius: 6px;
    color: var(--text);
    font-family: 'JetBrains Mono', monospace;
    font-size: 13px;
    padding: 11px 12px;
  }
  select:disabled { opacity: 0.4; }

  .btn-baixar {
    width: 100%;
    margin-top: 22px;
    background: var(--accent);
    color: #0d1410;
    border: none;
    border-radius: 6px;
    padding: 15px 0;
    font-family: 'JetBrains Mono', monospace;
    font-weight: 700;
    font-size: 14px;
    letter-spacing: 0.04em;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    transition: 0.15s;
  }
  .btn-baixar:hover { filter: brightness(1.08); }
  .btn-baixar:disabled { opacity: 0.5; cursor: not-allowed; }
  .btn-baixar .play { width: 0; height: 0; border-top: 6px solid transparent; border-bottom: 6px solid transparent; border-left: 9px solid #0d1410; }

  .painel-status { margin-top: 26px; display: none; }
  .painel-status.visivel { display: block; }

  .resumo {
    font-family: 'JetBrains Mono', monospace;
    font-size: 13px;
    padding: 10px 14px;
    border-radius: 6px;
    background: var(--panel-2);
    border: 1px solid var(--line);
    margin-bottom: 14px;
  }
  .resumo.info { color: var(--muted); }
  .resumo.sucesso { color: var(--accent); border-color: rgba(110,231,176,0.35); }
  .resumo.erro { color: var(--red); border-color: rgba(255,107,107,0.3); }

  .lista {
    border: 1px solid var(--line);
    border-radius: 8px;
    max-height: 340px;
    overflow-y: auto;
  }
  .item {
    display: flex;
    align-items: flex-start;
    gap: 10px;
    padding: 11px 14px;
    border-bottom: 1px solid var(--line);
    font-size: 13.5px;
    line-height: 1.4;
  }
  .item:last-child { border-bottom: none; }
  .icone { font-family: 'JetBrains Mono', monospace; font-size: 13px; width: 16px; flex-shrink: 0; margin-top: 1px; }
  .item.pendente .icone { color: var(--amber); }
  .item.sucesso .icone { color: var(--accent); }
  .item.erro .icone { color: var(--red); }
  .item .texto strong { font-weight: 500; }
  .item .motivo { color: var(--muted); font-size: 12px; display: block; margin-top: 2px; }
  .item a.baixar-link {
    margin-left: auto;
    color: var(--accent);
    text-decoration: none;
    font-family: 'JetBrains Mono', monospace;
    font-size: 11.5px;
    white-space: nowrap;
    flex-shrink: 0;
  }

  .acoes { display: flex; gap: 10px; margin-top: 14px; }
  .acoes button {
    flex: 1;
    background: var(--panel-2);
    border: 1px solid var(--line);
    color: var(--text);
    border-radius: 6px;
    padding: 11px 0;
    font-family: 'JetBrains Mono', monospace;
    font-size: 12.5px;
    cursor: pointer;
  }
  .acoes button:hover { border-color: var(--accent); }
  .acoes button:disabled { opacity: 0.35; cursor: not-allowed; }

  .rodape {
    margin-top: 30px;
    color: #565b66;
    font-size: 12px;
    text-align: center;
    line-height: 1.6;
  }
</style>
</head>
<body>
<div class="wrap">

  <div class="eyebrow"><span class="dot"></span>spool // downloader de áudio</div>
  <h1>Cola o link, eu cuido do resto.</h1>
  <div class="sub">Funciona com faixas, álbuns e playlists do SoundCloud. Faixas com DRM aparecem marcadas, não travam o resto da fila.</div>

  <div class="meter" id="meter">
    <span></span><span></span><span></span><span></span><span></span>
    <span></span><span></span><span></span><span></span><span></span>
    <span></span><span></span><span></span><span></span><span></span>
    <span></span><span></span><span></span><span></span><span></span>
  </div>

  <div class="card">
    <label for="url">Link</label>
    <input type="text" id="url" placeholder="https://soundcloud.com/artista/faixa">

    <div class="row">
      <div class="col">
        <label>Formato</label>
        <div class="segmented" id="segFormato">
          <button data-valor="WAV" class="ativo">WAV</button>
          <button data-valor="MP3">MP3</button>
        </div>
      </div>
      <div class="col">
        <label>Bitrate</label>
        <select id="bitrate" disabled>
          <option>128 kbps</option>
          <option>192 kbps</option>
          <option selected>320 kbps</option>
        </select>
      </div>
    </div>

    <button class="btn-baixar" id="btnBaixar">
      <span class="play"></span> Baixar
    </button>

    <div class="painel-status" id="painelStatus">
      <div class="resumo info" id="resumo">Iniciando...</div>
      <div class="lista" id="lista"></div>
      <div class="acoes">
        <button id="btnZip" disabled>Baixar tudo (.zip)</button>
        <button id="btnCopiarErros" disabled>Copiar nomes com erro</button>
      </div>
    </div>
  </div>

  <div class="rodape">
    Uso pessoal entre amigos. Respeite os direitos dos artistas — o que tem DRM, tem DRM por um motivo.
  </div>
</div>

<script>
const segFormato = document.getElementById('segFormato');
const bitrateSel = document.getElementById('bitrate');
const btnBaixar = document.getElementById('btnBaixar');
const urlInput = document.getElementById('url');
const painelStatus = document.getElementById('painelStatus');
const resumoEl = document.getElementById('resumo');
const listaEl = document.getElementById('lista');
const meterEl = document.getElementById('meter');
const btnZip = document.getElementById('btnZip');
const btnCopiarErros = document.getElementById('btnCopiarErros');

let formatoAtual = 'WAV';
let jobIdAtual = null;
let pollTimer = null;

segFormato.querySelectorAll('button').forEach(b => {
  b.addEventListener('click', () => {
    segFormato.querySelectorAll('button').forEach(x => x.classList.remove('ativo'));
    b.classList.add('ativo');
    formatoAtual = b.dataset.valor;
    bitrateSel.disabled = formatoAtual === 'WAV';
  });
});

function icone(tipo) {
  if (tipo === 'pendente') return '···';
  if (tipo === 'sucesso') return '✔';
  return '✖';
}

function renderItens(itens, arquivos) {
  const mapaArquivos = {};
  arquivos.forEach(a => { mapaArquivos[a.titulo] = a.arquivo; });

  listaEl.innerHTML = '';
  itens.forEach(item => {
    const div = document.createElement('div');
    div.className = `item ${item.tipo}`;

    const spanIcone = document.createElement('span');
    spanIcone.className = 'icone';
    spanIcone.textContent = icone(item.tipo);

    const spanTexto = document.createElement('span');
    spanTexto.className = 'texto';
    spanTexto.innerHTML = `<strong>${item.nome}</strong>` + (item.motivo ? `<span class="motivo">${item.motivo}</span>` : '');

    div.appendChild(spanIcone);
    div.appendChild(spanTexto);

    if (item.tipo === 'sucesso' && mapaArquivos[item.nome] && jobIdAtual) {
      const link = document.createElement('a');
      link.className = 'baixar-link';
      link.href = `/download/${jobIdAtual}/${encodeURIComponent(mapaArquivos[item.nome])}`;
      link.textContent = 'baixar';
      div.appendChild(link);
    }

    listaEl.appendChild(div);
  });

  listaEl.scrollTop = listaEl.scrollHeight;
}

async function poll() {
  if (!jobIdAtual) return;
  const resp = await fetch(`/status/${jobIdAtual}`);
  if (!resp.ok) return;
  const dados = await resp.json();

  resumoEl.textContent = dados.resumo;
  resumoEl.className = `resumo ${dados.cor}`;

  renderItens(dados.itens, dados.arquivos);

  const temErro = dados.itens.some(i => i.tipo === 'erro');
  btnCopiarErros.disabled = !temErro;
  btnZip.disabled = dados.arquivos.length === 0;

  if (dados.finalizado) {
    clearInterval(pollTimer);
    meterEl.classList.remove('ativo');
    btnBaixar.disabled = false;
  }
}

btnBaixar.addEventListener('click', async () => {
  const url = urlInput.value.trim();
  if (!url) { urlInput.focus(); return; }

  btnBaixar.disabled = true;
  painelStatus.classList.add('visivel');
  meterEl.classList.add('ativo');
  resumoEl.textContent = 'Iniciando...';
  resumoEl.className = 'resumo info';
  listaEl.innerHTML = '';
  btnZip.disabled = true;
  btnCopiarErros.disabled = true;

  const resp = await fetch('/iniciar', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ url, formato: formatoAtual, bitrate: bitrateSel.value }),
  });

  if (!resp.ok) {
    resumoEl.textContent = 'Não deu pra iniciar o download. Confere o link.';
    resumoEl.className = 'resumo erro';
    btnBaixar.disabled = false;
    meterEl.classList.remove('ativo');
    return;
  }

  const dados = await resp.json();
  jobIdAtual = dados.job_id;

  pollTimer = setInterval(poll, 1200);
  poll();
});

btnZip.addEventListener('click', () => {
  if (jobIdAtual) window.location.href = `/download_zip/${jobIdAtual}`;
});

btnCopiarErros.addEventListener('click', async () => {
  const resp = await fetch(`/status/${jobIdAtual}`);
  const dados = await resp.json();
  const texto = dados.itens
    .filter(i => i.tipo === 'erro')
    .map(i => i.motivo ? `${i.nome} — ${i.motivo}` : i.nome)
    .join('\n');
  await navigator.clipboard.writeText(texto);
  btnCopiarErros.textContent = 'Copiado!';
  setTimeout(() => { btnCopiarErros.textContent = 'Copiar nomes com erro'; }, 1500);
});
</script>
</body>
</html>
