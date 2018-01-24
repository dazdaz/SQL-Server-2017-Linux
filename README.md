<pre>
# Demo 1
# Quick demo of running the container version of SQL-Server-2017-Linux

docker run --name sql1 -d -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=S3^ret82!' -p 1433:1433 microsoft/mssql-server-linux:2017-latest

# Check that SQL Server 2017 on Linux was launched and is running as a Docker process
docker ps

# Run sqlcmd
docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 'S3^ret82!' -Q 'SELECT @@VERSION'

exit 0


# Demo 2
# Create the volume container – To view volumes, use “docker ps –a”
# Database is wrote here on the VM : /var/lib/docker/volumes/sqlvolume/
docker create -v /var/opt/mssql/data --name mssqldata microsoft/mssql-server-linux:2017-latest /bin/true

# Run SQL Server 2017 for Linux
docker run -d -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=S3^ret82!' -p 1433:1433 --volumes-from mssqldata --name sql1 microsoft/mssql-server-linux:2017-latest

# Run sqlcmd
docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 'S3^ret82!' -Q 'SELECT @@VERSION'

docker stop sql1
docker rm sql1
docker stop mssqldata
# Delete database (don't)
docker volume rm mssqldata
</pre>

Links :
https://www.slideshare.net/TravisWright4
