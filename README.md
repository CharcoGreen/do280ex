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
