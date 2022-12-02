# Instruction on how to setup erpnext via docker locally

## Note
### Values mentioned here are for demo purposes and should be replaced with appropriate ones for your setup
#### 1. TRAEFIK_DOMAIN IN STEP 1
#### 2. EMAIL IN STEP 1
#### 3. HASHED PASSWORD (ROOT) IN STEP 1
#### 4. DB_PASSWORD IN STEP 5
#### 5. DB_PASSWORD IN STEP 6 TO THAT GIVEN IN STEP 5
#### 6. SITES LISTED IN STEP 6
#### 7. value given to --mariadb-root-password in step 9 to that given in step 5
#### 8. value given to --admin-password in step 9
#### 9. value given to --new-site in step 9 to that given in step 6 
<pre>

</pre>

# Step 1
## Clone this directory

``` bash
git clone https://github.com/Yaltopia/frappe_docker.git
cd frappe_docker
```

## step 2
### make a directory in home called gitops where all configuration data will be kept



``` bash
mkdir ~/gitops
```

## step 3
### create traefik.env file and fill it with configuration information with the below command

``` bash
echo 'TRAEFIK_DOMAIN=traefik.yaltopia.site' > ~/gitops/traefik.env
echo 'EMAIL=yared-deyaso@yaltopia.com' >> ~/gitops/traefik.env
echo 'HASHED_PASSWORD='$(openssl passwd -apr1 root | sed 's/\$/\\\$/g') >> ~/gitops/traefik.env

```

## Step 4
### start the traefik docker container

``` bash
docker compose --project-name traefik \
  --env-file ~/gitops/traefik.env \
  -f docs/compose/compose.traefik.yaml  up -d

```

# Step 5
## create mariadb.env file and fill it with configuration information with the below command and then run 

``` bash
echo "DB_PASSWORD=root" > ~/gitops/mariadb.env
docker compose --project-name mariadb --env-file ~/gitops/mariadb.env -f docs/compose/compose.mariadb-shared.yaml up -d

```

# step 6
## create erpnext-one.env file and fill it with configuration information with the below command and then run 

``` bash
cp example.env ~/gitops/erpnext-one.env
sed -i 's/DB_PASSWORD=123/DB_PASSWORD=root/g' ~/gitops/erpnext-one.env
sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' ~/gitops/erpnext-one.env
sed -i 's/DB_PORT=/DB_PORT=3306/g' ~/gitops/erpnext-one.env
echo 'ROUTER=erpnext-one' >> ~/gitops/erpnext-one.env
echo "SITES=\`test.yaltopia.site\`,\`dev.yaltopia.site\`" >> ~/gitops/erpnext-one.env
echo "BENCH_NETWORK=erpnext-one" >> ~/gitops/erpnext-one.env

```

# step 7
## use the docker command below to create a custom docker-compose yaml file erpnext-one.yaml in the gitops directory

``` bash
docker compose --project-name erpnext-one \
  --env-file ~/gitops/erpnext-one.env \
  -f compose.yaml \
  -f overrides/compose.erpnext.yaml \
  -f overrides/compose.redis.yaml \
  -f docs/compose/compose.multi-bench.yaml config > ~/gitops/erpnext-one.yaml
```

# step 8
## start the erpnext one bench container 
``` bash
docker compose --project-name erpnext-one -f ~/gitops/erpnext-one.yaml up -d
```

# step 9
## create the sites you mentioned in erpnext-one.env in step 6

### note: will have to be repeated for each site listed in sites in step six

``` bash
docker compose --project-name erpnext-one exec backend \
  bench new-site test.yaltopia.site --mariadb-root-password root --install-app erpnext --admin-password root

```



