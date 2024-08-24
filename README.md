# [Angular Micro Frontend (MFE) Project Creation](https://blog.bitsrc.io/an-easy-to-follow-guide-to-create-angular-micro-frontends-9e6b7c0fff78)

Creating an Angular Micro-Frontend (MFE) project involves several steps and configurations. Here is a detailed guide on how to set up an Angular MFE project using the Angular CLI and Angular Architects' module federation.

### Step-by-Step Guide

#### 1. Create a New Angular Workspace

Create a new Angular workspace without creating an initial application:

```sh
ng new [project-name] --create-application=false
cd [project-name]
```

#### 2. Generate Applications

Create the applications within the workspace(project directory). For instance, you can create three applications:

```sh
ng g application shell --routing --style=scss
ng g application [application-name1] --routing --style=scss
ng g application [application-name2] --routing --style=scss
```

#### 3. Add Module Federation

Add the Angular Architects' Module Federation plugin to each application. Specify the type as `remote` and the port number for each application:

```sh
ng add @angular-architects/module-federation --project shell --type remote --port [port-number1]
ng add @angular-architects/module-federation --project [application-name1] --type remote --port [port-number1]
ng add @angular-architects/module-federation --project [application-name2] --type remote --port [port-number2]
```

#### 4. Generate Modules and Components

Navigate to the application directory and generate a new module with routing:

```sh
cd projects/[application-name]/src/app
ng g m [module-name] --route [module-name] --module app
```

Generate a new component within the created module:

```sh
ng generate c [component-name] --module=[module-name] --project [application-name]
```

#### 5. Modify the Shell Project

Update the shell project to dynamically load the remote applications.

##### Directory Structure

Ensure your directory structure looks like this:

```
projects/
  shell/
    src/
      app/
        app-routing.module.ts
      assets/mfe-manifest/
        manifest.dev.json
        manifest.json
    webpack.config.js
  angular.json
  package.json
```

##### `app-routing.module.ts`

Configure the routing to load remote applications:

```typescript
import { NgModule } from "@angular/core";
import { RouterModule, Routes } from "@angular/router";

const routes: Routes = [
	{
		path: "remote-app1",
		loadChildren: () =>
			import("remoteApp1/Module").then((m) => m.RemoteApp1Module),
	},
	{
		path: "remote-app2",
		loadChildren: () =>
			import("remoteApp2/Module").then((m) => m.RemoteApp2Module),
	},
	{ path: "", redirectTo: "remote-app1", pathMatch: "full" },
];

@NgModule({
	imports: [RouterModule.forRoot(routes)],
	exports: [RouterModule],
})
export class AppRoutingModule {}
```

##### `manifest.dev.json`

Define the remote applications in the manifest:

```json
{
	"remoteApp1": "http://localhost:[port-number1]/remoteEntry.js",
	"remoteApp2": "http://localhost:[port-number2]/remoteEntry.js"
}
```

##### `manifest.json`

Define the remote applications in the manifest:

```json
{
	"remoteApp1": "http://localhost:[port-number1]/remoteEntry.js",
	"remoteApp2": "http://localhost:[port-number2]/remoteEntry.js"
}
```

##### `webpack.config.js`

Configure module federation in `webpack.config.js`:

```javascript
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
const mf = require("@angular-architects/module-federation/webpack");
const path = require("path");
const share = mf.share;

module.exports = {
	output: {
		uniqueName: "shell",
		publicPath: "auto",
	},
	optimization: {
		runtimeChunk: false,
	},
	resolve: {
		alias: {
			...mf.sharedMappings.getAliases(),
		},
	},
	plugins: [
		new ModuleFederationPlugin({
			library: { type: "module" },
			name: "shell",
			remotes: {
				remoteApp1: "remoteApp1@http://localhost:[port-number1]/remoteEntry.js",
				remoteApp2: "remoteApp2@http://localhost:[port-number2]/remoteEntry.js",
			},
			shared: share({
				"@angular/core": {
					singleton: true,
					strictVersion: true,
					requiredVersion: "auto",
				},
				"@angular/common": {
					singleton: true,
					strictVersion: true,
					requiredVersion: "auto",
				},
				"@angular/router": {
					singleton: true,
					strictVersion: true,
					requiredVersion: "auto",
				},
				...mf.sharedMappings.getDescriptors(),
			}),
		}),
		mf.SharedMappings,
	],
};
```

##### `angular.json`

Ensure `angular.json` includes the appropriate build and serve configurations for each application:

```json
{
	"projects": {
		"shell": {
			// existing configuration
		},
		"[application-name1]": {
			// configuration for application-name1
		},
		"[application-name2]": {
			// configuration for application-name2
		}
	}
}
```

##### `package.json`

Ensure you have the necessary dependencies in `package.json`:

