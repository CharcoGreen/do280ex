# DO280 prepare exam

## Documentation

Documentación útil para el examen DO280
Aquí tienes algunos enlaces clave que puedes consultar durante el examen:

Documentación de OpenShift
[📌OpenShift Documentation](https://docs.openshift.com/container-platform/latest/)

Documentación de oc (CLI de OpenShift)
[📌OpenShift CLI Getting Started](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html)

Administración de OpenShift
[📌 OpenShift Cluster Tasks](https://docs.openshift.com/container-platform/latest/post_installation_configuration/cluster-tasks.html)

Gestión de usuarios y autenticación
[📌 OpenShift Authentication Documentation](https://docs.openshift.com/container-platform/latest/authentication/understanding-authentication.html)

Operaciones con oc adm
[📌 OpenShift Administrator CLI Commands](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/administrator-cli-commands.html)

Storage en OpenShift
[📌 OpenShift Storage Documentation](https://docs.openshift.com/container-platform/latest/storage/index.html)

Networking en OpenShift
[📌 OpenShift Networking Documentation](https://docs.openshift.com/container-platform/latest/networking/index.html)

Security y RBAC en OpenShift
[📌 OpenShift RBAC Documentation](https://docs.openshift.com/container-platform/latest/authentication/using-rbac.html)

## Some resources

[unofficial Wiki](https://www.enoks.fr/openshift/EX280/#manage-openshift-container-platform)  
[YouTube Videos 1](https://www.youtube.com/watch?v=YWMG6pZh8YY)  
[YouTube Videos 2](https://www.youtube.com/watch?v=nLhBrjhQRe8)  
[GitHub Repository: ex280v42](https://github.com/vbudi000/ex280v42)  
[GitHub Repository: EX280-study](https://github.com/bugbiteme/EX280-study)  

## Focus on

- Authentification
- Networkpolicies
- Helm
- Troubleshoot
- Users And groups
- Probes
- livenesprobe
- Autoscaling

## Example Questions

1. Create user mike with password blackcat  

    ```bash
    htpasswd -c -B -b /tmp/htpasswd mike blackcat
    ```

   Create user david with password openSky  

    ```bash
    ## Add users to file
    htpasswd -B -b /tmp/htpasswd david openSky
    ```

   Create user john with password dec2023  

   ```bash
   # Add users new to same file
   htpasswd -B -b /tmp/htpasswd john dec2023
   ```

    Identity provider name should be myidentity and secret name is mysecret  

    Create secret from htpasswd command.

    ```bash
    oc create secret generic mysecret \
    --from-file htpasswd=/tmp/htpasswd -n openshift-config
    ```

    > [!warning]  
    > Sure is that?

    Edit oauth cluster object

    ```bash
    apiVersion: config.openshift.io/v1
    kind: OAuth
        metadata:
        name: cluster
    spec:
        identityProviders:
        - name: my_htpasswd_provider
            mappingMethod: claim
            type: HTPasswd
            htpasswd:
            fileData:
                name: htpasswd-secret
    ```

2. Create project with name alpha, beta, gama  
   - mike user should have cluster administrator rights  
   - john user should have administrative access in project gama  
   - david should not have access to create project  
   - kubeadmin user should not exist  

    ```bash
    oc delete secret kubeadmin -n kube-system
    ```

3. Create a groups with named prod and dev  

   - First create group (out question)

   ```bash
    oc adm  group new prod
    oc adm group new dev
   ```

   - Add mike user in prod group  

    ```bash
    oc adm groups add-users prod mike
    ```

   add david and john users in dev  

    ```bash
    oc adm groups add-users dev john 
    oc adm groups add-users dev david
    ```

4. Give edit permission to dev groups on alpha project
   Give administrative permission to prod groups on beta project

5. Create resource quota by name myquota for project alpha
   Pods=10
   cpu=4
   services=6
   memory=1Gi
   secrets=5

    ```bash
    oc create quota myquota pods=10,cpus=4,memory=1G,services=6,secrets=5 -n alpha   
    ```

6. Create resource Limit by name mylimit for porject gama and definig resource limitrante as mentioned below:
   For pods min cpu limit is "3m" and max is 400m
   For containers min cpu limit is "10m" and max is "50m" and default request of "30m"
   For pods min pods min memory is "30Mi" and mas is "50Mi"
   For containers min memory is "20Mi" and max is "60Mi" and default request of "40Mi"

    ```yaml
    apiVersion: v1
    kind: limitRange
    metadata:  
        name: mylimit
        namespace: gama
    spec:
        limit:
        - type: Pod
          min:
            cpu: "3m"
            memory: "30Mi"
          max:
            cpu: "400m"
            memory: "50Mi"
        - type: Container
          min:
            cpu: "10m"
            memory: "20Mi"
          max:
            cpu: "50Mi"
            memory: "60Mi"
          defaultRequest:
            cpu: "30Mi"
            memory: "40Mi"
    ```

7. There is an application in project loan, application is running. Make you sure you are alble to access the application browser.  

8. Create Service Account by name ex280-sa in project project1, there is an application deployed in that project. The process inside

    make you sure you are application is able

    ```bash
    oc create serviceaccount ex280-sa
    ```

    - Add scc anyuid to sa

    ```bash
    oc adm policy add-scc-to-user anyuid -z ex280-sa
    ```

    - Mount sa in deploy

    ```bash
    oc set serviceaccount deployment.apps/name-deployment ex280-sa
    ```

    - Review if the application is successfull

9. In an insurance project you need to set up autoscaling for pods based on CPU utilization.  
    - The requerimente for autoscaling are as  follows:  
    - The minimum number of replicas should be 3  
    - The maximun number of replicas should be 9  
    - Autoscaling should be Triggered based on CPU utlilization reaching 70%  

    ```bash
    oc autoscale deployment.apps/namedeployment --min=3 --max=9 --cpu-percent=70
    ```

    ```bash
    oc get hpa
    ```

    - The containers in the pods have the following default CPU resource requests and limits  
    - Default CPU request for each container is 10m  
    - Default CPU limit for each  container is 100m  

    ```bash
    oc set resources deployment/deployment-name  --request cpu=10m --limit cpu=100m
    ```

10. Create secret with name circus in cloud project. The key name should be decode_value and the value of key should be babablachship=

   ```bash
   oc create secret generic circus --from-literal=decode_value="babablaship="
   ```

XX. Removing the default kubeadmin

```bash
oc delete secret kubeadmin -n kube-system
```

11. Create a secure route in quart project. There is already an executable file on server by name myCertificate.sh execute that script to generate new certificate. The certificate subject is -subj "/C=US/ST=Nort Carolina/L=Releigh/O=Red Hat\CN = todo-https.apps.ocp4.example.com"  you can either manually copy the values from subject or can pass the subject as an argument  
    - One application is already running. It should run on https with self-signed certificate
    - The application should produce the output

    After you create the certs

     ```bash
    oc create route edge todo-http  --servoce=todo-http --hostname=todo-https.apps.ocp4.example.com  --key devopstitan.key --cert devopstitan.crt 
    ```

14. Deploy application in the project rocket. There is one pod already running and application should produce output

- The deployment have a node selector with app=prod and the unique node have label app=dev, then we have than remove or overwrite.

    ```bash
    oc label nodes master03 app=prod --overwrite
    ```

  - The endpoint in service is missing. Edit service .

1.  An application is deployed  in the project modi. There is one pod already running and application should produce output

2.  An application is deployed in the project moon. There is one pod already running and, application should produce output. Don't make any changes in resources.

## Command Guide


Set data in secret auth-review from file.

```bash
oc set data secret auth-review --from-file htpasswd=~/DO280/labs/auth-review/tmp_users   -n openshift-config
oc adm policy add-cluster-role-to-group --rolebinding-name custom-self-provisioners self-provisioner managers
```

## Setting htpasswd file

```bash
# Create new file htpasswd 
htpasswd -c -B -b ~/DO280/labs/auth-review/tmp_users tester 'L@bR3v!ew'
# Validate Password to a specific user
htpasswd -v -b  ~/DO280/labs/auth-review/tmp_users tester 'L@bR3v!ew'
# Add new or update user and password to file.
htpasswd -b  ~/DO280/labs/auth-review/tmp_users tester 'L@bR3v!ew'
```

- Expose service (route) http

```bash
oc expose service/todo-http --hostname=todo-http.apps.ocp4.example.com
```

- Create edge route

```bash
oc create route edge todo-https \
    --service todo-http \
    --hostname todo-https.apps.ocp4.example.com
```

- Debug po
  
```bash
 oc debug -t deployment/todo-http \
    --image registry.ocp4.example.com:8443/ubi8/ubi:8.4
```

- Create Cert

```bash

## Run the following command to create the private key

openssl genrsa -out training.key 4096

## Run the following command to generate a certificate signing request

openssl req -new \
  -subj "/C=US/ST=North Carolina/L=Raleigh/O=Red Hat/CN=todo-https.apps.ocp4.example.com" \
  -key training.key -out training.csr

## Run the following command to generate a certificate

openssl x509 -req -in training.csr \
  -passin file:passphrase.txt \
  -CA training-CA.pem -CAkey training-CA.key -CAcreateserial \
  -out training.crt -days 1825 -sha256 -extfile training.ext

```

- Networkpolicy to ingress traffic from ingress namespace

```bash
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-from-openshift-ingress
spec:
  podSelector:
    matchLabels:
      deployment: 'hello'
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          policy-group.network.openshift.io/ingress: ""
```

- Internal traffic TLS
  
```bash
 oc annotate svc server service.beta.openshift.io/serving-cert-secret-name=server-secret
```

- Review cert.

```bash
openssl s_client -connect server.network-svccerts.svc:443
```

- Create SA

```bash
# Create empty configmap
oc crate configmap ca-bundle
# Annotate configmap
oc annotate configmap ca-bundle service.beta.openshift.io/inject-cabundle=true
```

## From O´reilly

https://github.com/sandervanvugt/ex280 

### Essential oc help

```bash
source <(oc completion)
oc set env -h 
oc set volumes -h 
oc set resources -h 
oc adm create-bootstrap-project-template
oc adm policy -h 
oc create quota -h
oc create secret tls -h 
oc describe clusterrole.rbac [|grep ^Name]
```


- bootstrap project with quota, limit range, networkpolicy

### Practice: Create project and secure app

- Create the project my-project
- In this project, run a deployment that is based on the Docker image sandervanvugt/openshift:latest with the name hello-app
- Configure this deployment such that is uses a TLS certificate at the expected location. Use the TLS certificate that was created in the prepartion steps of this lab.

```bash
oc new-project my-project
oc new-app --name hello-app --image=sandervanvugt/openshift:latest
oc create secret tls secure-app-tls --cert tls.crt --key tls.key
oc set volume deploy/hello-app --add --type secret --secret-name secure-app-tls --mount-path /run/secrets/nginx
```

### Practice: Configuring wordpress

- As developer user, use a deployment to create an application named wordpress in the microservice project
- Run this applicadtion with the anyuid security context assigned to the wordpress-sa service account
- Create a route to the wordpress application, using the hostname wordpress-microservice.apps-crc.testing
- Use secrets and or configmaps to set environment variables:
  - WORDPRESS_DB_HOST: is set to mysql
  - WORDPRESS_DB_NAME: is set to the value of wordpress
  - WORDPRESS_DB_USER: has the value "root"
  - WORDPRESS_DB_PASSWORD: is set to the value of the password key in the     mysql secret

```bash
# Login with developer user
oc login -u developer -p developer
# Create new project
oc new-project microservice
# Create new app wordpress
oc new-app --name wordpress --image wordpress
# Create sa to SCC
oc create sa wordpress-sa
# Login as admin
oc login -u admin -p password
# Create policy to scc to sa
oc adm policy add-scc-to-user  anyuid  -z wordpress-sa -n microservice
# Return login to user developer
oc login -u developer -p developer
# Set service Account deployment
oc set serviceacount deployment wordpress wordpress-sa
# Crate route from svc
oc expose svc wordpress
# Create cm wordpress with required values
oc create cm wordpress-cm --from-literal=host=mysql  --from-literal=name=wordpress --from-literal=user=root --from-literal=password=password
# Set env values to deployment.
oc set env deployment wordpress --prefix WORDPRESS_DB_ --from configmap/wordpress-cm
```

### Practice: Configuring MySQL

- As developer user, use a deployment to create an application named mysql in the mecroservice project
- Create a generic secret named mysql, using password as the key and mypassword as its value. Use this secret to set the MYSQL_ROOT_PASSWORD enviroment variable to the value of the password in the secret.
- Configure the MySQL application to mount a PVC to /mnt. The PVC must have a 1GiB, and the RWO access mode
- Use a node selector to ensure that MySQL will only run on your CRC node.

```bash
# Login as developer
oc login -u developer -p developer
# Change project if exist
oc project microservice
# Create new app mysql
oc new-app mysql --image=mysql
# Create generic Secret
oc create secret generic mysql --from-literal=password=mypassword
# Set secret in deployment like env
oc set env deploy/mysql --prefix MYSQL_ROOT_ --from secret/mysql
# Set PVC volume to deployment
oc set volumes deployment/mysql  --name mysql-pvc --add --type pvc --claim-size 1Gi --claim-mode rwo --mount-path /mnt
oc login -u admin -p password
oc get nodes --show-labes
# Edit to add nodeSelector value 
oc edit deployment mysql 
```

### Practice: Route passthrougt

- Use the linginx-v2.yaml file from the course Git repository to create an nginx application. Next, crate a passthrough route to access this application in a secure way. The name of the passthrough route must be set to linginx-default.apps-crc.testing
- The configuration file that is needed is provided as the file default.conf in the Git repository
- Use curl --insecure https://linginx-default.apps-crc.testing to verify that the route responds to external requests.

```bash
# First review the deployment file to inspect values 
# See the secret name and mount
# Create secret tls with current name
oc create sercret tls linginx-certs --cert lab6.crt --key lab6.key
# Review de configmap to nginx config and create configMap
oc create configmap nginxconfigmap --from-file=default.conf
# Also in the yaml file we have a service account so we need create
oc create sa linginx-sa
```

### Practice: Creating a Project template

- Create a new project template, according to the following requirements 
  - Each project has a label with the name of the project
  - Networkpolicy is set up such that:
    - Routes can be accessed by pods in namespaces with the network.openshift.io/policy-group=ingress label
    - Pods in the same namesapce can communicate with each other
    - Pods are only accesible to Pods in a different namespace if that namespace is configured with the nework.openshift.io/policy-group=ingress label
  - LimitRange is set up as follows:
    - Each container request 100 millicores of CPU
    - Each container request 50MiB of memory
    - Each container is limited to 200 millicores of CPU
    - Each container is limited to 100 MiB of memory
  - ResourceQuota is set up as follows:
    - Projects are limited to 20pods
    - Projects can request a maximun of 1GiB of memory
    - Projecs can request a maximum of 2CPUs
    - Projects can use a maximum of 2GiB of memory
    - Projects can use a maximum of 4 CPUs


```bash
oc new-project my-project 
oc describe project my-projects
```

```yaml
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "resource-limits"
spec:
  limits:
    - type: "Container"
      max:
        cpu: "200m"
        memory: "100Mi"
      defaultRequest:
        cpu: "100m"
        memory: "50Mi"
```