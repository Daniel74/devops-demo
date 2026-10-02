# devops-demo

## Was macht das Projekt?
Dieses Repository ist eine Lern-Demo für CI/CD mit GitHub Actions. Es enthält eine kleine Python-Anwendung mit Grundrechenarten-Funktionen (`summe`, `differenz`) samt Tests sowie zwei Workflows, die Build, Test und ein simuliertes Deployment über GitHub Actions zeigen.

## Pipeline im Überblick
Der für die Abgabe relevante Workflow ist **Pipeline** (`.github/workflows/pipeline.yml`). Er besteht aus vier Jobs:

1. **test**:
   - Repository auschecken
   - Python installieren (Version `3.10`)
   - Pip-Abhängigkeiten installieren (mit Cache)
   - Tests mit `pytest` ausführen

2. **build** – läuft nur nach erfolgreichem `test`-Job:
   - Repository auschecken
   - Anwendung als `build/app.zip` bauen
   - Build-Artifact hochladen (`app-paket`)

3. **release** – läuft nur nach erfolgreichem `build`-Job und nur auf `main`:
   - Repository auschecken
   - Build-Artifact herunterladen
   - GitHub Release erstellen und `app.zip` als Asset anhängen

4. **deploy** – läuft nur nach erfolgreichem `release`-Job und nur auf `main`:
   - Deploy-Secret prüfen (nur Länge, kein Wert) als Platzhalter für ein echtes Deployment

Die weiteren Workflows (`ci.yml`, `hello.yaml`) dienten nur zum Experimentieren und sind in den GitHub-Repository-Einstellungen deaktiviert.

## Trigger
- **Pipeline** (`pipeline.yml`): Push auf `main` sowie Pull Request gegen `main`.

## Secrets und Environment
**Secrets:**
- `GITHUB_TOKEN` – von GitHub automatisch bereitgestellt, wird im `release`-Job zum Erstellen des Releases genutzt
- `DEPLOY_TOKEN` – wird im `deploy`-Job geprüft (nur Länge, kein Wert wird ausgegeben)

**Variablen:**
- keine

**Environment:**
- Der `deploy`-Job läuft in der Environment `production`. Falls dafür in den Repository-Settings Protection Rules hinterlegt sind (z. B. Required Reviewers oder Wartezeiten), müssen diese vor dem Deployment bestätigt werden.

## Deployment
Nach erfolgreichen Tests und Build wird bei einem Push auf `main` im `release`-Job ein GitHub Release mit dem Tag `v1.0.<run_number>` erstellt, das gebaute `app.zip` wird als Asset angehängt. Geprüft werden kann das Ergebnis auf der **Releases-Seite** des Repositories. Der anschließende `deploy`-Job simuliert das eigentliche Deployment (prüft nur die Länge von `DEPLOY_TOKEN`) und dient als Platzhalter für einen echten Deployment-Schritt.

## Lokal ausführen
```bash
# Virtuelle Umgebung anlegen und aktivieren
python3 -m venv .venv
source .venv/bin/activate

# Abhängigkeiten installieren
pip install -r requirements.txt

# Tests ausführen
python -m pytest -v

# Paket bauen (wie in der Pipeline)
mkdir -p build
python -m zipfile -c build/app.zip src/
```