```json
{
  "name": "[project-name]",
  "version": "0.0.0",
  "scripts": {
    "ng": "ng",

    "shell": "ng serve shell --configuration development --port 4200 -o",
    "mfe-app1": "ng serve [application-name1] --configuration development --port 4201",
    "mfe-app2": "ng serve [application-name2] --configuration development --port 4202",

    "start": "concurrently \"npm run shell\" \"npm run mfe-app1\" \"npm run mfe-app2\" ",

    "lib:shared:build": "ng build @auc-lib/shared",
    "build:shell:dev": "ng build shell --configuration production",
    "build:app1:dev": "ng build [application-name1] --configuration production",
    "build:app2:dev": "ng build [application-name2] --configuration production",


    "build": "npm run lib:shared:build && npm run build:shell:dev && npm run build:app1:dev && npm run build:app2:dev",


         ------ Bellow Not Necessary to Chang -------

    "test:shell": "ng test shell",
    "test:[application-name1]": "ng test [application-name1]",
    "test:[application-name2]": "ng test [application-name2]",


    "test": "concurrently \"npm run test:shell\" \"npm run test:app1\" \"npm run test:app2\"",

    "lint:shell": "ng lint shell",
    "lint:[application-name1]": "ng lint [application-name1]",
    "lint:[application-name2]": "ng lint [application-name2]",


    "lint": "concurrently \"npm run lint:shell\" \"npm run lint:app1\" \"npm run lint:app2\" ",

    "e2e:shell": "ng e2e shell",
    "e2e:[application-name1]": "ng e2e [application-name1]",
    "e2e:[application-name2]": "ng e2e [application-name2]",


    "e2e": "concurrently \"npm run e2e:shell\" \"npm run e2e:app1\" \"npm run e2e:app2\""
  },
  "private": true,
  "dependencies": {
    "@angular/animations": "~14.0.0",
    "@angular/common": "~14.0.0",
    "@angular/compiler": "~14.0.0",
    "@angular/core": "~14.0.0",
    "@angular/forms": "~14.0.0",
    "@angular/platform-browser": "~14.0.0",
    "@angular/platform-browser-dynamic": "~14.0.0",
    "@angular/router": "~14.0.0",
    "rxjs": "~7.5.0",
    "tslib": "^2.3.0",
    "zone.js": "~0.11.4",
    "@angular-architects/module-federation": "^14.3.3"
  },
  "devDependencies": {
    "@angular-devkit/build-angular": "~14.0.0",
    "@angular/cli": "~14.0.0",
    "@angular/compiler-cli": "~14.0.0",
    "@types/jasmine": "~4.0.0",
    "@types/node": "^12.11.1",
    "concurrently": "^7.3.0",
    "jasmine-core": "~4.0.0",
    "karma": "~6.3.0",
    "karma-chrome-launcher": "~3.1.0",
    "karma-coverage": "~2.2.0",
    "karma-jasmine": "~4.0.0",
    "karma-jasmine-html-reporter": "^1.7.0",
    "typescript": "~4.6.2"
  }
}

```
#### 6. Modify the Created Application's Webpack Configuration

In each created application, you need to modify the webpack configuration `webpack.config.js` to include module federation and shared mappings:

```js
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
const mf = require("@angular-architects/module-federation/webpack");
const path = require("path");
const share = mf.share;

const sharedMappings = new mf.SharedMappings();
sharedMappings.register(path.join(__dirname, "../../tsconfig.json"), [
	/* mapped paths to share */
]);

module.exports = {
	output: {
		uniqueName: "[application-name]",
		publicPath: "auto",
	},
	optimization: {
		runtimeChunk: false,
	},
	resolve: {
		alias: {
			...sharedMappings.getAliases(),
		},
	},
	experiments: {
		outputModule: true,
	},
	plugins: [
		new ModuleFederationPlugin({
			library: { type: "module" },

			// For remotes (please adjust)
			name: "[application-name]",
			filename: "remoteEntry.js",
			exposes: {
				"./[module-name]":
					"./projects/[application-name]/src/app/[module-name]/[module-name].module.ts",
			},

			shared: share({
				"@angular/core": {
					singleton: true,
					strictVersion: true,
					requiredVersion: "auto",
				},
				"@angular/common": {
					singleton: true,
					strictVersion: true,
					requiredVersion: "auto",
				},
				"@angular/common/http": {
					singleton: true,
					strictVersion: true,
					requiredVersion: "auto",
				},
				"@angular/router": {
					singleton: true,
					strictVersion: true,
					requiredVersion: "auto",
				},

				...sharedMappings.getDescriptors(),
			}),
		}),
		sharedMappings.getPlugin(),
	],
};
```

### Summary

This guide outlines the process for creating an Angular Micro-Frontend (MFE) project. It includes steps to create a workspace and applications, add module federation, generate modules and components, and update configuration files for the shell project. This setup allows dynamic loading of remote applications using module federation.
