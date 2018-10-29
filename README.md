## Demo 1
* Quick demo of running the container version of SQL-Server-2017-Linux
```
docker run --name sql1 -d -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=S3^ret82!' -v sqlvolume:/var/opt/mssql -p 1433:1433 mcr.microsoft.com/mssql/server:2017-CUB

# Check that SQL Server 2017 on Linux was launched and is running as a Docker process
docker ps

# Run sqlcmd
docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 'S3^ret82!' -Q 'SELECT @@VERSION'

exit 0
```

## Demo 2
* Create the volume container – To view volumes, use “docker ps –a”
* Database is wrote here on the VM : /var/lib/docker/volumes/sqlvolume/
```
docker create -v /var/opt/mssql/data --name mssqldata microsoft/mssql-server-linux:2017-latest /bin/true

# Run SQL Server 2017 for Linux
docker run -d -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=S3^ret82!' -p 1433:1433 --volumes-from mssqldata --name sql1 mcr.microsoft.com/mssql/server:2017-CUB

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
