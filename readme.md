#### RUN TESTS
#### 1. DOWNLOAD GIT FILES FROM GITHUB
```
sudo git clone https://github.com/edersonrmachado/dockerNetflowTest.git
cd dockerNetflowAppSolTest
```
#### 2. PERMISSIONS TO FILES BE EXECUTABLES
```
sudo chmod +x appAndSol.sh sequenceOfTestsAppandSol.sh 
```
#### 2. START DOCKER
```
service docker start
```
#### 3. PULLING IMAGES FROM DOCKER HUB
```
docker image pull mysql:5.7 
docker image pull wordpress:php7.3 
docker image pull alpine:3.10.3 
docker image pull aqualtune/netflow_collector:latest  
docker image pull aqualtune/netflow_data_export:latest
```

#### 4. DEPLOY APPLICATION WITH SOLUTION TEST

```
sudo bash appAndSol.sh 2 3 appAndSol2x.csv
```  
first argument: number of app&sol to deploy
second argument: number of times that test will be run
third argument: name of csv file to store results
#### 5. DEPLOY MULTIPLES TESTS WITH BASH FILE
```
sudo bash sequenceOfTestsAppandSol.sh 
``` 