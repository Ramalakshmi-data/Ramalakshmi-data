# Keeps the "Latest from my GitHub" list in README.md up to date.
# Runs every day, and you can also run it by hand from the Actions tab.
name: Update README

on:
  schedule:
    - cron: "17 20 * * *"   # daily (early morning in Adelaide)
  workflow_dispatch:         # adds a "Run workflow" button
  push:
    branches: [main]
    paths: [".github/workflows/update-readme.yml"]

permissions:
  contents: write

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build latest-projects list
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          USER: ${{ github.repository_owner }}
        run: |
          python3 - <<'PY'
          import json, os, re, urllib.request

          user = os.environ["USER"]
          hide = {f"{user}.github.io".lower(), user.lower()}

          req = urllib.request.Request(
              f"https://api.github.com/users/{user}/repos?per_page=100&sort=pushed",
              headers={"Authorization": f"Bearer {os.environ['GH_TOKEN']}",
                       "Accept": "application/vnd.github+json"})
          repos = json.load(urllib.request.urlopen(req))
          repos = [r for r in repos
                   if not r["fork"] and not r["archived"] and r["name"].lower() not in hide]
          repos.sort(key=lambda r: r["pushed_at"], reverse=True)
          repos = repos[:6]

          def nice(name):
              return re.sub(r"[-_]+", " ", name).title()

          if repos:
              rows = ["| Project | Description | Language | Updated |", "|---|---|---|---|"]
              for r in repos:
                  desc = (r["description"] or "—").replace("|", "\\|")
                  rows.append(f"| [**{nice(r['name'])}**]({r['html_url']}) | {desc} | "
                              f"{r['language'] or '—'} | {r['pushed_at'][:10]} |")
              block = "\n".join(rows)
          else:
              block = "_New projects will appear here as I publish them._"

          path = "README.md"
          text = open(path, encoding="utf-8").read()
          new = re.sub(r"(<!-- LATEST_REPOS:START -->).*?(<!-- LATEST_REPOS:END -->)",
                       lambda m: f"{m.group(1)}\n{block}\n{m.group(2)}", text, flags=re.S)
          open(path, "w", encoding="utf-8").write(new)
          PY

      - name: Commit if changed
        run: |
          if git diff --quiet; then echo "No changes"; exit 0; fi
          git config user.name  "github-actions[bot]"
          git config user.email "41898283+github-actions[bot]@users.noreply.github.com"
          git commit -am "Update latest projects in README"
          git push
