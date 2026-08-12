# AIWise Cleaner Landing Page

Static marketing website for AIWise Cleaner (iOS + Android), built for GitHub Pages hosting.

## Files

- `index.html`: Page structure and content
- `styles.css`: Visual design and responsive layout
- `script.js`: Mobile menu, FAQ accordion, and reveal animations

## Run Locally

You can open `index.html` directly in your browser, or run a local server:

```bash
python3 -m http.server 8080
```

Then visit `http://localhost:8080`.

## Deploy to GitHub Pages

1. Push this folder to a GitHub repository.
2. In GitHub, open **Settings > Pages**.
3. Under **Build and deployment**, choose:
	- Source: `Deploy from a branch`
	- Branch: `main` (or your default branch)
	- Folder: `/ (root)`
4. Save and wait for deployment.

## Connect Custom Domain

1. In **Settings > Pages**, add your domain in **Custom domain**.
2. Create a `CNAME` file in the repository root with your domain as a single line:

```txt
nextstepailabs.com
```

3. Configure DNS at your domain provider:
	- For apex domain (`nextstepailabs.com`): add `A` records to these GitHub Pages IPs:
	  - `185.199.108.153`
	  - `185.199.109.153`
	  - `185.199.110.153`
	  - `185.199.111.153`
	- Optional: for `www.nextstepailabs.com`, add a `CNAME` record to `<your-github-username>.github.io`.
4. Enable **Enforce HTTPS** in GitHub Pages once SSL is ready.

## Notes

- Replace placeholder App Store and Google Play links in `index.html`.
- Update legal links (Privacy, Terms, Support) before production.
