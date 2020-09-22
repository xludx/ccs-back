
# About Particl Crowdfunding System (PCS)

Particl Crowdfunding System (PCS) is a simple web system for capturing donations made to fund community projects

# CCS Deployment Quickstart

## Requirements
```
docker
docker-compose
```

## Deployment

Checkout and configure CCS backend, frontend and proposals repositories (replace `<REPOSITORY_CCS_BACKEND>`,
 `<REPOSITORY_CCS_FRONTEND>`, `<REPOSITORY_CCS_PROPOSALS>` with the actual URLs)

```
cd YOUR_PROJECT_DIR

git clone <REPOSITORY_CCS_BACKEND> ccs-back
git clone <REPOSITORY_CCS_FRONTEND> ccs-front
git clone <REPOSITORY_CCS_PROPOSALS> ccs-back/storage/app/proposals
```

create the .env file.

```
cd ccs-back
cp .env.example .env
```


Spin up MYSQL server, create new database, user and grant user access to it  
Open `.env` in editor of choice and edit the following lines:  
> `COIN` - choose one of supported coins: `monero`, `zcoin` or `particl`
> `REPOSITORY_URL` - CCS proposals Github URL or GitLab API endpoint (e.g. https://\<GITLAB_DOMAIN>/api/v4/projects/\<PROJECT_ID>)>  
> `GITHUB_ACCESS_TOKEN` - leave empty if you are not using Github or visit https://github.com/settings/tokens to generate new `public_repo` token
```
APP_URL=http://<HOSTNAME>

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=<DB_NAME>
DB_USERNAME=<DB_USER_NAME>
DB_PASSWORD=<DB_USER_PASSWORD>

RPC_URL=http://127.0.0.1:28080/json_rpc
RPC_USER=
RPC_PASSWORD=

COIN=<COIN>

REPOSITORY_URL=<REPOSITORY_URL>
GITHUB_ACCESS_TOKEN=
```

Start everything.
```
docker-compose up
```



## TODO

- cron jobs
- particl support
- nginx not needed, use traefik when deploying to prod
- ...


Set up a cron job that will run periodic updates (every minute) and generate static HTML files
```
* * * * * git -C /var/www/html/ccs-back/storage/app/proposals/ pull; php /var/www/html/ccs-back/artisan schedule:run; jekyll build --source /var/www/html/ccs-front --destination /var/www/html/ccs-front/_site
```

Instead of scheduling a cron job you can run the following commands in no particular order
1. Update CCS system proposals intenal state
    ```
    php /var/www/html/ccs-back/artisan proposal:process
    php /var/www/html/ccs-back/artisan generate:addresses
    php /var/www/html/ccs-back/artisan wallet:notify
    php /var/www/html/ccs-back/artisan proposal:update
    ```
2. Process incoming donations  
*Run it either on new block/tx notification or schedule it to run every minute or so*
    ```
    php /var/www/html/ccs-back/artisan particl:notify
    ```
1. Generate static HTML files
    ```
    jekyll build --source /var/www/html/ccs-front --destination /var/www/html/ccs-front/_site
    ```
2. Get the full list of processed transactions in JSON format
    ```
    php /var/www/html/ccs-back/artisan deposit:list
    ```
