Azure Dev Spaces
================

Create an AKS cluster to test out Azure Dev Spaces:

```sh
az group create --name devspaces --location australiaeast
az aks create -g devspaces -n devspaces --location australiaeast --generate-ssh-keys
```

Register for Helm 3 preview (unless it has since gone GA) for Dev Spaces: https://docs.microsoft.com/en-us/azure/dev-spaces/how-to/helm-3

Install Dev Spaces in your cluster:

```sh
az aks use-dev-spaces -g devspaces -n devspaces --space dev --yes
azds show-context
azds space list
```

Clone Azure Dev Spaces Samples repository
-----------------------------------------

```sh
git clone https://github.com/Azure/dev-spaces
cd dev-spaces
```

Demo 1 - Single nodeJS service
------------------------------

```sh
azds space select -n dev/azureuser2 -y
```

```sh
cd samples/nodejs/getting-started/webfrontend
code .
```

Open Command Palette (CTRL+SHIFT+P):

* Select `Azure Dev Spaces: Prepare configuration files for Azure Dev Spaces`
* Choose `Yes` for public endpoint

Debug app:

* Debug with configuration - Launch Server (AZDS)
* Examine generated files (Helm, Dockerfile, azds.yaml)
* Access website URL

Update code - `server.js`, line 13:

* Stop Debugging
* `res.send('Hello from webfrontend in Azure');`
* Launch Server (AZDS)
* Test changed in app
* NOTE: Can also Restart Debugging (CTRL+SHIFT+F5) without stopping and relaunching server

Set breakpoint:

* Toggle breakpoint (F9)
* Test app, see breakpoint hit
* Examine vars, callstack
* Toggle breakpoint (F9)
* Continue debugging (F5)

Live reload:

* Stopping Debugging
* Debug with configuration - Attach to a Server (AZDS)
* Edit code, save file
* See changes in browser

Cleanup:

```sh
# cd dev-spaces/samples/nodejs/getting-started/webfrontend
azds down
rm .dockerignore azds.yaml Dockerfile
rm -rf charts
```

Demo 2 - Multiple dependent services
------------------------------------

```sh
azds space select -n dev/azureuser2 -y
```

Open follow two projects in VSCode windows:

* samples/nodejs/getting-started/mywebapi
* samples/nodejs/getting-started/webfrontend

In `webapi` vscode:

* Select from Command Palette: `Azure Dev Spaces: Prepare configuration files for Azure Dev Spaces`
* Set a breakpoint in the GET / handler
* Debug with F5 - Launch Server (AZDS)

In `webfrontend` vscode:

* Comment/uncomment sections of code in `server.js`
* Uncomment `var request = require('request');`
* Comment the exiting `app.get('/api', ...`
* Uncomment

```js
app.get('/api', function (req, res) {
   request({
      uri: 'http://mywebapi',
      headers: {
         /* propagate the dev space routing header */
         'azds-route-as': req.headers['azds-route-as']
      }
   }, function (error, response, body) {
       res.send('Hello from webfrontend and ' + body);
   });
});
```

* Comment the `server.close()` line at the end of `server.js`
* Set breakpoint in the GET / handler callback
* Debug with F5 - Launch Server (AZDS)

Stop debugging both apps.

Cleanup:

* Run `azds down` in both app directories (if required).
* In `dev-spaces/samples/nodejs/getting-started/webfrontend`, run:

```sh
rm .dockerignore azds.yaml Dockerfile
rm -rf charts
```

Demo 3 - Team development on AKS
--------------------------------

* Deploy bikesharingapp to `dev` namespace

```sh
cd samples/BikeSharingApp/
azds show-context
# Edit samples/BikeSharingApp/charts/values.yaml and replace placeholder with HostSuffix
cd charts/
helm install bikesharingsampleappsampleapp . --dependency-update --namespace dev --atomic
```

* Create child dev spaces

```sh
azds space select -n dev/azureuser1 -y
azds space select -n dev/azureuser2 -y
azds space list
```

* Change into a service directory and make code changes

```sh
vim samples/BikeSharingApp/BikeSharingWeb/components/Header.js
```

* Deploy private changes to AKS

```sh
azds space select -n dev/azureuser2
cd ../BikeSharingWeb/
azds up
# CTRL+C to stop streaming logs
```

* View the modified web app in yor local dev space:

http://azureuser2.s.dev.bikesharingweb.XXXXXX.XXXXX.azds.io/

* Verify existing web app in `dev` dev space is unchanged:

http://dev.bikesharingweb.XXXXXX.XXXXX.azds.io/

* Cleanup

```sh
azds down # cleanup child dev space
```

Commands
--------

* `azds show-context`
* `azds list-uris`
* `azds list-uris --all`
* `azds space list`
* `azds space select -n <space_name> -y`
* `azds list-up`
* `azds controller list`

Upgrading CLI
-------------

Upgrade `azds` on Linux:

```sh
curl -L https://aka.ms/get-azds-linux | bash
```
