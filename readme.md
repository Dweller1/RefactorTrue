# Лабораторні з реінжинірингу (8×)

[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Sane4ka126_1NetSdrClient1&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=Sane4ka126_1NetSdrClient1)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=Sane4ka126_1NetSdrClient1&metric=coverage)](https://sonarcloud.io/summary/new_code?id=Sane4ka126_1NetSdrClient1)
[![Bugs](https://sonarcloud.io/api/project_badges/measure?project=Sane4ka126_1NetSdrClient1&metric=bugs)](https://sonarcloud.io/summary/new_code?id=Sane4ka126_1NetSdrClient1)
[![Code Smells](https://sonarcloud.io/api/project_badges/measure?project=Sane4ka126_1NetSdrClient1&metric=code_smells)](https://sonarcloud.io/summary/new_code?id=Sane4ka126_1NetSdrClient1)
[![Vulnerabilities](https://sonarcloud.io/api/project_badges/measure?project=Sane4ka126_1NetSdrClient1&metric=vulnerabilities)](https://sonarcloud.io/summary/new_code?id=Sane4ka126_1NetSdrClient1)
[![Duplicated Lines (%)](https://sonarcloud.io/api/project_badges/measure?project=Sane4ka126_1NetSdrClient1&metric=duplicated_lines_density)](https://sonarcloud.io/summary/new_code?id=Sane4ka126_1NetSdrClient1)
[![Security Rating](https://sonarcloud.io/api/project_badges/measure?project=Sane4ka126_1NetSdrClient1&metric=security_rating)](https://sonarcloud.io/summary/new_code?id=Sane4ka126_1NetSdrClient1)
[![Maintainability Rating](https://sonarcloud.io/api/project_badges/measure?project=Sane4ka126_1NetSdrClient1&metric=sqale_rating)](https://sonarcloud.io/summary/new_code?id=Sane4ka126_1NetSdrClient1)
[![Reliability Rating](https://sonarcloud.io/api/project_badges/measure?project=Sane4ka126_1NetSdrClient1&metric=reliability_rating)](https://sonarcloud.io/summary/new_code?id=Sane4ka126_1NetSdrClient1)

Цей репозиторій використовується для курсу **реінжиніринг ПЗ**.
Мета — провести комплексний реінжиніринг спадкового коду NetSdrClient, включаючи рефакторинг архітектури, покращення якості коду, впровадження сучасних практик розробки та автоматизацію процесів контролю якості через CI/CD пайплайни.

---

## Структура 8 лабораторних

Кожна робота — **через Pull Request**. У PR додати короткий опис: _що змінено / як перевірити / ризики_ + звіт про хід виконання в Classroom.

### Лаба 1 — Підключення SonarCloud і CI

**Мета:** створити проект у SonarCloud, підключити GitHub Actions, запустити перший аналіз.

**Необхідно:**

- .NET 8 SDK
- Публічний GitHub-репозиторій
- Обліковка SonarCloud (організація прив’язана до GitHub)

**1) Підключити SonarCloud**

- На SonarCloud створити проект з цього репозиторію (_Analyze new project_).
- Згенерувати **user token** і додати в репозиторій як секрет **`SONAR_TOKEN`** (_Settings → Secrets and variables → Actions_).
- Додати/перевірити `.github/workflows/sonarcloud.yml` з тригерами на PR і push у основну гілку.
  `sonarcloud.yml`:

```yml
# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

# This workflow helps you trigger a SonarCloud analysis of your code and populates
# GitHub Code Scanning alerts with the vulnerabilities found.
# Free for open source project.

# 1. Login to SonarCloud.io using your GitHub account

# 2. Import your project on SonarCloud
#     * Add your GitHub organization first, then add your repository as a new project.
#     * Please note that many languages are eligible for automatic analysis,
#       which means that the analysis will start automatically without the need to set up GitHub Actions.
#     * This behavior can be changed in Administration > Analysis Method.
#
# 3. Follow the SonarCloud in-product tutorial
#     * a. Copy/paste the Project Key and the Organization Key into the args parameter below
#          (You'll find this information in SonarCloud. Click on "Information" at the bottom left)
#
#     * b. Generate a new token and add it to your Github repository's secrets using the name SONAR_TOKEN
#          (On SonarCloud, click on your avatar on top-right > My account > Security
#           or go directly to [https://sonarcloud.io/account/security/](https://sonarcloud.io/account/security/))

# Feel free to take a look at our documentation ([https://docs.sonarcloud.io/getting-started/github/](https://docs.sonarcloud.io/getting-started/github/))
# or reach out to our community forum if you need some help ([https://community.sonarsource.com/c/help/sc/9](https://community.sonarsource.com/c/help/sc/9))

name: SonarCloud analysis

on:
  push:
    branches: ["master"]
  pull_request:
    branches: ["master"]
  workflow_dispatch:

permissions:
  pull-requests: read # allows SonarCloud to decorate PRs with analysis results

jobs:
  sonar-check:
    name: Sonar Check
    runs-on: windows-latest # безпечно для будь-яких .NET проектів
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }

      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: "8.0.x"

      # 1) BEGIN: SonarScanner for .NET
      - name: SonarScanner Begin
        run: |
          dotnet tool install --global dotnet-sonarscanner
          echo "$env:USERPROFILE\.dotnet\tools" >> $env:GITHUB_PATH
          dotnet sonarscanner begin `
          /d:sonar.projectKey="<Project Key>" `
          /d:sonar.organization="<Organization Key>" `
          /d:sonar.token="${{ secrets.SONAR_TOKEN }}" `
          /d:sonar.cs.opencover.reportsPaths="**/coverage.xml" `
          /d:sonar.cpd.cs.minimumTokens=40 `
          /d:sonar.cpd.cs.minimumLines=5 `
          /d:sonar.exclusions=**/bin/**,**/obj/**,**/sonarcloud.yml `
          /d:sonar.qualitygate.wait=true
        shell: pwsh
      # 2) BUILD & TEST
      - name: Restore
        run: dotnet restore NetSdrClient.sln
      - name: Build
        run: dotnet build NetSdrClient.sln -c Release --no-restore
      #- name: Tests with coverage (OpenCover)
      #  run: |
      #    dotnet test NetSdrClientAppTests/NetSdrClientAppTests.csproj -c Release --no-build `
      #      /p:CollectCoverage=true `
      #      /p:CoverletOutput=TestResults/coverage.xml `
      #      /p:CoverletOutputFormat=opencover
      #  shell: pwsh
      # 3) END: SonarScanner
      - name: SonarScanner End
        run: dotnet sonarscanner end /d:sonar.token="${{ secrets.SONAR_TOKEN }}"
        shell: pwsh
```
