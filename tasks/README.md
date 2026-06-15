# Workshop - modul 4: supply chain security 

Temaet for denne workshoppen er sikker kode og pipeline herding. Vi skal gå gjennom tre temaer: utviklingsmiljø, pipeline og git repository. Det vil være oppgaver knyttet til hver av disse temaene.

Krav: 
- GitHub bruker
- (Valgfritt) IDE
    - Oppgavene som krever kodeendring, kan enten gjøres lokalt på din maskin, eller bruke GitHub Desktop.

### Oppsett
1. Gi GitHub brukernavnet til fasilitator, slik at dere kan bli invitert til organisasjonen. 
2. Godkjenn invitasjonen
3. Gå til [cnapp-module](https://github.com/msilabben/cnapp-module)
4. Klikk på "Fork" inne på cnapp-module repoet, oppe til høyre
5. Velg organisasjonen som eier.
6. Trykk på "<> Code", og enten clone ned repoet lokalt på maskinen din eller trykk på headeren "Codespaces" og velg "Create codespace on main". 


Si ifra til fasilitator hvis dere møter på problemer. 


## Oppgaver

### Forke M4 repo
1. Gå til msilabben og fork repoet som heter: cnapp-module-4-builder
2. Velg msilabben som miljø 
3. Gå til "Actions" og godta at workflows får lov til å kjøre. 
4. Velg "pull-request" på høyre side og manuaelt sett igang workflowen. Vent til den har kjørt ferdig. 
5. Velg "publish" på høyre side, og manuelt sett igang workflowen. 
6. Gå til organisasjonssiden "msilabben", og velg "packages" i menyen på toppen. Ser du de tilhørende imagene du publiserte?

### Verifiser signatur
1. Åpne et nytt codespace ved å gå til hjemmeområdet til ditt M4 repository, klikk på "Code", deretter "Create new codespace on main"
2. Log in på GHCR med docker cli
```bash
gh auth token | docker login ghcr.io --username $(gh api user -q ".login") --password-stdin
```
3. Retrieve and verify the SBoM
```bash
cosign verify-attestation \
  --certificate-identity "https://github.com/msilabben/<your repository name>/.github/workflows/push-main.yml@refs/heads/main" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  --type spdxjson \
  --output json ghcr.io/msilabben/cnapp-module-4-builder/trivy:0.71.1 2>/dev/null | jq > result.json
```
4. Take a minute to reflect. What are we seeing in result.json?
5. Decode the payload and inspect the contents
```bash
jq -r '.payload' result.json | base64 -d > payload.json
```
6. What does the 'subject' and 'predicate' tell us in payload.json?

### Oppdate M2 pipelines med ny kilde
1. Gå til msilabben og fork repoet som heter: cnapp-module-2-application
2. Velg msilabben som miljø. 
3. Gå til "actions" og godta at workflows får lov til å kjøre. 
4. Åpne et workspace i dette repoet. Det gjøres fra hovedsiden til repoet: code -> create workspace. 
5. Lag en ny branch, og checkout inn i den nye branchen. 
6. Gå til filen ".github/workflows/pull-request.yml". 
7. Oppdater semgrep og trivy jobbene til å bruke det nye imaget. Bytt kilden til imaget. 
8. Gi workflowen tilgang til å lese fra packages ved å legge til "packages: read" under "premissions". 
9. Aktiver workflowene ved å sette "if: false" til "if: true" på både semgrep og trivy. 
9. Legg til filene, commit og push til repoet. 
10. Opprett en PR inn til (din) main, og se om workflowen kjører vellykket. 

### Oppdatere M2 pipelines med å knytte til sha
1. Gå tilbake til workspace, og til ".github/workflows/pull-request.yml". 
2. For imaget til semgrep, knytt imaget til sha istedenfor versjonsnummer. 
3. For imaget til trivy, knytt imaget til sha istedenfor versjonsnummer. 
4. Add, commit og push til repoet. 
5. Gå til den eksisterende PR'en (eller lag en ny) og se om workflowen kjører som den skal. 

### Sett begrensninger på npm og uv
1. Gå til workspacet for M2. 
2. Oppdater uv i backend til å ikke tillate pakker som er nyere enn 7 dager. (Oppdater backend/pyproject.toml under "[tool.uv]")
3. Oppdater npm i frontend til å ikke tillate pakker som er nyere enn 7 dager. (Legg til en ny fil ".npmrc", og en config linje i den filen)
4. Legg til, commit og push filene til repoet. 

### Lag OPA policy 
1. Gå til workspacet for M2.
2. Gå til ".github/workflows/pull-request.yml" og se på opa-jobben. 
3. Gå til "policy/semgrep.rego" og se på semgrep regelen. 
4. Med inspirasjon fra semgrep, prøv å lag en OPA-regel som kun tillater images fra msilabben sin ghcr, med pull-request workflowen som inputet. (Lag en ny .rego fil, oppdater pull-request.yml med regelen, og sett inputet til å være .github/workflow/pull-request.yml)