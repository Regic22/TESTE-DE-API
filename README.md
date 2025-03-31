from flask import Flask, request, jsonify
import pandas as pd
import requests
import os
from apscheduler.schedulers.background import BackgroundScheduler

app = Flask(__name__)
CSV_URL = "https://dadosabertos.ans.gov.br/FTP/PDA/operadoras_de_plano_de_saude_ativas/"
CSV_FILE = "operadoras.csv"

def download_csv():
    response = requests.get(CSV_URL)
    if response.status_code == 200:
        with open(CSV_FILE, 'wb') as f:
            f.write(response.content)
        print("CSV atualizado com sucesso.")
    else:
        print("Erro ao baixar o CSV.")

def load_data():
    if os.path.exists(CSV_FILE):
        return pd.read_csv(CSV_FILE, sep=';', encoding='latin1')
    return pd.DataFrame()

data = load_data()

@app.route("/buscar", methods=["GET"])
def buscar():
    termo = request.args.get("termo", "").lower()
    if termo:
        resultados = data[data.apply(lambda row: termo in str(row).lower(), axis=1)]
        return jsonify(resultados.to_dict(orient="records"))
    return jsonify([])

scheduler = BackgroundScheduler()
scheduler.add_job(download_csv, "interval", days=1)
scheduler.start()

download_csv()
app.run(debug=True)

