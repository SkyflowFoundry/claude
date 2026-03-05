# Skill: Quickstart Skyflow JS Browser (Vite)

This skill sets up a simple front-end using Vite to demonstrate Skyflow Elements from `skyflow-js` alongside the existing Node backend in this repo.

## Goals

- Add a Vite-powered front-end that mounts Skyflow Elements.
- Keep the existing Node backend intact.
- Allow production builds to be served by the Node server.

## Prerequisites

- Node.js installed
- Repo root: `skyflow-node-sample`

## Steps

### 1) Install dependencies

```sh
npm install
```

If `skyflow-js` is not already installed:

```sh
npm install skyflow-js
```

Install Vite as a dev dependency:

```sh
npm install -D vite@^7.3.1
```

### 2) Add Vite config

Create `vite.config.ts` at the repo root:

```ts
import { defineConfig } from "vite";
import path from "path";

export default defineConfig({
	root: process.cwd(),
	publicDir: "public",
	build: {
		outDir: path.resolve(process.cwd(), "dist", "client"),
		emptyOutDir: true,
	},
	server: {
		port: 5173,
	},
});
```

### 3) Add the front-end page

Create `index.html` at the repo root (Vite entry). Ensure the mount targets have a defined height to avoid the default 150px iframe height:

```html
<!doctype html>
<html lang="en">
	<head>
		<meta charset="UTF-8" />
		<meta name="viewport" content="width=device-width, initial-scale=1.0" />
		<title>Skyflow Elements Demo</title>
		<style>
			#card-number,
			#cardholder-name,
			#expiry-month,
			#expiry-year {
				height: 34px;
			}
		</style>
	</head>
	<body>
		<div>
			<h1>Skyflow Elements Demo</h1>
			<div id="card-number"></div>
			<div id="cardholder-name"></div>
			<div id="expiry-month"></div>
			<div id="expiry-year"></div>
			<button type="button" id="submit">Create Tokens</button>
		</div>

		<script type="module" src="/src/client.ts"></script>
	</body>
</html>
```

### 4) Add the client entry

Create `src/client.ts`:

```ts
/// <reference types="vite/client" />

import Skyflow from "skyflow-js";

const submitButton = document.querySelector<HTMLButtonElement>("#submit");
if (!submitButton) {
	throw new Error("Missing submit button");
}

const vaultID = import.meta.env.VITE_SKYFLOW_VAULT_ID as string | undefined;
const vaultURL = import.meta.env.VITE_SKYFLOW_VAULT_URL as string | undefined;
const bearerToken = import.meta.env.VITE_SKYFLOW_BEARER_TOKEN as
	| string
	| undefined;

if (!vaultID) {
	throw new Error("Missing VITE_SKYFLOW_VAULT_ID");
}
if (!vaultURL) {
	throw new Error("Missing VITE_SKYFLOW_VAULT_URL");
}
if (!vaultURL.startsWith("https://")) {
	throw new Error("VITE_SKYFLOW_VAULT_URL must begin with 'https://'");
}
if (!bearerToken) {
	throw new Error("Missing VITE_SKYFLOW_BEARER_TOKEN");
}

const skyflow = Skyflow.init({
	vaultID,
	vaultURL,
	getBearerToken: async () => {
		return bearerToken;
	},
});

const elements = skyflow.container(Skyflow.ContainerType.COLLECT);

const hiddenLabelStyles = {
	base: {
		position: "absolute",
		width: "1px",
		height: "1px",
		padding: "0",
		margin: "-1px",
		overflow: "hidden",
		clip: "rect(0, 0, 0, 0)",
		border: "0",
	},
};

const cardNumber = elements.create({
	table: "credit_cards",
	column: "card_number",
	type: Skyflow.ElementType.CARD_NUMBER,
	label: "Card number",
	labelStyles: hiddenLabelStyles,
	placeholder: "4111 1111 1111 1111",
});
const cardholderName = elements.create({
	table: "credit_cards",
	column: "cardholder_name",
	type: Skyflow.ElementType.CARDHOLDER_NAME,
	label: "Cardholder name",
	labelStyles: hiddenLabelStyles,
	placeholder: "Jane Doe",
});
const expiryMonth = elements.create({
	table: "credit_cards",
	column: "expiry_month",
	type: Skyflow.ElementType.EXPIRATION_MONTH,
	label: "Expiry month",
	labelStyles: hiddenLabelStyles,
	placeholder: "MM",
});
const expiryYear = elements.create({
	table: "credit_cards",
	column: "expiry_year",
	type: Skyflow.ElementType.EXPIRATION_YEAR,
	label: "Expiry year",
	labelStyles: hiddenLabelStyles,
	placeholder: "YY",
});

cardNumber.mount("#card-number");
cardholderName.mount("#cardholder-name");
expiryMonth.mount("#expiry-month");
expiryYear.mount("#expiry-year");

submitButton.addEventListener("click", async () => {
	const response = await elements.collect({
		tokens: true,
	});
	console.log("Skyflow collect response:", response);
});
```

### 5) Update package scripts

Add scripts to `package.json`:

```json
{
	"scripts": {
		"build": "npm run build:client && tsc",
		"build:client": "vite build",
		"dev": "tsc --watch",
		"dev:client": "vite",
		"preview:client": "vite preview"
	}
}
```

### 6) Update the Node server static hosting

Ensure the backend serves `dist/client` when present, falling back to `public`.

```ts
const distClientDir = path.resolve(process.cwd(), "dist", "client");
const publicDir = path.resolve(process.cwd(), "public");
const staticDir = existsSync(distClientDir) ? distClientDir : publicDir;
```

### 7) Run in development

Front-end dev server:

```sh
npm run dev:client
```

Backend watch mode:

```sh
npm run dev
```

### 8) Build for production

```sh
npm run build:client
npm run build
npm start
```

The backend will serve the built front-end from `dist/client`.

## Notes

- `skyflow-js` is browser-compatible, but it is distributed as an npm module, which is why Vite (or another bundler) is required.
- Replace placeholder values for `vaultID`, `vaultURL`, and `getBearerToken` with your real values.
- If you run `npm audit fix --force`, ensure `vite` remains on a patched release (currently `^7.3.1`).
