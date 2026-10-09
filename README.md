# Pipeline Training

This project is a **training exercise** to learn how to use CI/CD pipelines with **GitHub Actions**.

The application itself is very simple: an Angular "Hello World" page. The goal is not the app, but the **pipelines** around it:

- check the code automatically when a pull request is opened
- run the tests
- build the app
- deploy it automatically to **GitHub Pages** when a pull request is merged

## Workflows

The workflows are in the `.github/workflows/` folder.

### 1. Lint (`lint.yml`)

**When does it run?**
- when a pull request targets the `main` branch (and on every new push to that pull request)
- manually, from the **Actions** tab (`workflow_dispatch`)

**What does it do?**
1. Gets the code of the repository (`actions/checkout`)
2. Installs Node.js 22
3. Installs the dependencies with `npm ci`
4. Runs ESLint with `npm run lint`

If ESLint finds a problem, the workflow fails and the pull request shows a red check.

### 2. Build and deploy (`deploy.yml`)

**When does it run?**
- when code is pushed to `main` (merging a pull request is a push to `main`)
- manually, from the **Actions** tab (`workflow_dispatch`)

This workflow has **two jobs**. They run on two different virtual machines, one after the other.

**Job `build`**
1. Gets the code of the repository
2. Installs Node.js 22
3. Installs the dependencies with `npm ci`
4. Runs the unit tests in a headless Chrome browser. If a test fails, the workflow stops and nothing is deployed.
5. Builds the Angular app with the right base path for GitHub Pages
6. Uploads the built site (`dist/hello-world/browser`) as an **artifact**

**Job `deploy`**
- Starts only if the `build` job succeeded (`needs: build`)
- Takes the artifact from the `build` job and publishes it on GitHub Pages

The artifact is how the two jobs share files: the `deploy` machine cannot see the disk of the `build` machine.

## Git workflow

The `main` branch is the source of truth. Nobody works directly on it.

1. Create a new branch from `main`
2. Commit and push your changes
3. Open a pull request to `main` → the **Lint** workflow runs
4. Merge the pull request → the **Build and deploy** workflow runs and the website is updated

## Run the project locally

You need Node.js 20.19+ or 22.12+.

```bash
npm install
npm start        # starts the app on http://localhost:4200
npm run lint     # runs ESLint
npm test         # runs the unit tests
npm run build    # builds the app into the dist/ folder
```

## Tools used

- [Angular](https://angular.dev) – web framework
- [ESLint](https://eslint.org) – code linter
- [Jasmine](https://jasmine.github.io) and [Karma](https://karma-runner.github.io) – unit tests
- [GitHub Actions](https://docs.github.com/en/actions) – CI/CD pipelines
- [GitHub Pages](https://pages.github.com) – hosting for the website
