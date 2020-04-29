# bluemix-letsencrypt
A script for configuring [Let's Encrypt](https://letsencrypt.org) SSL certificates for CloudFoundry apps on IBM Cloud (formerly known as Bluemix).

Using the `--path` argument of the `cf map-route` command, you can configure a specific path to be directed to a separate app.  The benefit, in this situation, is that you can automate the configuration of SSL certificates for your custom domain applications by running the [letsencrypt certbot](https://github.com/certbot/certbot) code in a separate instance without disrupting your application.

```
NAME:
   map-route - Add a url route to an app

USAGE:
   cf map-route APP_NAME DOMAIN [--hostname HOSTNAME] [--path PATH]

EXAMPLES:
   cf map-route my-app example.com                              # example.com
   cf map-route my-app example.com --hostname myhost            # myhost.example.com
   cf map-route my-app example.com --hostname myhost --path foo # myhost.example.com/foo

OPTIONS:
   --hostname, -n   Hostname for the route (required for shared domains)
   --path           Path for the route
```

Firstly you must have the [IBM Cloud CLI](https://cloud.ibm.com/docs/cli?topic=cloud-cli-install-ibmcloud-cli) installed, custom domains created, DNS configured, and know your cloud foundry org and space.

Once ready:

1. download/clone this repo
2. install the requests and pyyaml packages (e.g. `pip install requests pyyaml`)
3. rename `domains.json.example` to `domains.json` and edit it:
   - enter your email address (e.g. for certificate renewal reminders)
   - enter your custom domain name and its corresponding hostnames

   Each [host].[domain] combination will become a separate DNS name in the SAN field of the requested certificate. Set the first host value to '.' to set the Subject Common Name to the name of the domain.

   Note: During testing, please set `staging` to `true` in order to keep load off the production Let's Encrypt environment and reduce the chance of hitting their rate limits (https://letsencrypt.org/docs/staging-environment/).
4. Log in to the IBM Cloud (e.g. `ibmcloud login`), set your target org and space (e.g. `ibmcloud target --cf -o your_org -s dev`), and finally run `python setup-app.py`. It will:
   1. push the cf-letsencrypt application
   2. map the routes needed for Let's Encrypt to verify that you own the domain
   3. initiate and complete the Let's Encrypt ACME protocol for obtaining a certificate
   4. download the resulting certificate files, and
   5. upload it into IBM Cloud for your custom domain
