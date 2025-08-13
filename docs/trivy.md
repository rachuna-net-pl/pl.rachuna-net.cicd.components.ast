# 🛠 Job template: 🔬 Trivy image scan (DAST)

> [!NOTE]
>
> Job **`Trivy image scan (DAST)`** umożliwia **automatyczne skanowanie obrazów kontenerów** pod kątem luk bezpieczeństwa przy użyciu narzędzia **Trivy**.
> Szablon można uruchamiać w etapie `integration-test`, aby upewnić się, że obraz nie zawiera znanych podatności o poziomie **HIGH** lub **CRITICAL**.

---
## Wymagania wstępne

* [Trivy](https://aquasecurity.github.io/trivy/) – wbudowany w używany obraz Dockera (ustawiany przez `inputs.docker_image`).
* Obraz kontenera, który ma zostać przeskanowany (ustawiany przez `inputs.container_to_analyze`).

---
## ⚙️ Parametry wejściowe (`inputs`)

| Nazwa                  | Typ    | Domyślna wartość                                                                            | Opis                                       |
| ---------------------- | ------ | ------------------------------------------------------------------------------------------- | ------------------------------------------ |
| `docker_image`         | string | `registry.gitlab.com/pl.rachuna-net/containers/trivy:1.0.0`                                 | Obraz Dockera zawierający narzędzie Trivy  |
| `logo_url`             | string | `https://gitlab.com/pl.rachuna-net/cicd/gitlab-ci/-/raw/main/_configs/_logo?ref_type=heads` | Adres URL logo wyświetlanego w logach      |
| `container_to_analyze` | string | *(puste)*                                                                                   | Pełna nazwa obrazu, który ma być skanowany |

---
## 🧬 Zmienne środowiskowe ustawiane w jobie

* `CONTAINER_IMAGE_TRIVY` – obraz Dockera Trivy (na podstawie `inputs.docker_image`)
* `CONTAINER_TO_ANALYZE` – nazwa obrazu kontenera do przeskanowania (na podstawie `inputs.container_to_analyze`)
* `TRIVY_NO_PROGRESS` – wyłącza pasek postępu w logach (domyślnie `true`)
* `TRIVY_CACHE_DIR` – lokalny katalog cache Trivy (domyślnie `.trivycache/`)

---
## 📤 Output

Job **wykonuje następujące kroki**:

1. Uruchomienie skanowania obrazu kontenera (`CONTAINER_TO_ANALYZE`) z użyciem **Trivy**:

   * Najpierw wyszukiwanie luk o poziomie **HIGH** – raport ostrzegawczy (`--exit-code 0`).
   * Następnie wyszukiwanie luk o poziomie **CRITICAL** – pipeline zakończy się niepowodzeniem, jeśli takie wystąpią (`--exit-code 1`).
2. Wykorzystanie cache Trivy w katalogu `.trivycache/` w celu przyspieszenia kolejnych skanów.

---
## 🧪 Dostępny job

### **🔬 trivy (dast)**

Skanuje wskazany obraz kontenera i kończy pipeline, jeśli znajdzie luki o poziomie **CRITICAL**:

```yaml
🔬 trivy (dast):
  image: $CONTAINER_IMAGE_TRIVY
  stage: integration-test
  services:
    - name: $CONTAINER_IMAGE_TRIVY
      alias: trivy-dind
  allow_failure: true
  before_script:
    - git config --global --add safe.directory ${CI_PROJECT_DIR}
    - [ ! -z "${LOGO_URL}" ] && curl -s "${LOGO_URL}"
    - !reference [.ast:trivy_input_variables]
  script:
    - trivy image --exit-code 0 --severity HIGH $CONTAINER_TO_ANALYZE
    - trivy image --exit-code 1 --severity CRITICAL $CONTAINER_TO_ANALYZE
  cache:
    paths:
      - .trivycache/
```

---
## 🧪 Przykład użycia w pipeline

```yaml
include:
  - component: $CI_SERVER_FQDN/pl.rachuna-net/cicd/components/ast/trivy@$COMPONENT_VERSION_TRIVY

🔬 trivy (dast):
  allow_failure: true
  variables:
    CONTAINER_TO_ANALYZE: $CI_REGISTRY_IMAGE:$RELEASE_CANDIDATE_VERSION
  rules:
    - when: on_success
```
