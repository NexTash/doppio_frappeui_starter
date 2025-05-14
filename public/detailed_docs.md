Integrating a Vue.js App with a Frappe App Using Doppio FrappeUI Starter
This guide provides a step-by-step process to integrate a Vue.js application with a Frappe app using the doppio_frappeui_starter template. It covers setting up the development environment, configuring API proxies, preparing for production, and testing the integration. The guide explains all non-bench commands in detail, assuming familiarity with Frappe bench commands.
Prerequisites

A Frappe bench environment set up with a site.
Node.js and Yarn installed on your system.
Basic knowledge of Vue.js, Vite, and Frappe framework.
Access to the doppio_frappeui_starter template: GitHub Develop Branch.

Step-by-Step Integration Process
1. Clone the Vue Frontend Template
Navigate to the apps/todo directory within your Frappe bench (replace todo with your app name if different).
cd apps/todo

Use the npx degit command to clone the doppio_frappeui_starter template into a frontend directory:
npx degit NagariaHussain/doppio_frappeui_starter frontend

Command Explanation:

npx: Runs a package (degit) without installing it globally.
degit: A lightweight tool for cloning Git repositories without the full Git history, ideal for templates.
NagariaHussain/doppio_frappeui_starter: The GitHub repository containing the Vue.js template.
frontend: The destination directory where the template is cloned.

This creates a frontend directory with the Vue.js app structure, including Vite configuration and Frappe UI components.
2. Configure Vite Proxy for API Calls
Edit the frontend/vite.config.js file to add a proxy configuration, enabling the Vue app to communicate with the Frappe backend during development.
Add the following to the server section of vite.config.js:
server: {
  proxy: {
    '/api': {
      target: 'http://{BASE_LINK}:{SITE_PORT}', // e.g., http://127.0.0.1:8015
      changeOrigin: true,
      secure: false
    }
  }
}

Configuration Explanation:

