# dummy-web

# CICD using Github Action

This repo is a tutorial on how to setup CICD pipeline for our dummy web using Github Action.

## 0. Requirements

For this to work, you need to setup these first:
- Firebase account
- Firebase CLI installed
- Node v14.8 installed
- NPM v8 installed

## 1. Running in Local

Please make sure these commands are success in your Local

```bash
npm ci
npm run build
npm run lint
npm run test
```

Then you should also make sure our dummy web is accessible in local. Run this command and open localhost.

```bash
npm start
```

You should see something like this.

![](https://github.com/hezbymuhammad/dummy-web/blob/main/assets/npm-serve.png?raw=true)

## 2. Github Action

Github Action is automation builder and runner. Using Github Action, you can create and run your own CICD automation. There are several components of Github Action: workflow, events, jobs, actions, and runners. You can read more [here](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions).

![](https://docs.github.com/assets/cb-25535/images/help/images/overview-actions-simple.png)

To put it simply, you can use Github Action to run predefined scripts. For example, when you define your workflow as follows:

```yaml
name: learn-github-actions
on: [push]
jobs:
  check-bats-version:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '14'
      - run: npm install -g bats
      - run: bats -v
```

Github Action will run these steps in order when you push something to your repo:

1. Run on ubuntu server
2. Check out to your repository
3. Install Node v14
4. Install npm package `bats`
5. Run `bats`

## 3. Your First Pipeline

Our 1st objective is to automate build and deploy of our dummy web using CICD pipeline. We are going to use github action.

### 3. a. Setup Firebase Token

GitHub requires a `$FIREBASE_TOKEN` to be able to deploy our dummy web to Firebase.

```bash
❯ firebase login:ci

########
# you will get something like below message
########

Visit this URL on this device to log in:
https://accounts.google.com/o/oauth2/auth?client_id=....

Waiting for authentication...

########
# and after auth success, you will get something like below message
########

✔  Success! Use this token to login on a CI server:

1//....

Example: firebase deploy --token "$FIREBASE_TOKEN"
```

Copy your token (this one `1//....`)

Go to settings page on your repository, then go to sub menu Secrets. Enter your copied token there.

![](https://github.com/hezbymuhammad/dummy-web/blob/main/assets/gh-action.png?raw=true)

You then can see that your secret added

![](https://github.com/hezbymuhammad/dummy-web/blob/main/assets/gh-action-2.png?raw=true)

### 3. b. Initialize Firebase

First, you need to delete these 2 files: `.firebaserc` and `firebase.json`.

Then you can run `firebase init hosting`. Make sure you pick choices as follows

![](https://github.com/hezbymuhammad/dummy-web/blob/main/assets/firebase-init.png?raw=true)

### 3. c. Setup Workflow

There are several parts of Github Workflow we're going to explore in each sections. Let's examine our workflow file:

```yaml
name: CI

on:
  push:
    branches:
    - main

jobs:
  build-and-deploy:

    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [14.x]

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ matrix.node-version }}

    - name: Install dependencies
      run: npm install

    - name: Run the tests
      run: npm test

    - name: Build
      run: npm run build

    - name: Firebase Deployment
      uses: w9jds/firebase-action@master
      with:
        args: deploy --only hosting
      env:
        FIREBASE_TOKEN: ${{ secrets.FIREBASE_TOKEN }}
```

The top most part is the name of our workflow name which is `CI`.

```yaml
name: CI
```

Next section is called `triggers`. In our action, we define event that will trigger Github Action as every push on `main` branch.

```yaml
on:
  push:
    branches:
    - main
```

Next section is called `runner`. It's definition of what VM we're going to use to run our workflow. We define it as `ubuntu-latest`.

```yaml
runs-on: ubuntu-latest
```

Next section is called `steps`. It's grouped and ordered actions. A step can have name and action. You can define your own name. And you can pick any available action from action marketplace [here](https://github.com/marketplace?type=actions). In our workflow example below, we define step name as `Checkout Repository` and it's going to use `actions/checkout@v2`.

```yaml
steps:
    - name: Checkout repository
      uses: actions/checkout@v2
```

### 3. d. Running Pipeline

Let's run our pipeline. You can run it by pushing any commit to `main` branch. If everything succeed, you would see something like these

![](https://github.com/hezbymuhammad/dummy-web/blob/main/assets/cd-success-1.png?raw=true)

![](https://github.com/hezbymuhammad/dummy-web/blob/main/assets/cd-success-2.png?raw=true)

Then you can open your project at `https://<YOUR-PROJECT-ID>.firebaseapp.com`. Alternatively, you can also find deployed URL from the pipeline's log.

On my case in this repository, you can open https://dummy-web-b7e39.firebaseapp.com. This is your production URL.

### 4. Quests

To familiarize yourself, I provide 3 quests for you:

- Fork this repo and create your own pipeline
- In this repo, I only demonstrate CD part of CICD. Your quest is to implement CI part, every pull requests and commit to `main` branch should not break `npm run lint` and `npm run test`. If it breaks those 2 command, PR can't be merged.
- In this repo, I only demonstrate 1 environment deployment. The code goes directly to production URL (https://dummy-web-b7e39.firebaseapp.com). Your quest is to implement `staging` deployment.
  There will be 2 branches `main` and `staging`. 
  The `main` branch CICD is deployed to production (https://<YOUR-PROJECT-NAME>-<random string generated by gcp>.firebaseapp.com).
  The `staging` branch CICD is deployed to staging (https://<YOUR-PROJECT-NAME>-staging-<random string generated by gcp>.firebaseapp.com).

# Acknowledgement

The web implementation itself is from [emoji-search](https://github.com/ahfarmer/emoji-search).
This repo only is for providing guide on how to create build pipeline and deploy using github action. No intention of claiming emoji-search implementation.
