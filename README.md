# devops-demo

## Was macht das Projekt?
Dieses Repository ist eine Lern-Demo für CI/CD mit GitHub Actions. Es enthält eine kleine Python-Anwendung mit Grundrechenarten-Funktionen (`summe`, `differenz`) samt Tests sowie zwei Workflows, die Build, Test und ein simuliertes Deployment über GitHub Actions zeigen.

## Pipeline im Überblick
Der Haupt-Workflow **CI** (`.github/workflows/ci.yml`) besteht aus zwei Jobs:

1. **test** – läuft bei jedem Lauf:
   - Repository auschecken und Lauf-Informationen anzeigen
   - Python installieren (Version aus `env.PYTHON_VERSION`)
   - Pip-Abhängigkeiten installieren (mit Cache)
   - Tests mit `pytest` ausführen
   - Versionsnummer ermitteln (`1.0.<run_number>`)
   - Anwendung als `build/app.zip` bauen
   - Build-Artifact hochladen (`app-paket`)

2. **deploy** – läuft nur nach erfolgreichem `test`-Job und nur auf `main`:
   - Repository auschecken
   - Build-Artifact herunterladen und Inhalt prüfen
   - Deployment-Ziel anzeigen
   - Deploy-Secret prüfen (nur Länge, kein Wert)
   - GitHub Release erstellen und `app.zip` als Asset anhängen

Zusätzlich existiert der Demo-Workflow `hello.yaml` mit drei sequenziellen Jobs (`information` → `second-job` → `third-job`), der Job-Abhängigkeiten (`needs`) und Bedingungen (`if`) veranschaulicht.

## Trigger
- **CI** (`ci.yml`): Push auf `main`, Pull Request gegen `main`, sowie manuell per `workflow_dispatch`.
- **hello.yaml**: Push auf jeden Branch, sowie manuell per `workflow_dispatch`. Der `second-job` läuft dabei nur auf `main`.

## Secrets und Environment
**Secrets:**
- `DEPLOY_TOKEN` – wird im `deploy`-Job geprüft (nur Länge, kein Wert wird ausgegeben)
- `GITHUB_TOKEN` – von GitHub automatisch bereitgestellt, wird zum Erstellen des Releases genutzt

**Variablen:**
- `DEPLOY_TARGET` (Repository-/Environment-Variable) – wird beim Deployment angezeigt
- `PYTHON_VERSION`, `APP_ENV` – als Workflow-`env` definiert, keine Secrets

**Environment:**
- Der `deploy`-Job läuft in der Environment `production`. Falls dafür in den Repository-Settings Protection Rules hinterlegt sind (z. B. Required Reviewers oder Wartezeiten), müssen diese vor dem Deployment bestätigt werden.

## Deployment
Das Deployment läuft nur bei einem Push auf `main`, nachdem die Tests erfolgreich waren. Es wird ein GitHub Release mit dem Tag `v1.0.<run_number>` erstellt, das gebaute `app.zip` wird als Asset angehängt. Geprüft werden kann das Ergebnis auf der **Releases-Seite** des Repositories – dort erscheint die neue Version mit der Notiz „Automatisches Deployment aus GitHub Actions“.

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
