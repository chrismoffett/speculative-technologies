# Speculative Technologies — MSTU 5199
A card-based prototyping tool for the Speculative Technologies final project.

## Deploying to GitHub Pages (two-hour setup)

### Step 1: Create your GitHub repo
1. Go to github.com → New repository
2. Name it `speculative-technologies` (or whatever you want)
3. Set it Public
4. Don't initialize with README

### Step 2: Update the base path
In `vite.config.js`, change the `base` to match your repo name:
```js
base: '/your-repo-name/',
```

### Step 3: Push the source
```bash
cd tftf-course
git init
git add .
git commit -m "initial"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
git push -u origin main
```

### Step 4: Build and deploy
Install the deploy tool once:
```bash
npm install --save-dev gh-pages
```

Add to `package.json` scripts:
```json
"deploy": "vite build && gh-pages -d dist"
```

Then run:
```bash
npm run deploy
```

### Step 5: Enable GitHub Pages
- Go to your repo → Settings → Pages
- Source: Deploy from branch → `gh-pages` branch → `/root`
- Save. Your site will be live at: `https://YOUR-USERNAME.github.io/YOUR-REPO/`

### Updating later
Just run `npm run deploy` again after any changes.

## Customizing the card data
All card content is in `src/cards.js`. Each category is an array of strings — add, remove, or edit freely.

## Local development
```bash
npm install
npm run dev
```
