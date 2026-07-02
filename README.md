import os
import re
import uuid
import zipfile
import subprocess
import threading
from pathlib import Path

from flask import Flask, request, jsonify, send_file, render_template, abort

app = Flask(__name__)

BASE_DIR = Path("/tmp/scdl_jobs")
BASE_DIR.mkdir(parents=True, exist_ok=True)

JOBS = {}
JOBS_LOCK = threading.Lock()

# Captura linhas como: ERROR: [soundcloud] 2300024921: This video is DRM protected
REGEX_ERRO = re.compile(r'^ERROR:\s*(?:\[(?P<site>[\w:]+)\]\s*(?P<id>[^:]+):\s*)?(?P<msg>.+)$')


def obter_mapa_titulos(url):
    """Busca leve (sem resolver formatos) para traduzir id -> título.
    Funciona mesmo para faixas com DRM, já que não checa disponibilidade."""
    mapa = {}
    try:
        comando = [
            "yt-dlp", "--flat-playlist", "--skip-download", "--no-warnings",
            "--ignore-errors", "--print", "%(id)s\t%(title)s", url,
        ]
        resultado = subprocess.run(
            comando, capture_output=True, text=True,
            encoding="utf-8", errors="ignore", timeout=60,
        )
        for linha in resultado.stdout.splitlines():
            linha = linha.strip()
            if "\t" in linha:
                id_faixa, titulo = linha.split("\t", 1)
                if id_faixa and titulo:
                    mapa[id_faixa.strip()] = titulo.strip()
    except Exception:
        pass
    return mapa


def atualizar_job(job_id, **kwargs):
    with JOBS_LOCK:
        JOBS[job_id].update(kwargs)


def upsert_item(job_id, nome, tipo, motivo=None):
    """Adiciona ou atualiza uma linha na lista de status do job."""
    with JOBS_LOCK:
        job = JOBS[job_id]
        item = {"nome": nome, "tipo": tipo, "motivo": motivo}
        idx = job["indice"].get(nome)
        if idx is not None:
            job["itens"][idx] = item
        else:
            job["itens"].append(item)
            job["indice"][nome] = len(job["itens"]) - 1


