"""Mini-App fuer das Linux Security Lab.

Bewusst winzig gehalten: Es geht hier nicht um die App, sondern darum,
sie *sicher* zu verpacken und auszuliefern.
"""

from flask import Flask

app = Flask(__name__)


@app.get("/")
def index():
    return {"message": "Linux Security Lab laeuft – und zwar als Nicht-root."}


@app.get("/health")
def health():
    # Wird vom HEALTHCHECK im Dockerfile angesprochen.
    return {"status": "ok"}, 200


if __name__ == "__main__":
    # host="0.0.0.0", damit der Container von aussen erreichbar ist.
    # Port 8080 statt 80 -> Ports unter 1024 brauchen root-Rechte, die wir
    # bewusst nicht haben wollen.
    app.run(host="0.0.0.0", port=8080)
