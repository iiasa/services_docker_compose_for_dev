# How to start accelerator service locally
# prerequisites
1. setup docker engine 
1. install kubernetes
1. enable kubernetes with docker desktop
# general
1. Copy .env* files under root directory and remove .sample suffix
1. Create registry_auth if not exists
# .env
1. Complete the directories in .env, the path should be $pwd/ $(the actual directory as written in the env)
# .env.web.be
# .env.web.fe
# misc
    create localip entry in /etc/hosts
    # ACCELERATOR IPS
    xxx.xxx.xxx.xxx localip
    xxx.xxx.xxx.xxx registry
    xxx.xxx.xxx.xxx web_be
    xxx.xxx.xxx.xxx indicates your ip address under iiasa network, check it out using ifconfig command
# titiler
1. clone the repo `docker compose -f docker-compose.dev.yml up minio --build`
1. use commit `git checkout 6bc1429` for the time being  
1. create self signed certificates that will be used for titiler, as well as minio with following oneliner command
  `openssl req -x509 -newkey rsa:2048 -keyout private.key -out public.crt -days $DAYS -nodes -subj "/CN=localip"`
1. pub self signed certificates under certs, copy and rename it as minio-cert.crt under dockerfiles directory
# minio
1. copy the above created private public set and place them under certs directory
1. execute `docker compose -f docker-compose.dev.yml up minio --build`
1. access minio by https:localhost:9001 and create a set of access key.
1. fill these 2 fields with the created access key inside .env.web.be and .env.celery INITIAL_S3_API_KEY= INITIAL_S3_SECRET_KEY=
# .env.celery
1. in .env.celery change `IMAGE_REGISTRY_URL=registry:8443`, `IMAGE_REGISTRY_USER=myregistry`, `IMAGE_REGISTRY_PASSWORD=myregistrypassword`
1. convert ~/.kube/config to json and then base64 string, fill field `WKUBE_SECRET_JSON_B64` with the processed base64 string
1. use command `python3 -c "import sys, yaml, json; print(json.dumps(yaml.safe_load(sys.stdin), indent=2))" < ~/.kube/config > config.json` to convert kubernetes config yaml as a json file
1. execute `base64 -w 0 ./config.json` to convert the config json to a base64 string
# registry
1. execute command `docker run --rm --entrypoint htpasswd httpd:2 -Bbn myregistry myregistrypassword > /home/caosimin/Development/accelerator_service/registry_auth/htpasswd` to generate htpasswd
# database
1. execute `docker compose -f docker-compose.dev.yml up db --build` to build the db image
1. enter the db container with `docker exec -ti db /bin/sh`
1. create databases inside the container with `su -- postgres -c "createdb accelerator"` and `su -- postgres -c "createdb acceleratortest"`

# startup the project
# `NOTE`
1. make sure to config ~/.kube/config to json before convert it to base64 string
2. inside `generic_scenario_explorer_backend` ignore the .env.sample, the configs are passing down as the containers are orchestrated
