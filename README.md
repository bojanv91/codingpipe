# codingpipe

## Prerequisites

- Node v20 (LTS)

For example, using NVM:

```bash
nvm use 20.16.0
```

## Getting Started

[Link to eleventy-base-blog README](https://github.com/11ty/eleventy-base-blog#readme)

### 1. Make a directory and navigate to it:

```bash
mkdir codingpipe
cd codingpipe
```

### 2. Clone this repository

```bash
git clone https://github.com/bojanv91/codingpipe.git .
```

Review `eleventy.config.js` and `_data/metadata.js` to configure the site’s options and data.

### 3. Install dependencies

```bash
npm install
```

### 4. Run Eleventy

Build and host on a local development server (hot-reload is automatically turned on):

```bash
npm run start
```

The project will start at <http://localhost:8080/>.

Generate a production-ready build to the `_site` folder:

```bash
npm run build
```
