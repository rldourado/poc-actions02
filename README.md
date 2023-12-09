# ViaCEP App
This is a simple application to request and manage Brazilian postal codes.
Each module is independent and based in Flask to comunicate information.
Once a response is received from ViaCEP, it is keeped in "cache" and can be
listed ou searched.

## Modules Diagram
This simple diagram explains comunication between modules.
```
                    ┌──────┐
                    │viacep│third-party
                    └──────┘  service
                        ▲
                        │
┌────┐    ┌────┐     ┌──┴──┐    ┌────┐
│user├───►│main├─┬──►│quest├───►│keep│
└────┘    └────┘ │   └─────┘    └────┘
                 │                 ▲
                 │   ┌────────┐    │
                 ├──►│list_csv├────┤
                 │   └────────┘    │
                 │                 │
                 │   ┌─────────┐   │
                 ├──►│list_html├───┤
                 │   └─────────┘   │
                 │                 │
                 │   ┌──────────┐  │
                 ├──►│search_csv├──┤
                 │   └──────────┘  │
                 │                 │
                 │   ┌───────────┐ │
                 └──►│search_html├─┘
                     └───────────┘
```
- **viacep**: a third-party service to consult brazilian postal codes;
- **main**: facade to control and forward access to other modules;
- **quest**: module to search a postal code in viacep service;
- **keep**: module to keep all postal codes searched by quest module;
- **list_csv**: list all postal codes in keep module (CSV format);
- **list_html**: list all postal codes in keep module (html format);
- **search_csv**: search a postal code in keep module (CSV format);
- **search_html**: search a postal code in keep module (HTML format).

## Modules implementation
The status of each module is:
- **main**: working, automated tests;
- **quest**: working;
- **keep**: working;
- **list_csv**: working;
- **list_html**: working;
- **search_csv**: not implemented;
- **search_html**: not implemented;
- **utils**: working, automated tests;

## Prepare Development Environment
Install:
```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
```
Use python3.10 if possible.

Create a Virtual Environment:
```bash
# access pproject root directory
python3.10 -m venv .venv
source .venv/bin/activate
pip3 install -r requirements.txt
pip3 install -r requirements-dev.txt
```

Check code style with PyLint:
```bash
pip3 install pylint==2.15.9
python3 -m pylint *.py utils/*.py
python3 -m pylint *.py utils/*.py || :
```

Execute tests with PyTest:
```bash
pip3 install pytest==7.2.0
python3 -m pytest -v
```

Deactivate virtual environment:
```bash
deactivate
```

## Start Application Locally
In this example, all modules are running in the same terminal.
Please, consider use a terminal to each module.
```bash
export $(grep -v -e '^#' config/tests_local_vars.env)
python3 main.py &
python3 quest.py &
python3 keep.py &
python3 list_csv.py &
python3 list_html.py &
```

To test run:
```bash
wait_time='2s'
main_ip=localhost
curl -sS $main_ip:5000
sleep $wait_time
for pc in 02739000 02201000 22041011 22050002 32310210 33120510
do 
  curl -sS $main_ip:5000/quest/$pc
  sleep $wait_time
done
curl -sS $main_ip:5000/list_csv
sleep $wait_time
curl -sS $main_ip:5000/list_html
sleep $wait_time
curl -sS $main_ip:5000/list_html|grep -A9 22041-011
```

## Dockerfiles and images
Dockerfiles are in ./dockerfiles folder and there are one Dockerfile to each module.
To build all images use:
```bash
for x in main quest keep list_csv list_html
do 
  docker build -t cep-$x -f dockerfiles/$x.dockerfile .
done
```

To deploy all images locally use:
```bash
CONT=0
docker network create cep-network
for x in main quest keep list_csv list_html
do 
  docker run -d --name cep-$x --publish 500$CONT:500$CONT --net cep-network cep-$x:latest
  CONT=$(( $CONT + 1 ))
done
```

To check the logs of all services use:
```bash
for x in main quest keep list_csv list_html
do 
  echo MODULE=$x
  docker logs cep-$x
  echo -------------------------------------
  sleep 5s
  echo
done
```

To remove the application use:
```bash
for x in main quest keep list_csv list_html
do 
  docker container rm cep-$x --force
done
docker network rm cep-network
```

Use the same strategy to test but change de IP to the Docker host IP.

## Manifests
There are a folder with all necessary manifests to deploy application in Kuberntes.
The pipeline build and send images to Dockerhub, and updates Deployment manifests automatically with new image.
The example used to update Deployments may be used to update any information used in deployment.
To deploy the manifests use:
```bash
kubectl apply -f manifests/
kubectl get pod,deploy,service,configmap
```

To deploy the manifests in :
```bash
CEP_NAMESPACE=cep
kubectl -n $CEP_NAMESPACE apply -f manifests/
kubectl -n $CEP_NAMESPACE get pod,deploy,service,configmap
```

## Semantic Git Messages:
- FIX/HOTFIX: fixes in the code.
- FEAT: add new funcionality.
- UPDT: update information or configuration.
- PIPELINE: changes in pipelines.
- TRIG: initiate any pipeline.
- DOCS: change documentation files.
- REFACTOR: refactor code (rename variable, split files in mode modules, etc.).
- TEST: changes in unit tests (add, fix, remove, etc.).

## Known bugs
- The same postal code quest many time will be keeped many times


## Contact
Cícero Woshington: cicerow.ordb@gmail.com woshington_leite@atlantico.com.br +55 (88) 997293453


## Contact
Cícero Woshington: cicerow.ordb@gmail.com woshington_leite@atlantico.com.br +55 (88) 997293453

