# Angular 14 - Module Federation

## About

The purpose of this project is to document the installation, configuration and integration process of the following concepts.

### 1. [Multiple Apps in Workspace](#1-multiple-angular-apps-in-workspace)

### 2. [ESLint and Prettier and Husky](#2-eslint-and-prettier-and-husky-hooks)

### 3. [Micro Frontends - Module Federation](#3-micro-frontends-module-federation)

**Note:** Normally i would have each project in there own Git-repo. To compress this example into one project here on github, I have baked everything together in a monorepo, which in some project forms has its advantages. This have caused some manuall steps that are not required when having one project per Git-repo.

**Note:** IMHO - many build systems require much ocnfigration and a careful hand to maintain them. From my experience it's easy to tip over the edge where getting too technical only causes problems between different version of Angular, plugins and third-partuy libraries. Keep it simple. In this setup we will use `ESLint`, `Prettier` and `Husky` to run pre-commit and pre-push hooks to lint, format and test the code.

## 1. Multiple Angular Apps in Workspace

Chapter describing the process of setting up multiple projects in the same workspace using the same Git-repo.

### 1.1. Created Empty Workspace

_I had cloned an empty repo from GitHub, thats why i used the directory switch set to the current directory._

```
$ ng new ModuleFederation --create-application="false" --directory ./
```

### 1.2. Add Shell Application

```
$ ng generate application shell-app --style=scss --routing=true
```

### 1.3. Add Micro Frontend Applications

Add one or many applications that are used as Micro Frontends.

First Micro Frontend.

```
$ ng generate application mf-header --style=scss --routing=false
```

Second Micro Frontend.

```
$ ng generate application mf-movies --style=scss --routing=true
```

### 1.4. Serving applications

Two ways of serving the applications.

```
$ ng serve <project-name>
$ ng server --project="<project-name>"
```

### 1.5. Building applications

To build applications for production use the following commands.

```
$ ng build --prod --project="shell-app"
$ ng build --prod --project="mf-header"
$ ng build --prod --project="mf-movies"
```

## 2. ESLint and Prettier and Husky-hooks

This chapter is all about making the code more maintainable, readable, consistent and professional.

### 2.1 Install ESLint

**Why:** Linting is performed to reduce errors and improve the overall quality of the application.

```
$ ng add @angular-eslint/schematics
```

The ESLint configuration is now done thanks to the `ng add` command that sets up all necessary config files (_Not true because of monorepo, keep on reading_). The following command can be run to check for linting errors.

```
$ ng lint
```

If the linter finds errors, the following command can be run to try and fix them automatically.

```
$ ng lint --fix
```

**Note:** There are two additional configurations requred because of this beeing a monorepo with mutliple projects.

**Step 1:** Add a configuration file. This file is normally added automatically when using the `$ ng add @angular-eslint/schematics` command in a single project.

```
$ touch .eslintrc.json
```

Add the following config to the `.eslintrc.json` file.

```jsonc
{
    "root": true,
    "overrides": [
        {
            "files": ["*.ts"],
            "parserOptions": {
                "project": ["tsconfig.json"],
                "createDefaultProgram": true
            },
            "extends": [
                "plugin:@angular-eslint/recommended",
                "plugin:@angular-eslint/template/process-inline-templates"
            ],
            "rules": {
                "@angular-eslint/directive-selector": [
                    "error",
                    {
                        "type": "attribute",
                        "prefix": "app",
                        "style": "camelCase"
                    }
                ],
                "@angular-eslint/component-selector": [
                    "error",
                    {
                        "type": "element",
                        "prefix": "app",
                        "style": "kebab-case"
                    }
                ]
            }
        },
        {
            "files": ["*.html"],
            "extends": ["plugin:@angular-eslint/template/recommended"],
            "rules": {}
        }
    ]
}
```

**Step 2:** The second step is to configure linting for all the projects in the repository.

For all projects in the `angular.json` file append a `lint section` with that project name.

```jsonc
    // ...
    // other project-config
    // ...
    "test": {
        //......
    },
    "lint": {
        "builder": "@angular-eslint/builder:lint",
        "options": {
        "lintFilePatterns": [
            "projects/shell-app/src/**/*.ts",
            "projects/shell-app/src/**/*.html"
        ]
    }
}
```

Thats it! Linting is now installed and configured.

### 2.2 Install Prettier

**Why:** Prettier is used to format code. This make the codebase more readable and maintainable.

```
$ npm install prettier --save-dev
```

After the installation add a configuration file.

```
$ touch .prettierrc.json
```

Add the following config to the `.prettierrc.json` file.

```jsonc
{
    "tabWidth": 4,
    "useTabs": false,
    "singleQuote": true,
    "semi": true,
    "bracketSpacing": true,
    "arrowParens": "avoid",
    "trailingComma": "es5",
    "bracketSameLine": true,
    "printWidth": 80
}
```

Change the following property in the `.editorconfig` file. This is just a precaution.

```
indent_size = 4
```

Create `.prettierignore` that is based on the `.gitignore` file.

```
$ cp .gitignore .prettierignore
```

The Prettier-configuration is now done. The following command can be run to format all files in the projects.

```
$ npx prettier --write .
```

**Note:** Two npm aliases for Prettier will be set up in the next Husky section.

### 2.3 Install Husky

**Why:** Husky makes working with Git Hooks easier. The pre-commit-hook will make shure that both formatting and linting have been done before a commit is allowed to be made. The pre-push-hook will run tests before the code can be pushed to remote.

**Note:** The hooks are meant to be helpful in preventing mistakes or forgotten code. You should still be able to push unfinished code to remote when working on a feature branch with others.

```
$ npm install husky --save-dev
```

Add new scripts to the script section in the `package.json` file.

```jsonc
{
    // ...
    // other package.json config
    // ...
    "scripts": {
        // ...
        // other npm-scripts
        // ...
        "prettier-write": "npx prettier --write .",
        "prettier-check": "npx prettier --check .",
        "pre-commit": "npm run prettier-check && npm run lint",
        "pre-push": "echo TODO pre-push",
        "configure-husky": "npx husky install && npx husky add .husky/pre-commit \"npm run pre-commit\" && npx husky add .husky/pre-push \"npm run pre-push\""
    }
}
```

Run the configure-husky script. This will make shure that Husky is installed and create the pre-commit and pre-push hook.

```
$ npm run configure-husky
```

Thats it! Everything is now configured. When a commit is made `$ git commit -m "My changes"` both code formatting using Prettier and linting using ESLint will be applied to the project.

### 2.4 The Workflow

Now thats everything regarding formatting and linting is done. Here is a step-by-step checklist.

1. Write your code
2. Stage files to be commited `$ git add .`
3. Commit files `$ git commit -m "My changes"`
4. Pre-Commit hook runs `prettier-check` and `ng lint`

If any errors are found, the commit is aborted. You now need to fix them manually or via the `$ ng lint --fix` command. Also run formatting on the files `$ npm run prettier-write`.

## 3. Micro Frontends Module Federation
