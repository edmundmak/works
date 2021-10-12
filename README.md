# WTF2

The main integration layer. Hosted in [Thorntail](https://thorntail.io) (previously *Wildfly Swarm*)

### Active WTF2 Deployments
Env | URL | EB Application | RDS DB |
|--|--|--|--|
| `Dev` | `https://wtf-dev.iwyze.co.za` | `wtf-10-dev` | `wyze-pg-dev/wtf` |
| `Staging` | `https://wtf-staging.iwyze.co.za` | `wtf-20-staging` | `wyze-pg-staging/wtf` |
| `Production` | `https://wtf.iwyze.co.za` | `wtf-30-prod` |`wtf-30-prod` |

### Get it running locally

#### Prerequisite software

 - JDK8+
 - Redis
	 - `localhost:6379`
 - PostgreSQL 9.6+
	 - Create DB: `wtf`
	 - Default user: `rabbit`
	 - Default password: `rabbit01`
 - Git
 	- Greater than 2.13.0


#### Run
Pull the repo and run the following command in the root project directory to configure git hooks:

```

git config core.hooksPath .githooks

```
run the following command in the `wtf/project` directory:

```

./mvnw install

```
This command will build all modules and run the application.

For a quick run where no re-compilation is necessary (after having ran the above command):
```

./mvnw thorntail:run -f ".\wtf-war\pom.xml"

```
Start without running tests, checkstyle and typescript generation:
```
mvnw.cmd install -P faster
```

**Note**: It is important to use the Maven wrapper script (`mvnw` not `mvn`) to ensure the correct version of maven is used and that the additional functionality [defined here](./wtf/util/CustomMVNWrapper/README.txt) is available.


#### Testing
Use the [RequestReplay](./wtf/util/RequestReplay) tool to test real world API calls against your local deployment (or any other of your choosing)

To run a single test file and generate reports use:

```
mvnw.cmd test -Dtest=DBSuburbRiskUpdateServiceTest -f wtf-ejb\pom.xml
```

## PostGis setup and ag_subtown (Suburb lookup from shape data)
**Install PostGis on your local PostgreSql server:**
- Check your version compatability on the [Compatability Matrix](https://trac.osgeo.org/postgis/wiki/UsersWikiPostgreSQLPostGIS)
- Download the [Zip and Installer files](http://download.osgeo.org/postgis/windows) from https://postgis.net/windows_downloads for your PostgreSql version.
- Unzip it and manually merge all the folders with your PostgreSql installation directory. Note: There is a batch file to do this, however, if you don't have a specific installation, it will not work, so rather manually merge the folders.
- Then on the wtf database, run:
```sql
CREATE EXTENSION postgis;
```
- You now have PostGis installed and can make use of the functions e.g. [ST_DWithin](https://postgis.net/docs/ST_DWithin.html)

**Get the suburb shape data:**
- To get the shape data download and unzip the SQL script from https://iwyze-share.s3-eu-west-1.amazonaws.com/ag_subtown.zip
- Run the script using psql - do not try using the ui, it will explode 💥
- psql should be in `C:\Program Files\PostgreSQL\9.6\bin\psql.exe`
- Open cmd
- Run:
```bash
psql.exe -U rabbit -d wtf -a -f "<location where you unzip\>ag_subtown.sql" >nul 2>&1
```
- It may take some time to import all the data - be patient 🤒

Example PostGis query:
```sql
SELECT ST_Distance(ST_MakePoint(29.6644, -26.45485)::geography, subtown_shape), *
FROM ag_subtown
WHERE ST_DWithin(ST_MakePoint(29.6644, -26.45485)::geography, subtown_shape, 50000) -- 50km range
```
