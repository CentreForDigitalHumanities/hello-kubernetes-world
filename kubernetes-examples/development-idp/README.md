# Development IdP on Kubernetes example

This example shows how to deploy the [development IdP](https://github.com/CentreForDigitalHumanities/Development-IdP) on Kubernetes.

## Caveats

The development IdP is a very simple app that is specifically designed to be run as a single container (with an embedded database or external database). 
A 'real' django app will most likely require a more complex setup to handle various duties.

## Variants

It comes in two forms:
- Plain manifests (used for the presentation)
- Using kustomize

The kustomize variant should be considered _advanced_ and is only included for those who are interested in diving deeper into Kubernetes.

### Plain manifests

The plain manifests are a 'stripped down' version, with the minimum required to get the app running. It deploys the app with an embedded SQLite database.

To get started:
1. Create `secrets.secret.yaml` using the example file. For testing, you can use the example values.
2. Create `certs.secret.yaml` using the example file. For testing, you can use the example values.
3. Update `ingress.yaml` with the domain of your choice
4. Update `config-map.yaml` as instructed in the file
5. Deploy the app using `kubectl apply -f . -n <your namespace>`

#### Files

If you're familiar with docker-compose, you can compare the following files to the [docker-compose](https://github.com/CentreForDigitalHumanities/Development-IdP/blob/main/docker-compose.yaml) file of the development IdP:

- `deployment.yaml`: the main configuration of the 'idp' pod. Most of the settings in this file are similar to `services` in a docker-compose file.
- `service.yaml`: the service that exposes the 'idp' pod to the cluster. It can be (very) roughly compared to the `ports` setting in a docker-compose file.
- `ingress.yaml`: the ingress that exposes the 'idp' service to the outside world. This has no equivalent in docker-compose.
- `config-map.yaml`: the configuration that the 'idp' pod needs to run. This file functions similarly to an env file in the `env_file` setting in a docker-compose file.
- `secrets.secret.yaml`: the secrets that the 'idp' pod needs to run. In the docker-compose file, these settings are part of env file in the `env_file` setting.
- `certs.secret.yaml`: the certificates that the 'idp' pod will need. In the docker-compose file, these are mounted as a host-dir volume. 

The files themselves also have extensive comments to help you understand what each part does.

### Using kustomize

The first variant showed how to deploy a single container app using plain manifests. However, in the real world, we often want to deploy multiple different versions of the same app, or deploy the same app to different environments. 
For example, you might want to deploy a 'production', an 'acceptation' and a 'development' version of the same app. Using the above method, you'll have to copy the manifests and change them for each version. This is error-prone and hard to maintain.

What we want, is a way to create a 'base' configuration, and then 'patch' it with different configurations to create different variants of the same app.

This is where kustomize comes in. While there are other tools to do this, kustomize is the simplest tool to get started with. (Also, it's built into kubectl, so you don't need to install anything extra!)

Note: the kustomize variant is more complex than the plain manifests, and is only included for those who are interested in diving deeper into Kubernetes. It includes running a separate database, and some other extras not present in the plain manifests example.

#### Structure

The kustomize variant is structured as follows:
- `base`: the base configuration of the app. This is further split out into two parts:
  - `django`: the main configuration of the django app. These files are broadly similar to the files found in the plain manifests example. (In fact, the plain manifests example was created from these files!)
  - `psql`: the postgres database configuration. This is a new part that was not present in the plain manifests example.
- `overlay`: this folder contains the 'variants' of the app. In this example, there are two variants:
  - `test`: this variant deploys the app using settings more suited to a test environment. 
  - `production`: this variant deploys the app using settings more suited to a production environment.

Inside the `base/django` and `base/psql` folders, you'll find a file called `kustomization.yaml`. In this example, these files only list the manifest-files that need to be included in the final configuration.
In more advanced setups, you might also want to add more kustomize-configuration to these files.

Inside the `overlay/test` and `overlay/production` folders, you'll also find a file called `kustomization.yaml`. This is the actual file that will be used to generate our final configuration. 
In this file, you can see that it includes the base configuration, and then 'patches' it with the settings specific to the test or production environment.

Please read the comments in the `kustomization.yaml` files to get a better understanding of what each part does.

Side-note: the source manifests are not as extensively commented as the plain manifests example. This is because the author ran out of time. If you're interested in a more detailed explanation, please let me know!

#### Usage

To deploy the app using kustomize, you just use this simple command:
```
kubectl apply -k overlay/test -n <your namespace>
```