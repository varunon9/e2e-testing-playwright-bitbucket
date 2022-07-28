# e2e-testing-playwright-bitbucket
Bitbucket yaml configuration file for performing end to end integration testing using playwright

## Objective

Write a configuration file in yaml for bitbucket pipeline to run integration testing on every Pull request raised to master branch

Source Code: https://github.com/varunon9/e2e-testing-playwright-bitbucket/blob/master/bitbucket-pipelines.yml

## Resources

1. Get started with Bitbucket Pipelines: https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/
2. Getting started with Playwright: https://playwright.dev/docs/intro
3. Bitbucket Pipeline for Playwright: https://playwright.dev/docs/ci#bitbucket-pipelines
4. Setting up SSL in local using mkcert: https://github.com/FiloSottile/mkcert
5. SSL HTTP proxy using a self-signed certificate: https://github.com/cameronhunter/local-ssl-proxy#readme

## Breakdown of configuration file

### 1

```
image: mcr.microsoft.com/playwright:v1.24.0-focal
```

Base docker image for Bitbucket pipeline. Source: https://github.com/microsoft/playwright/blob/main/utils/docker/Dockerfile.focal

### 2

```
pipelines:
  pull-requests:
    'master':
```

Run the steps on every pull request raised to master.

### 3

```
- parallel:
          - step:
              name: e2e integration test cases
              ...
          - step:
              name: Code linting
              ...
          - step:
              name: Checking validity of types 
              ...       
```

Run these tasks in parallel.

### 4

```
caches:
  - node
```

Cache node_modules to save build time.

### 5

```
echo "127.0.0.1	localhost dev.indmoney.com" >> /etc/hosts
```

Make an entry in hosts file so that dev.indmoney.com points to localhost

### 6

```
apt update && apt install libnss3-tools
```

Install certutil on container. Reference: https://github.com/FiloSottile/mkcert

### 7

```
- curl -JLO "https://dl.filippo.io/mkcert/latest?for=linux/amd64"
- chmod +x mkcert-v*-linux-amd64
- cp mkcert-v*-linux-amd64 /usr/local/bin/mkcert
```

Install mkcert using pre-built binaries. Reference: https://github.com/FiloSottile/mkcert

### 8

```
- mkcert -install
- mkcert dev.indmoney.com
```

Self-signed trusted certificate using mkcert for domain dev.indmoney.com
Reference: https://github.com/cameronhunter/local-ssl-proxy#readme

### 9

```
- yarn install
- ./node_modules/.bin/playwright install
```

Install project dependencies and playwright testing framework

### 10

```
- (yarn dev&)
```

Start `yarn dev` in background. This will start nodejs server on port 3000

### 11

```
- (yarn ssl-proxy&)
```

Create a local SSL proxy in background.
This resolves to `npx local-ssl-proxy --key dev.indmoney.com-key.pem --cert dev.indmoney.com.pem --source 443 --target 3000`

### 12

```
- yarn test
```

Start executing e2e test cases. This resolves to `playwright test`


