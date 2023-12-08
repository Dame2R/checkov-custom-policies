#   Checkov Nutzung

**Schritt-für-Schritt Anleitung**
#   a. Lokale Maschine

1.  **Installation von Checkov:**

`pip install checkov`

2.  **Ausführen von Checkov:**
   - Für aktuellen Ordner: `checkov -d .`
   - Für beliebigen Ordner: `checkov --directory DIRECTORY `
   - Für spezifische Datei: `checkov --file /path/to/file.yml`

3.  **Konfiguration:**
   - Definieren des Schweregrads: `--soft-fail-on SEVERITY`
   - Integration von Bridgecrew: `--bc-api-key API_KEY`
   - Festlegung des Frameworks: `--framework FRAMEWORK`
   - Nutzung des Rule Repositories (wenn Bridgecrew Account keine Custom Rules enthält): `--external-checks-git GIT_RULE_REPOSITORY`

4.  **Konsolidierter Command**

`checkov -d . --soft-fail-on MEDIUM --bc-api-key API_KEY --framework cloudformation --external-checks-git GIT_RULE_REPOSITORY` 


#   b. CI/CD Pipeline

1.  **Dockerfile ohne Dateiendung anlegen und befüllen:**

    *Dockerfile:*
    
    #Verwendung des offiziellen Python-Images als Basis
    
    FROM python:3.9-slim
    
    #Installiere Checkov, Boto3 (Security Hub Reporting; optional) und AWS CLI
    
    RUN pip install checkov boto3
    
    RUN apt-get update && apt-get install -y awscli docker.io
    
    #Installiere weitere Befehle und Abhängigkeiten; Lösche redundante Dateien
    
    RUN apt-get update && apt-get install -y \
    
    gettext-base \
    
    && rm -rf /var/lib/apt/lists/
    

3.	**Build des Dockers**
   
`docker build PATH_TO_DOCKERFILE -t tag/description:version (bspw. iac/checkov:latest)`

4.  **Push des Docker images**
   
`docker push tag/description:version`

5.  **Einbindung des Dockers in Buildspec.yml**

`docker pull tag/description:version`

6.  **Ausführung des Docker Images in Buildspec.yml mit Aufruf des Testskripts**
    
`docker run --rm -v $CODEBUILD_SRC_DIR:/src tag/description:version bash /src/security/iac-tests.sh`


**Shellscript**

7.  **Nutzung eines exit code Handlers**
   
`final_exit_code=0`

8.  **Bypass des automatischen Exits wenn ein System Exit-Code von 1 erreicht wird; nur für den Checkov Befehl**
   
`set +e`

9.  **Aufruf von Checkov mit JUNITXML Report für CodeBuild**

`checkov -d . --soft-fail-on MEDIUM --bc-api-key API_KEY --repo-id User/bc-repo-id -o junitxml > checkov-report.xml --framework cloudformation`

10.  **Zwischenspeicherung des Exit-Codes**

`exit_code=$?`

11. **Deaktivierung des Bypasses**

`set -e`

12. **Aktualisieren des finalen Exit-Codes, falls Checkov einen Fehler gefunden hat**

if [ $exit_code -ne 0 ]; then
 
  final_exit_code=$exit_code

fi

14. **Gebe den Exit Code am Ende erst zurück um einen frühzeitigen Abbruch zu verhindern**

`exit $final_exit_code`

