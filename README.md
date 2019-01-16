## Demo 1
* Quick demo of running the container version of SQL-Server-2017-Linux
```
docker run --name sql1 -d -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=S3^ret82!' -v sqlvolume:/var/opt/mssql -p 1433:1433 mcr.microsoft.com/mssql/server:2017-latest-ubuntu

# Check that SQL Server 2017 on Linux was launched and is running as a Docker process
docker ps

# Run sqlcmd
docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 'S3^ret82!' -Q 'SELECT @@VERSION'

exit 0
```
```
You can hardcore the release number of SQL Server 2017 - mcr.microsoft.com/mssql/server:2017-CU10
mcr.microsoft.com/mssql/rhel/server:vNext-CTP2.0 SQL Server 2019
```

## Demo 2
* Create the volume container – To view volumes, use “docker ps –a”
* Database is wrote here on the VM : /var/lib/docker/volumes/sqlvolume/
```
docker create -v /var/opt/mssql/data --name mssqldata microsoft/mssql-server-linux:2017-latest /bin/true

# Run SQL Server 2017 for Linux
docker run -d -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=S3^ret82!' -p 1433:1433 --volumes-from mssqldata --name sql1 mcr.microsoft.com/mssql/server:2017-CU8

# Run sqlcmd
docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 'S3^ret82!' -Q 'SELECT @@VERSION'

docker stop sql1
docker rm sql1
docker stop mssqldata
# Delete database (don't)
docker volume rm mssqldata
```

Links :
https://www.slideshare.net/TravisWright4
https://www.youtube.com/watch?v=DEUeAgLCAng Excellent 10 minute video on howto configure pacemaker for SQL Server on Linux

## Demo 3
* Execute all the SQL files in a mounted folder:
```
  mssql:
    image: mcr.microsoft.com/mssql/server:2017-latest
    environment: 
      - SA_PASSWORD=Admin123
      - ACCEPT_EULA=Y
    volumes:
     - ./data/mssql:/scripts/
    command:
      - /bin/bash
      - -c 
      - |
        # Launch MSSQL and send to background
        /opt/mssql/bin/sqlservr &
        # Wait 30 seconds for it to be available
        # (lame, I know, but there's no nc available to start prodding network ports)
        sleep 30
        # Run every script in /scripts
        # TODO set a flag so that this is only done once on creation, 
        #      and not every time the container runs
        for foo in /scripts/*.sql
          do /opt/mssql-tools/bin/sqlcmd -U sa -P $$SA_PASSWORD -l 30 -e -i $$foo
        done
        # So that the container doesn't shut down, sleep this thread
        sleep infinity
```
* https://github.com/Microsoft/mssql-docker/issues/11#issuecomment-452272205


## Demo 4
```
kubectl create secret generic mssql --from-literal=SA_PASSWORD="{SA PASSWORD}"
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mssql
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  minReadySeconds: 5
  selector:
    matchLabels:
      app: mssql
  template:
    metadata:
      labels:
        app: mssql
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: mssql
        image: microsoft/mssql-server-linux:latest
        ports:
        - containerPort: 1433
        env:
        - name: MSSQL_PID
          value: "Developer"
        - name: ACCEPT_EULA
          value: "Y"
        - name: SA_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mssql
              key: SA_PASSWORD
        volumeMounts:
        - name: mssqldb
          mountPath: /var/opt/mssql
      volumes:
      - name: mssqldb
        persistentVolumeClaim:
          claimName: mssql-data
---
apiVersion: v1
kind: Service
metadata:
  name: mssql
spec:
  selector:
    app: mssql
  ports:
    - protocol: TCP
      port: 1433
      targetPort: 1433
  type: LoadBalancer

```

* http://aka.ms/sqlcontainers
* https://github.com/Microsoft/sqllinuxlabs
* https://github.com/Microsoft/bobsql/tree/master/demos
* https://github.com/Microsoft/sql-server-samples

What?s new in SQL Server on Linux and containers
https://medius.studios.ms/Embed/Video/IG18-BRK3228?SFYT=true
https://mediusprodstatic.studios.ms/presentations/Ignite2018/BRK3228.pptx

Inside SQL Server containers
https://medius.studios.ms/Embed/Video/IG18-BRK4017?SFYT=true
https://mediusprodstatic.studios.ms/presentations/Ignite2018/BRK4017.pptx

Databases, containers, and pods: SQL Server on Kubernetes
https://medius.studios.ms/Embed/Video/IG18-THR2096?SFYT=true
https://mediusprodstatic.studios.ms/presentations/Ignite2018/THR2096.pptx

Deploying a highly available SQL Server solution in Kubernetes
https://medius.studios.ms/Embed/Video/IG18-THR2171?SFYT=true
