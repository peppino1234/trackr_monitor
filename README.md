name: Trackr Monitor

on:
  schedule:
    - cron: "*/30 * * * *"
  workflow_dispatch: {}

permissions:
  contents: write

jobs:
  check-trackr:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Installa dipendenze
        run: |
          pip install playwright beautifulsoup4
          playwright install chromium
          playwright install-deps chromium

      - name: Esegui Trackr Monitor
        env:
          TELEGRAM_BOT_TOKEN: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          TELEGRAM_CHAT_ID: ${{ secrets.TELEGRAM_CHAT_ID }}
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: python trackr_monitor.py

      - name: Salva stato
        run: |
          git config user.name "trackr-bot"
          git config user.email "actions@github.com"
          git add seen_positions.json || true
          git diff --staged --quiet && echo "Nessuna modifica" || (git commit -m "Aggiorna stato" && git push)
