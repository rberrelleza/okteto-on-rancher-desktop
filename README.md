# Run Okteto in Rancher Desktop

This guide will walk you through the process of installing Okteto in Rancher Desktop

## Installation Requirements
Before you begin, make sure you have the following command-line interfaces (CLIs) installed on your machine:

okteto cli >= 3.5.0 (okteto installation guides)
helm >= 3.14 (helm installation guides)

You'll also need:

- An Okteto License
- Rancher Desktop installed with Kubernetes enabled

## Getting your Okteto License
A license is mandatory to use Okteto. You'll receive a license key as part of your subscription to Okteto. If you haven't received it, please open a support ticket.

If you are interested in evaluating Okteto, [sign up for our free 30 days trial](https://www.okteto.com/free-trial/). No credit card required.


## Installing Okteto
Okteto is installed using a Helm chart. Let's start the process:

### Disable Traeffik
Uncheck Enable Traefik from the Kubernetes Settings page to disable Traefik. You may need to exit and restart Rancher Desktop for the change to take effect.

### Add the Okteto Helm repository

You'll need to add the Okteto Helm repository to be able to install Okteto:

```
helm repo add okteto https://charts.okteto.com
helm repo update
```

### Create the Helm configuration file
In order to install Okteto you need to first create a config.yaml for the installation process. Replace license with your own values, and initialize your Helm configuration file as shown below.

> These settings will install Okteto with the minimum configuration and resource consumption, this isn't recommended for production.

```
license:  <YOUR_LICENSE>

api:
  replicaCount: 1

buildkit:
  network:
    mode: none
  replicaCount: 1
  resources:
    requests:
      cpu: 100m
      memory: 100Mi

frontend:
  replicaCount: 1

okteto-nginx:
  controller:
    replicaCount: 1

ingress-nginx:
  controller:
    replicaCount: 1
    service:
      type: NodePort

defaultBackend:
  replicaCount: 1

registry:
  replicaCount: 1

regcredsManager:
  replicas: 1

resourceManager:
  enabled: false

sshAgent:
  replicaCount: 1

telemetry:
  enabled: false

webhook:
  replicaCount: 1
```

### Installing the Okteto Helm chart
Install the latest version of Okteto by running:

```
helm upgrade --install okteto okteto/okteto -f config.yaml --namespace=okteto --create-namespace --version=1.30.1
```

After a few seconds, all the resources will be created. The output will look something like this:

```
Release "okteto" has been upgraded. Happy Helming!
NAME: okteto
LAST DEPLOYED: Mon Feb  3 22:21:49 2025
NAMESPACE: okteto
STATUS: deployed
REVISION: 13
TEST SUITE: None
NOTES:
Congratulations! Okteto is successfully installed!

Follow these steps to access your Okteto instance:

1. Start a port-forward to the ingress service by running:

   $ sudo kubectl port-forward service/okteto-ingress-nginx-controller 443:443 --namespace okteto

2. Access your Okteto instance at https://okteto.localtest.me/login#token=XXXXX

Follow these steps to configure the Okteto CLI:

1. Install the Okteto CLI:

   $ curl https://get.okteto.com -sSfL | sh

2. Configure the Okteto CLI context:

   $ okteto context use https://okteto.localtest.me --token XXXXXXXX --insecure-skip-tls-verify

3. You're all set to complete our getting started guide:

   https://www.okteto.com/docs/get-started/deploy-your-app/

Happy coding!
```

### Access your Instance
In order to access Okteto, you need to port-forward the `okteto-ingress-nginx-controller` service to your local machine using the command below:

```
sudo kubectl port-forward service/okteto-ingress-nginx-controller 443:443 --namespace okteto
```


### Sign in to your Okteto instance

After a successful installation, you can access your Okteto instance at https://okteto.localtest.me. Your account will be automatically created as part of the login process. The first user to successfully login into the instance will be automatically assigned the administrator role.

### Configure the Okteto CLI

Install the Okteto CLI if you haven't done it yet and set the Okteto CLI context with your Okteto instance. To do this, run the command below replacing TOKEN with the value generated during the helm installation:

okteto context use https://okteto.localtest.me --insecure-skip-tls-verify --token $TOKEN

Once your Okteto instance is up and running and your Okteto CLI properly configured, you are going to deploy your first app to Okteto 😎

#### Optional: Trust the self-signed certificate

The self-signed certificate is stored on the `default-ssl-certificate-selfsigned` secret in the `okteto` namespace.  Run the commands below to trust it:

On MacOS:

```
kubectl get secret -n=okteto default-ssl-certificate-selfsigned -o jsonpath="{.data['ca\.crt']}" | base64 -d > $TMPDIR/okteto-wildcard-ca.crt
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain $TMPDIR/okteto-wildcard-ca.crt
```
