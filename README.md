# ☀️ Wake HANA Bot – Automatically Start SAP HANA Cloud with GitHub Actions

Tired of logging into SAP BTP Cockpit every morning just to click “Start Instance” while your coffee is still brewing? Same here.

Welcome to **Wake HANA Bot** – the easiest way to auto-start your HANA Cloud instance so you can stay in bed 5 minutes longer 😴

---

## 🚀 What Does It Do?

This GitHub Actions workflow automatically starts your **SAP HANA Cloud** instance every morning, without manual clicks or terminal stress.

Because let’s be honest – nobody wants to type `cf update-service` before their first coffee.

---

## 🤔 Why This?

Sure, there are tons of ways to start HANA — CLI spells, cockpit clicking marathons, REST API calls, etc.
But sometimes, **GitHub Actions** is just the most elegant and visible way — especially when your repo is public and you want to share the joy 🎉

---

## ⏰ When Does It Run?

- Every morning at **09:00 UTC+3** ☀️
- Or manually via “Run Workflow” if you oversleep 😅

## 🔐 GitHub Secrets – Setup Guide

This project uses **repository secrets** to securely authenticate to your SAP BTP environment – no plain passwords in the workflow! 🕵️‍♀️

You need to define the following secrets:

| Secret Name           | Required? | Description                                                                     |
|-----------------------|-----------|---------------------------------------------------------------------------------|
| `CF_API`              | ✅        | Your Cloud Foundry API endpoint (e.g. `https://api.cf.eu20.hana.ondemand.com`) |
| `CF_USERNAME`         | ✅        | Your SAP BTP username (usually your email address)                             |
| `CF_PASSWORD`         | ✅        | Your SAP BTP password                                                          |
| `CF_ORG`              | ✅        | Your SAP BTP Organization Name                                                 |
| `CF_SPACE`            | ✅        | Your SAP BTP Space Name                                                        |  
| `CF_SERVICE_INSTANCE` | ✅        | The name of your HANA Cloud service instance (e.g. `hana-cloud-dev`)           |

---

### ➕ How to Add Secrets in GitHub

1. Go to your GitHub repository 🧭  
2. Click on **Settings**  
3. Navigate to **Secrets and variables > Actions**  
4. Click the **"New repository secret"** button  
5. Add each of the four secrets above one by one:
   - `CF_API`
   - `CF_USERNAME`
   - `CF_PASSWORD`
   - `CF_ORG`
   - `CF_SPACE`
   - `CF_SERVICE_INSTANCE`

> 🔒 Your secrets will be encrypted and will not appear in logs or UI – so keep them accurate!

---

Once your secrets are in place, you're ready to fly 🚀



---

## ⚙️ How It Works

1. Downloads and installs `cf` CLI
2. Logs into SAP BTP Cloud Foundry using secrets
3. Starts the HANA Cloud instance by setting `serviceStopped: false`

---

## 🔧 .github/workflows/start-hana.yml

```yaml
name: Start HANA Cloud Automatically

on:
  schedule:
    - cron: '0 6 * * *' # 09:00 UTC+3
  workflow_dispatch:

jobs:
  start-hana:
    runs-on: ubuntu-latest
    steps:
      - name: Download cf CLI
        run: |
          curl -L "https://github.com/cloudfoundry/cli/releases/latest/download/cf8-cli_8.14.1_linux_x86-64.tgz" -o cf8.tgz
          tar -xzf cf8.tgz
          chmod +x cf8
          mkdir -p ~/.local/bin
          cp cf8 ~/.local/bin/cf
          echo "$HOME/.local/bin" >> $GITHUB_PATH

      - name: Login to Cloud Foundry
        env:
          CF_API: ${{ secrets.CF_API }}
          CF_USERNAME: ${{ secrets.CF_USERNAME }}
          CF_PASSWORD: ${{ secrets.CF_PASSWORD }}
        run: |
          cf login -a "$CF_API" \
            -u "$CF_USERNAME" \
            -p "$CF_PASSWORD"

      - name: Wake up HANA 🐣
        env:
          CF_SERVICE_INSTANCE: ${{ secrets.CF_SERVICE_INSTANCE }}
        run: |
          echo "⏰ Waking up HANA like a sleepy cat..."
          cf update-service "$CF_SERVICE_INSTANCE" -c '{"data":{"serviceStopped":false}}' --wait
