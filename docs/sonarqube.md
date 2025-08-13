# 🛠 Job template: 💪 SonarQube Scanner (SAST)

> [!note]
>
> Job **`SonarQube Scanner (SAST)`** umożliwia **analizę jakości kodu źródłowego oraz wykrywanie błędów** z wykorzystaniem platformy **SonarQube/SonarCloud**. Szablon należy uruchamiać w etapie `sast`, aby mieć pewność, że jakość kodu spełnia wymagania organizacji.

---
## Wymagania wstępne

* [SonarQube Scanner](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner/) – dostępny w obrazie Dockera ustawianym przez `inputs.docker_image`.
* Skonfigurowane konto w [SonarCloud](https://sonarcloud.io/) lub własnej instancji SonarQube (parametry: `sonar_host_url`, `sonar_token`, `sonar_project_key`).

---
## ⚙️ Parametry wejściowe (`inputs`)

| Nazwa                       | Typ    | Domyślna wartość                                                                            | Opis                                                |
| --------------------------- | ------ | ------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| `docker_image`              | string | `registry.gitlab.com/pl.rachuna-net/containers/sonar-scanner:2.1.0`                         | Obraz Dockera zawierający SonarQube Scanner         |
| `logo_url`                  | string | `https://gitlab.com/pl.rachuna-net/cicd/gitlab-ci/-/raw/main/_configs/_logo?ref_type=heads` | Adres URL logo wyświetlanego w logach               |
| `sonar_user_home`           | string | `${CI_PROJECT_DIR}/.sonar`                                                                  | Lokalny katalog roboczy SonarQube                   |
| `sonar_host_url`            | string | `https://sonarcloud.io`                                                                     | Adres instancji SonarQube lub SonarCloud            |
| `sonar_organization`        | string | `pl-rachuna-net`                                                                            | Nazwa organizacji w SonarCloud                      |
| `sonar_project_name`        | string | *(puste)*                                                                                   | Nazwa projektu analizowanego w SonarQube            |
| `sonar_project_key`         | string | *(puste)*                                                                                   | Klucz projektu w SonarQube (unikalny identyfikator) |
| `sonar_token`               | string | *(puste)*                                                                                   | Token uwierzytelniający do SonarQube                |
| `sonar_project_description` | string | *(puste)*                                                                                   | Opis projektu w SonarQube                           |
| `sonar_project_version`     | string | `1.0.0`                                                                                     | Wersja analizowanego projektu                       |
| `sonar_sources`             | string | `.`                                                                                         | Ścieżka do katalogu ze źródłami do analizy          |

---
## 🧬 Zmienne środowiskowe ustawiane w jobie

* `CONTAINER_IMAGE_SONAR_SCANNER` – obraz Dockera SonarQube Scanner (na podstawie `inputs.docker_image`)
* `SONAR_USER_HOME` – katalog roboczy SonarQube
* `SONAR_HOST_URL` – adres instancji SonarQube/SonarCloud
* `SONAR_ORGANIZATION` – nazwa organizacji
* `SONAR_PROJECT_KEY` – klucz projektu w SonarQube
* `SONAR_PROJECT_NAME` – nazwa projektu
* `SONAR_PROJECT_DESCRIPTION` – opis projektu
* `SONAR_PROJECT_VERSION` – wersja projektu
* `SONAR_SOURCES` – ścieżki źródeł do analizy
* `SONAR_TOKEN` – token uwierzytelniający do SonarQube

---
## 📤 Output

Job **wykonuje następujące kroki**:

2. Wyświetlenie zmiennych wejściowych i konfiguracji SonarQube.
3. Uruchomienie **SonarQube Scanner** z zadeklarowanymi parametrami:

   * analiza kodu z podanych źródeł (`SONAR_SOURCES`),
   * przesłanie wyników do wskazanej instancji (`SONAR_HOST_URL`).

Pipeline zakończy się niepowodzeniem, jeśli analiza SonarQube wykryje błędy jakościowe powodujące blokadę (quality gate failure).

---
## 🧪 Dostępny job

### **💪 sonarqube scanner (sast)**

Analizuje kod źródłowy i przesyła wyniki do SonarQube/SonarCloud:

```yaml
💪 sonarqube scanner:
  stage: sast
  image: $CONTAINER_IMAGE_SONAR_SCANNER
  before_script:
    - git config --global --add safe.directory ${CI_PROJECT_DIR}
    - [ ! -z "${LOGO_URL}" ] && curl -s "${LOGO_URL}"
    - !reference [.ast:_function_print_row]
    - !reference [.ast:sonarqube_input_variables]
    - !reference [.ast:sonarqube-init]
  script:
    - |
      sonar-scanner \
        -Dsonar.projectKey="${SONAR_PROJECT_KEY}" \
        -Dsonar.organization="${SONAR_ORGANIZATION}" \
        -Dsonar.projectName="${SONAR_PROJECT_NAME}" \
        -Dsonar.projectDescription="${CI_PROJECT_DESCRIPTION}" \
        -Dsonar.projectVersion="${SONAR_PROJECT_VERSION}" \
        -Dsonar.sources="${SONAR_SOURCES}" \
        -Dsonar.sourceEncoding=UTF-8 \
        -Dsonar.token="${SONAR_TOKEN}"
  rules:
    - when: on_success
```

---
## 🧪 Przykład użycia w pipeline

```yaml
include:
  - component: $CI_SERVER_FQDN/pl.rachuna-net/cicd/components/ast/sonarqube@$COMPONENT_VERSION_SONARQUBE

💪 sonarqube scanner:
  variables:
    SONAR_PROJECT_KEY: $CI_PROJECT_NAME
    SONAR_PROJECT_NAME: "Project $CI_PROJECT_NAME"
    SONAR_TOKEN: $SONAR_TOKEN
  rules:
    - when: on_success
```