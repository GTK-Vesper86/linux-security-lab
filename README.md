## 📁 Was liegt hier rum?

| Datei | Was sie tut | Sicherheits-Lektion |
|-------|-------------|---------------------|
| `app.py` | Mini-Flask-App mit `/` und `/health` | Läuft auf Port 8080 (kein root nötig) |
| `requirements.txt` | Gepinnte Abhängigkeit | Reproduzierbar + von Dependabot überwachbar |
| `Dockerfile` | Baut ein abgesichertes Image | Multi-Stage, `USER`, schlankes Base-Image |
| `.dockerignore` | Hält Müll & Geheimnisse draußen | Kleinere Angriffsfläche im Build-Kontext |
| `.github/workflows/security.yml` | Linten, Bauen, Scannen | CI als automatischer Türsteher |
| `.github/dependabot.yml` | Wacht über Abhängigkeiten | Rauchmelder für veraltete Pakete |

---

## 🧱 Die Sicherheits-Entscheidungen im Klartext

### 1. Schlankes Base-Image (`python:3.12-slim`)
Statt eines fetten `ubuntu:latest` nehmen wir ein schlankes Image. Weniger
installierte Pakete bedeuten weniger potenzielle Schwachstellen. Anders
gesagt: Wer wenig Türen hat, muss weniger Schlösser kontrollieren.

### 2. Multi-Stage-Build
Stage 1 (`builder`) installiert die Abhängigkeiten. Stage 2 (`runtime`)
übernimmt nur das fertige Ergebnis. Der ganze Build-Ballast — Caches,
Werkzeuge — bleibt zurück und landet nie im ausgelieferten Image.

### 3. Nicht-root-Benutzer (`USER appuser`) ⭐
Die wichtigste einzelne Zeile. Standardmäßig läuft im Container alles als
`root`. Bricht jemand aus dem Container aus, steht er dann als `root` auf
dem Host. Mit einem eigenen, rechtearmen `appuser` ist ein Ausbrecher nur
ein entmachteter Gast statt ein Hausherr.

### 4. `.dockerignore`
Verhindert, dass `.git`, `.env` & Co. überhaupt in den Build-Kontext
geraten. Ein versehentlich mitkopierter API-Key ist einer der häufigsten
echten Vorfälle — hier wird er an der Tür abgewiesen.

### 5. `HEALTHCHECK`
Der Container meldet selbst, ob er gesund ist. Orchestrierer können kranke
Container so automatisch neu starten, statt stillschweigend kaputte
Instanzen weiterlaufen zu lassen.

---

## 🤖 Was die CI/CD-Pipeline automatisch erledigt

Bei jedem `push` und Pull Request laufen drei Jobs:

1. **Hadolint** — linted das Dockerfile und meckert bei schlechten Praktiken.
2. **Trivy** — scannt das gebaute Image auf bekannte CVEs (`HIGH`/`CRITICAL`).
3. **Demonstrate** — zeigt live im Log, dass der Container als `uid=10001`
   und eben *nicht* als root läuft.

> 💡 Der Trivy-Scan steht auf `exit-code: '0'` — er **berichtet** nur, statt
> den Build abzubrechen. Sobald du dich sicher fühlst, stell ihn auf `'1'`:
> Dann wird aus dem freundlichen Hinweis ein echtes Stoppschild.

---

## 🚀 Loslegen

```bash
# 1. Repo lokal bauen und testen (falls Docker installiert ist)
docker build -t security-lab .
docker run --rm -p 8080:8080 security-lab
# -> http://localhost:8080  und  http://localhost:8080/health

# 2. Beweis antreten: Wer läuft da drin?
docker run --rm security-lab id
# Erwartet: uid=10001(appuser) gid=10001(appgroup)
```

Danach: Repo zu GitHub pushen, im Tab **Actions** der Pipeline beim Arbeiten
zusehen und im Tab **Security** Dependabot + Scans aktivieren.

---

## 🎯 Nächste Ausbaustufen (wenn dir langweilig wird)

- [ ] Trivy-`exit-code` auf `'1'` setzen und sehen, wie der Build bei einem
      verwundbaren Paket bewusst scheitert.
- [ ] Eine veraltete Flask-Version eintragen und Dependabot beim Vorschlagen
      eines Updates beobachten.
- [ ] `--cap-drop=ALL` beim `docker run` ausprobieren und schauen, was noch
      funktioniert.
- [ ] CodeQL-Scanning für `app.py` aktivieren.

Viel Spaß — und denk dran: Das beste Türschloss nützt nichts, wenn der
Schlüssel unter der Fußmatte (im Git-Repo) liegt. 🗝️