def processar_download(job_id, url, formato, bitrate):
    pasta = BASE_DIR / job_id
    pasta.mkdir(parents=True, exist_ok=True)

    atualizar_job(job_id, resumo="Buscando lista de músicas...", cor="info")
    mapa_titulos = obter_mapa_titulos(url)
    atualizar_job(job_id, resumo="Baixando...", cor="info")

    tentativas = []
    sucesso_titulos = set()
    erro_geral = None
    total_erros = 0

    comando = [
        "yt-dlp", "--ignore-errors", "--no-warnings", "--no-color",
        "-f", "bestaudio", "-x",
        "-o", str(pasta / "%(artist)s - %(title)s.%(ext)s"),
        "--print", "before_dl:###INICIO###%(title)s",
        "--print", "after_move:###FIM###%(title)s|||%(filepath)s",
    ]
    if formato == "WAV":
        comando += ["--audio-format", "wav"]
    else:
        comando += ["--audio-format", "mp3", "--audio-quality", bitrate.replace(" kbps", "K")]
    comando.append(url)

    try:
        processo = subprocess.Popen(
            comando, stdout=subprocess.PIPE, stderr=subprocess.STDOUT,
            text=True, encoding="utf-8", errors="ignore", bufsize=1,
        )

        for linha in processo.stdout:
            linha = linha.strip()
            if not linha:
                continue

            if linha.startswith("###INICIO###"):
                titulo = linha.replace("###INICIO###", "").strip()
                if titulo not in tentativas:
                    tentativas.append(titulo)
                upsert_item(job_id, titulo, "pendente")

            elif linha.startswith("###FIM###"):
                resto = linha.replace("###FIM###", "").strip()
                if "|||" in resto:
                    titulo, caminho = resto.split("|||", 1)
                else:
                    titulo, caminho = resto, ""
                titulo = titulo.strip()
                sucesso_titulos.add(titulo)
                upsert_item(job_id, titulo, "sucesso")

                nome_arquivo = os.path.basename(caminho.strip()) if caminho.strip() else None
                if nome_arquivo:
                    with JOBS_LOCK:
                        JOBS[job_id]["arquivos"].append({"titulo": titulo, "arquivo": nome_arquivo})

            elif linha.startswith("ERROR:"):
                m = REGEX_ERRO.match(linha)
                if m:
                    id_bruto = (m.group("id") or "Faixa").strip()
                    msg = m.group("msg").strip()
                else:
                    id_bruto = "Faixa"
                    msg = linha.replace("ERROR:", "").strip()

                nome = mapa_titulos.get(id_bruto, id_bruto)
                total_erros += 1
                upsert_item(job_id, nome, "erro", msg)

        processo.wait()

    except FileNotFoundError:
        erro_geral = "yt-dlp não encontrado no servidor."
    except Exception as e:
        erro_geral = str(e)

    # Faixas que começaram mas nunca terminaram e não caíram em nenhum ERROR explícito
    falhas_titulo = [t for t in tentativas if t not in sucesso_titulos]
    for titulo in falhas_titulo:
        with JOBS_LOCK:
            idx = JOBS[job_id]["indice"].get(titulo)
            ja_marcado_erro = idx is not None and JOBS[job_id]["itens"][idx]["tipo"] == "erro"
        if not ja_marcado_erro:
            upsert_item(job_id, titulo, "erro", "Download não concluído")

    with JOBS_LOCK:
        total_falhas = sum(1 for i in JOBS[job_id]["itens"] if i["tipo"] == "erro")
    total_sucesso = len(sucesso_titulos)

    if erro_geral:
        resumo, cor = f"Erro: {erro_geral}", "erro"
    elif not tentativas and total_erros == 0:
        resumo, cor = "Nenhuma música foi processada. Verifique o link.", "erro"
    elif total_falhas == 0:
        resumo, cor = f"Concluído! {total_sucesso} música(s) baixada(s) com sucesso.", "sucesso"
    else:
        resumo, cor = f"Concluído: {total_sucesso} com sucesso, {total_falhas} com erro.", "erro"

    atualizar_job(job_id, resumo=resumo, cor=cor, finalizado=True)


@app.route("/")
def index():
    return render_template("index.html")


@app.route("/iniciar", methods=["POST"])
def iniciar():
    dados = request.get_json(force=True, silent=True) or {}
    url = (dados.get("url") or "").strip()
    formato = dados.get("formato", "MP3")
    bitrate = dados.get("bitrate", "320 kbps")

    if not url:
        return jsonify({"erro": "URL vazia"}), 400

    job_id = uuid.uuid4().hex[:12]
    with JOBS_LOCK:
        JOBS[job_id] = {
            "itens": [], "indice": {}, "arquivos": [],
            "resumo": "Iniciando...", "cor": "info", "finalizado": False,
        }

    threading.Thread(
        target=processar_download, args=(job_id, url, formato, bitrate), daemon=True
    ).start()

    return jsonify({"job_id": job_id})


@app.route("/status/<job_id>")
def status(job_id):
    with JOBS_LOCK:
        job = JOBS.get(job_id)
        if not job:
            return jsonify({"erro": "job não encontrado"}), 404
        return jsonify({
            "itens": job["itens"],
            "resumo": job["resumo"],
            "cor": job["cor"],
            "finalizado": job["finalizado"],
            "arquivos": job["arquivos"],
        })


@app.route("/download/<job_id>/<path:nome_arquivo>")
def download_arquivo(job_id, nome_arquivo):
    caminho = BASE_DIR / job_id / nome_arquivo
    if not caminho.exists():
        abort(404)
    return send_file(caminho, as_attachment=True)


@app.route("/download_zip/<job_id>")
def download_zip(job_id):
    pasta = BASE_DIR / job_id
    if not pasta.exists():
        abort(404)
    zip_path = BASE_DIR / f"{job_id}.zip"
    with zipfile.ZipFile(zip_path, "w") as zf:
        for arquivo in pasta.iterdir():
            if arquivo.is_file():
                zf.write(arquivo, arcname=arquivo.name)
    return send_file(zip_path, as_attachment=True, download_name="musicas.zip")


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=int(os.environ.get("PORT", 8000)), debug=False)
