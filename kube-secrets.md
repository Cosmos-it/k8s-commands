# Using secrets in kubernetes

A secret represents an object that contains sentive data.


## Creating secrets

Pods can be used with pods in two ways

- Files in a volume mounted on one or more containers
- Used by kubletes when pulling for the pod.


Say that some of your pods needs to access database. The database needs username and password. The files are: ./username./txt and ./password.txt on your machine

- `echo -n 'admin' > ./username.txt`
- `echo -n 'sdfgsgf'  > ./password.txt`
 

Use kubectl create secret command: This packages the files ihto a secrete and creates the object on the Apiserver

Results in: secret 'db-user-pass" created

Note: Special characters such as $, \, * and ! will need to be escaped

- `kubectl create secret generic dev-db-secret --from-literal=username=devuser --from-literal=password='S!B\*d$zDsb'`

Then check your secrets

- `kubectl get secrets`
- `kubectl describe secrets/db-user-pass`


## Creating Secret Manually

You can create file in a json or yaml format and then create the object.
The file contains two maps.
- Data and StringData

For instance, we can store two strings in a scret using data field and convert them as follows.

```
echo -n 'admin' | base64
YWRtaW4=`
echo -n '1f2d1e2e67df' | base64
MWYyZDFlMmU2N2Rm
```

## Write a file that looks like this and give it a name: secret.yaml

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

Now apply the screte
- `kubectl apply -f ./secret.yaml`


When deploying an actual application which uses a secret, you will need to set a configuration file that can be used during deployment process.

Something like this.

```
apiUrl: "https://my.api.com/api/v1"
username: "user"
password: "password"
```

To this:

```sh
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
stringData:
  config.yaml: |-
    apiUrl: "https://my.api.com/api/v1"
    username: {{username}}
    password: {{password}}
```

The deployment tool could then replace the username and password template varible befoer running kubectl apply .
stringData is a write-only convenience field.

kubectl get secret mysecret -o yaml


## Creating a Secret from Generator


### Create a kustomization.yaml file with SecretGenerator

```
cat <<EOF >./kustomization.yaml
secretGenerator:
- name: db-user-pass
  files:
  - username.txt
  - password.txt
EOF
```

`kubectl apply -k .`


Then check the status

`kubectl get secrets`

For example, to generate a Secret from literals username=admin and password=secret, you can specify the secret generator in kustomization.yaml as


### Create a kustomization.yaml file with SecretGenerator


```
cat <<EOF >./kustomization.yaml
secretGenerator:
- name: db-user-pass
  literals:
  - username=admin
  - password=secret
EOF
```

## Decoding Secrets

`kubectl get secret`


kubectl get secret mysecret -o yaml


`echo 'MWYyZDFlMmU2N2Rm' | base64 --decode`

## Editing a Secret

`kubectl edit secrets mysecret`


## Using Secrets

Secrets can be used a data volumes or be exposed as environment variables to be used by a container in a pod.
They can also be used by other parts of the system, without being directly exposed to the pod. 

Eg. Hold credentials that other parts of the system should use to interact with external systems on your behalf.


### Using secrets as Files from a pod.

To consume a secret in a volume in a pod:

- Create a secret or use existing one. Multiple pods can reference the same pod.
- Modify your pod definition to add a volume under .spec.volumes[] Name the volume anything, and have a .spec.volumes.secret.secretName field equals to the name of the secret object.
- Add a .spec.containers[].volumeMounts[] to each container that needs the secret. Specify .spec.containers[].volumeMounts[].readOnly = true and .spec.containers[].volumeMounts[].mountPath to an unused directory name where you would like the secrets to appear.

- Modify your image and/or command line so that the program looks for files in that directory. Each key in the secret data map becomes the filename under mountPath

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```
Each secret you want to use needs to be referred to in .spec.volumes.

If there are multiple containers in the pod, then each container needs its own volumeMounts block, but only one .spec.volumes is needed per secret

## Using Secrets as Environment Variables