server.proxy: Configures Vite’s development server to forward requests matching a pattern.
'/api': Matches all requests starting with /api (Frappe API endpoints).
target: The Frappe backend URL (e.g., http://127.0.0.1:8015). Replace {BASE_LINK} and {SITE_PORT} with your site’s host and port.
changeOrigin: true: Modifies the request’s origin header to match the target, avoiding CORS issues.
secure: false: Allows connections to non-HTTPS backends (common in local development).

This ensures that API calls from the Vue app are proxied to the Frappe backend, bypassing CORS restrictions.
3. Configure the Frappe Site for API Access
In your Frappe site’s site_config.json file (located in the site directory, e.g., sites/your_site/site_config.json), add the following setting:
{
  "ignore_csrf": 1
}

Setting Explanation:

"ignore_csrf": 1: Disables CSRF (Cross-Site Request Forgery) checks for API requests. This is necessary for the Vue app to make API calls to the Frappe backend without CSRF token validation, simplifying development. Note: Use with caution in production, as disabling CSRF can pose security risks. Consider enabling CSRF protection with proper token handling in production.

4. Set Up the Vue Frontend
Navigate to the frontend directory:
cd frontend

Install dependencies using Yarn:
yarn

Command Explanation:

yarn: Installs all dependencies listed in frontend/package.json. Yarn is a package manager that ensures consistent dependency installation for the Vue app, including Vite, Vue.js, and Frappe UI libraries.

Start the Vite development server:
yarn dev

Command Explanation:

yarn dev: Runs the dev script defined in frontend/package.json, which starts Vite’s development server. Vite provides hot module replacement (HMR) for fast development, typically running on http://localhost:5173 (check the terminal output for the exact port).

Access the Vite URL (e.g., http://localhost:5173) in your browser. The Vue app should load, and API calls to the Frappe backend should work, thanks to the proxy configuration.
5. Set Up Production Configuration
To prepare the app for production, configure the Frappe app to serve the Vue app’s built assets and integrate it with the Frappe website.
Create a Root package.json
In the app’s root directory (e.g., apps/todo), create a package.json file with the following content:
{
    "private": true,
    "workspaces": [
        "frontend",
        "frappe-ui"
    ],
    "scripts": {
        "postinstall": "cd frontend && yarn install",
        "dev": "cd frontend && yarn dev",
        "build": "cd frontend && yarn build"
    }
}

File Explanation:

"private": true: Marks the package as private, preventing accidental publication.
"workspaces": Declares subdirectories (frontend and frappe-ui) as Yarn workspaces, enabling shared dependency management.
"scripts":
postinstall: Runs yarn install in the frontend directory after the root package.json dependencies are installed.
dev: Runs the Vite development server in the frontend directory.
build: Triggers the production build in the frontend directory.



Update Frontend package.json
Modify the frontend/package.json file to include build configurations that integrate with Frappe’s asset structure. Add or update the following scripts:
{
  "scripts": {
    "build": "vite build --base=/assets/pox/frontend/ && yarn copy-html-entry",
    "copy-html-entry": "cp ../pox/public/frontend/index.html ../pox/www/pox.html"
  }
}

Configuration Explanation:

"build": Runs two commands in sequence:
vite build --base=/assets/pox/frontend/: Builds the Vue app with Vite, setting the base path for assets to /assets/pox/frontend/. This ensures that the built assets are correctly referenced when served by Frappe.
yarn copy-html-entry: Runs the copy-html-entry script.


"copy-html-entry": Copies the index.html file from pox/public/frontend to pox/www/pox.html. This makes the Vue app’s entry point accessible as a Frappe web page (pox.html).

Command Explanation:

vite build: Compiles the Vue app into static assets (HTML, CSS, JS) optimized for production.
--base=/assets/pox/frontend/: Sets the base URL for asset paths, aligning with Frappe’s asset directory structure.
cp: Copies a file from one location to another (cp source destination).
../pox/public/frontend/index.html: The source file (Vue’s built index.html).
../pox/www/pox.html: The destination, where Frappe serves web pages.

Configure .gitignore
In the app’s root directory, create or update the .gitignore file to exclude production build artifacts and unnecessary files from version control:
# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
pip-wheel-metadata/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg

# Website build
pox/public/frontend
pox/public/node_modules
pox/www/pox.html
pox/components.d.ts

File Explanation:

Ignores Python-related build artifacts (e.g., .Python, build/, *.egg-info/), which are generated during Frappe app installation.
Ignores Node.js modules (pox/public/node_modules) to avoid committing dependencies.
Ignores Vue build outputs (pox/public/frontend, pox/www/pox.html) and TypeScript declaration files (pox/components.d.ts) to keep the Git repository clean.

Configure Website Routes in hooks.py
In your app’s hooks.py file (e.g., apps/pox/pox/hooks.py), add the following route configuration:
website_route_rules = [
    {"from_route": "/pox/<path:app_path>", "to_route": "pox"},
]

Configuration Explanation:

website_route_rules: Defines URL routing rules for the Frappe website.
{"from_route": "/pox/<path:app_path>", "to_route": "pox"}: Redirects all URLs starting with /pox/ (e.g., /pox/some/path) to the pox route, which serves pox.html. This ensures that the Vue app handles client-side routing for all subpaths.

6. Build and Test the Application
Build the Vue app for production:
cd apps/todo
yarn build

Command Explanation:

yarn build: Runs the build script in the root package.json, which executes cd frontend && yarn build. This triggers Vite to build the Vue app and copy the index.html file to pox/www/pox.html.

Test the app in both environments:

Development: Access the Vite server URL (e.g., http://localhost:5173). Verify that the Vue app loads and API calls work.
Production: Access the Frappe site’s URL (e.g., http://127.0.0.1:8015/pox). Verify that the built Vue app loads and API calls function correctly.

7. Troubleshooting

API Calls Fail: Ensure the vite.config.js proxy target matches your Frappe site’s URL and port, and "ignore_csrf": 1 is set in site_config.json.
Assets Not Loading in Production: Verify the --base path in the build script matches the Frappe asset directory (/assets/pox/frontend/).
Page Not Found: Confirm the website_route_rules in hooks.py are correctly set and pox.html exists in pox/www/.

Conclusion
You have successfully integrated a Vue.js app with a Frappe app using the doppio_frappeui_starter template. The development environment uses Vite for fast iteration, while the production setup serves the Vue app through Frappe’s web server. Both environments support API calls to the Frappe backend, enabling a seamless full-stack application.
For further customization, refer to the doppio_frappeui_starter documentation or explore Frappe UI components for enhanced integration.
