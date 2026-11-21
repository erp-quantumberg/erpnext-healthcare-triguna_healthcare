# Triguna Healthcare

Triguna Healthcare is a Frappe app that serves the Bloodtestnearme public website. Its user interface is a Vue 3 single-page application built with Vite and mounted in Frappe. The frontend retrieves package, booking, blog, job, and diagnostic-center data through Frappe API methods.

This repository contains the website app and frontend; the API implementation is provided by a separate backend app. See [Required apps](#required-apps) before installing.

## Features

- Browse blood tests, health checkups, categories, offers, and featured packages.
- View package details and included tests, and add packages to a cart. Cart contents are persisted in the browser's local storage.
- Submit an order with the selected packages and collection details through the backend API.
- Find diagnostic centers, view center details, and check supported pincodes.
- Read health and wellness articles, FAQs, and company information, including contact, privacy, and terms pages.
- View job openings and submit job applications.
- Support QR-code entry, city-specific landing pages, responsive navigation, and search.
- Provide page titles, descriptions, canonical URLs, and sitemap generation for the public site.

## Requirements

- A working Frappe Bench and a Frappe site. Frappe is managed by Bench and is not installed with `npm`.
- Python 3.10 or later for this app's Python package metadata.
- Node.js and npm for installing and building the Vue frontend.
- The backend app described below, installed on the same site, for the API methods used by the website.

### Required apps

| App | Requirement | Purpose |
| --- | --- | --- |
| `frappe` | Required | Provides the site, routing, and API infrastructure. |
| Backend app exposing the `bloodtestnearme.api.*` namespace | Required for API-backed website features | Implements the package, order, blog, job, and diagnostic-center methods called by the frontend. The Python namespace does not necessarily match the app's Bench install name; use the name documented by the backend app. |
| `erpnext` | Not required by this app | This app does not use ERPNext-specific functionality. Install ERPNext only if your site needs it for other apps or workflows. |

The backend app is not included in this repository. Install it from its own repository and confirm that it is installed on the same site. These are runtime requirements; this app does not currently enforce them through Frappe's `required_apps` hook.

### Frontend npm packages

The frontend's direct runtime dependencies are declared in [`frontend/package.json`](frontend/package.json) and pinned in [`frontend/package-lock.json`](frontend/package-lock.json):

- Vue 3 and Vue Router
- Frappe UI
- Pinia
- Axios
- Element Plus
- Bootstrap and Bootstrap Icons
- Feather Icons
- Vue G-Tag
- Vue 3 Carousel

Build tooling includes Vite, the Vue Vite plugin, Tailwind CSS, PostCSS, and Autoprefixer. Use npm from the `frontend/` directory; there is no root-level frontend package manifest.

## Installation

From your Bench directory, install the backend app from its project repository if it is not already available:

```bash
cd /path/to/your-bench
bench get-app <BACKEND_APP_REPOSITORY_URL>
bench get-app https://github.com/Nitish-Mashal/Triguna_Healthcare.git
```

Install both apps on the target site, backend first:

```bash
bench --site <site-name> install-app <BACKEND_APP_NAME>
bench --site <site-name> install-app triguna_healthcare
bench --site <site-name> migrate
```

Replace the backend placeholders with the repository URL and Frappe app name used by your backend project. If the backend app is already installed on the site, do not install it a second time.

Install frontend dependencies and build the app:

```bash
cd apps/triguna_healthcare/frontend
npm ci
npm run build
```

After building, return to the Bench directory and build/refresh Frappe assets as required by your deployment:

```bash
cd ../../..
bench build --app triguna_healthcare
bench --site <site-name> clear-cache
```

Restart the Bench processes after deploying the updated assets.

## Setup

### Local development

Start Bench from the Bench directory:

```bash
bench start
```

In another terminal, start the Vite development server:

```bash
cd /path/to/your-bench/apps/triguna_healthcare/frontend
npm ci
npm run dev
```

The Vite server listens on port `8080`. Open your Frappe site's hostname on that port (for example, `http://mysite.localhost:8080`). The Vite configuration reads `sites/common_site_config.json` to proxy requests to the Bench web server. Ensure the site hostname resolves locally and that the backend app is installed and available.

### Production build

From `apps/triguna_healthcare/frontend`, run:

```bash
npm ci
npm run build
```

The Vite build output is configured under the app's `triguna_healthcare/public/frontend` directory. Run the appropriate Bench asset build and restart/reload your deployment so the latest frontend assets are served.

## Troubleshooting

- **`npm` cannot find `package.json` or the lockfile:** Run npm commands from `apps/triguna_healthcare/frontend`, not from the app or Bench root.
- **Vite cannot reach Frappe or API requests fail in development:** Start `bench start`, check that `sites/common_site_config.json` contains the correct `webserver_port`, and open the site using its configured hostname on port `8080`.
- **API requests return 404 or “method not found”:** Verify the backend app is installed on the same site and provides the methods under `bloodtestnearme.api.*` referenced by the frontend.
- **API requests return permission or validation errors:** Check the backend method's permissions, required request fields, and site configuration. The frontend does not replace backend validation.
- **The production website shows old assets:** Re-run `npm run build`, refresh the Bench assets, clear the site cache, and restart/reload the relevant web processes.
- **A direct page URL fails on refresh:** Confirm the requested frontend route is covered by the website route rules in `triguna_healthcare/hooks.py`, then clear cache and restart after changing route rules.

## Configuration notes

- Frappe website route rules in `triguna_healthcare/hooks.py` preserve system routes such as `/api`, `/app`, and `/assets`, and direct frontend routes to the Vue application.
- Frontend API requests use same-origin Frappe method URLs, primarily under `/api/method/bloodtestnearme.api.*`. The backend app must expose and authorize those methods.
- Vite's development server uses port `8080` and proxies to the Bench webserver port read from `sites/common_site_config.json`.
- The canonical website origin is currently set to `https://www.trigunahealthcare.com` in `frontend/src/App.vue`. Update it when deploying under a different domain.
- The Google tag ID is configured in `frontend/src/main.js`. Change it to the appropriate property for your deployment.
- Route metadata and the client-side route list are maintained in `frontend/src/router.js`. The sitemap generator is in `frontend/scripts/generate-sitemap.mjs`.

## Contributing

1. Make changes on a feature branch and keep frontend changes in `frontend/`.
2. Install the repository's pre-commit hooks from the app root:

   ```bash
   cd apps/triguna_healthcare
   pre-commit install
   ```

3. For frontend changes, run the production build from `frontend/`:

   ```bash
   cd frontend
   npm ci
   npm run build
   ```

4. For app-level formatting and lint checks, run the configured pre-commit checks before submitting a pull request.

Please include a description of the behavior changed and any setup or migration steps needed by the backend app.

## License

This project is distributed under the MIT License. See [`license.txt`](license.txt) for the license text.
