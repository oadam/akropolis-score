# Akropolis Score

A score counter for the board game [Akropolis](https://www.gigamic.com/jeu/akropolis).

## Local development

**Prerequisites:** [nvm](https://github.com/nvm-sh/nvm)

```bash
# Load nvm (add this to your shell profile if not already there)
export NVM_DIR="$HOME/.config/nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Install dependencies
npm install

# Start the dev server
npm run dev
```

Then open [http://localhost:5173](http://localhost:5173).

The dev server supports hot module replacement — edits to `src/App.vue` reflect instantly in the browser without a full reload.

## Build

```bash
npm run build   # output goes to dist/
npm run preview # serve the production build locally
```

## Deployment

Pushes to `main` automatically deploy to GitHub Pages via the workflow in `.github/workflows/deploy.yml`.

To enable it, go to **Settings → Pages → Source** and select **GitHub Actions**.
