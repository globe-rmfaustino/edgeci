# Edge Continuous Integration Commands
`edgeci` is a command line utility for [Apigee Edge](https://apigee.com/api-management) to help with continuous integration. The commands help to pull or push API proxy and configuration information between Apigee Edge and a source control repository like git.

To install:

```
$ npm install -g edgeci
```

Setup your Apigee Edge username and password in environment variables `$EDGE_USERNAME` and `$EDGE_PASSWORD`

```
$ export EDGE_USERNAME=<your Apigee Edge username>
$ export EDGE_PASSWORD=<your Apigee Edge password>
```

Usage:

```
$ edgeci --help
usage: edgeci.js [-h] [-v] [-q] [-V] {pull,push} ...

edgeci command line tool

Optional arguments:
  -h, --help     Show this help message and exit.
  -v, --version  Show program version number and exit.
  -q, --quite    no console logs
  -V, --verbose  verbose logging to console

command:
  {pull,push}
    pull         pull help
    push         push help
```

# Why?
We already have [Maven Deploy Plugin](https://github.com/apigee/apigee-deploy-maven-plugin) and [Grunt Deploy Plugin](https://github.com/apigeecs/apigee-deploy-grunt-plugin). Why do we need another build and deploy tool for Edge?

Both the Maven and Grunt plugins are built with the idea that Edge API development should happen on a code editor. But using a code editor for Edge API development is not developer friendly. Especially compared to Edge Management UI.
- No prefilled policy XMLs available on code editors unlike Management UI. Development involves a lot of toggles to copy paste XML configs from docs reference or from Management UI itself.
- There is really no compile, build or deploy tasks involved in these plugins. They just zip and upload a fixed folder structure for API proxy. The actual compile, build and deploy happens after the proxy bundle is uploaded to Edge management server. So, is a build tool the best choice for the task at hand?
- The _test-change-deploy-test_ feedback loop is longer with code editor compared to development using Management UI.

# So ...
`edgeci` tool is built on the opinion that Edge Management UI is the best place to develop API proxies. With this, continuous integration steps are:
1. A developer works on an API proxy in a development environment on Edge Management UI, testing the changes with curl, Postman, Apickli, etc.
2. When the developer is satisfied with the proxy behavior, he/she executes `edgeci pull` command to export the API proxy to local git repo.
3. On the local git repo, developer compares the changes with previous version and commits/pushes the changes to remote git repo.
4. The push to the remote git repo is picked up by a CI task (Ex. Jenkins git plugin)
5. The next CI step deploys the new changes into a staging environment on Edge using `edgeci push` command.
6. Once deployed, the next CI step executes automated tests like Apickli.
7. Finally CI summarizes the build results.

# `edgeci pull` Command
`edgeci pull` command exports one or more API proxies from an Edge org into the specified destination folder.

```
$ edgeci pull --help
usage: edgeci.js pull [-h] -o ORG -p [PROXY [PROXY ...]] [-d DESTINATION] [-c]
                      [-i INTERVAL]


Optional arguments:
  -h, --help            Show this help message and exit.
  -o ORG, --org ORG     name of Edge org to pull proxy from
  -p [PROXY [PROXY ...]], --proxy [PROXY [PROXY ...]]
                        proxies to pull, space separated names
  -d DESTINATION, --destination DESTINATION
                        destination dir to download proxy files
  -c, --continuous      continuously check for updates
  -i INTERVAL, --interval INTERVAL
                        interval to check for updates in seconds
```


# `edgeci push` Command

`edgeci push` command imports one or more API proxies to an Edge org from the specified source folder.

```
$ edgeci push --help
usage: edgeci.js push [-h] -o ORG -p [PROXY [PROXY ...]] [-s SOURCE] [-u]
                      [-e ENV]


Optional arguments:
  -h, --help            Show this help message and exit.
  -o ORG, --org ORG     name of Edge org to push proxy to
  -p [PROXY [PROXY ...]], --proxy [PROXY [PROXY ...]]
                        proxies to push, space separated names
  -s SOURCE, --source SOURCE
                        source dir to pick proxy files to upload
  -u, --update          update current revision
  -e ENV, --env ENV     deploy to env specified
```



# Examples - Pull

To pull a proxy named proxy1 from org1 to destination folder `./apis`

```
$ edgeci pull --org org1 --proxy proxy1 --destination ./apis
```

To pull three proxies named proxy1, proxy2, proxy3 from org1 to destination folder `./apis`

```
$ edgeci pull --org org1 --proxy proxy1 proxy2 proxy3 --destination ./apis
```

To pull all proxies from org1 to destination folder `./apis`

```
$ edgeci pull --org org1 --proxy all --destination ./apis
```

Continuously pull proxy1 from org1 to destination folder `./apis`. Check every 15 seconds

```
$ edgeci pull --org org1 --proxy proxy1 --destination ./apis --continuous --interval 15
```

# Examples - Push

To push a proxy named proxy1 to org1 from source folder `./apis`

```
$ edgeci push --org org1 --proxy proxy1 --source ./apis
```

To push three proxies named proxy1, proxy2, proxy3 to org1 from source folder `./apis`

```
$ edgeci push --org org1 --proxy proxy1 proxy2 proxy3 --source ./apis
```

To push all proxies to org1 from source folder `./apis`

```
$ edgeci push --org org1 --proxy all --source ./apis
```

# Use as module
`edgeci` can also be used as a module within your nodeJS scripts.

```JavaScript
var edgeci = require("./edgeci.js");
edgeci.pull("username@email.com", "password", "org", ["proxy1", "proxy2"], "destination");
```

```JavaScript
var edgeci = require("./edgeci.js");
edgeci.push("username@email.com", "password", "org", ["proxy1", "proxy2"], "source");
```

# TODOs

## Items to Pull(Export) / Push(Import)
- [ ] Develop
	- [ ] Specs
	- [√] API Proxy
	- [ ] Shared Flow
- [ ] Publish
	- [x] Developers (data not config)
	- [x] Apps (data not config)
	- [ ] API Products
- [ ] Analyze
	- [ ] Reports
- [ ] Admin
	- [ ] Users
	- [ ] Roles
	- [ ] Environments
		- [ ] Caches
		- [ ] Flow Hooks
		- [ ] Key Value Maps
		- [?] References
		- [?] Target Servers
		- [?] TLS Keystores
		- [?] Virtual Hosts
- [ ] Miscellaneous
	- [ ] Monetization
	- [ ] Microgateway

## Folders struture
- apis
	+ specs
	+ proxies
	+ sharedflows
	+ products
- admin
	+ reports
	+ users
	+ roles
- env
	+ caches
	+ flowhooks
	+ kvms
	+ refs
	+ targets
	+ keystores
	+ vhs
