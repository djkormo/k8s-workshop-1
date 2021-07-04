
## Show list of cluster nodes

```bash
kubectl get nodes
```

Show all namespaces

```bash
kubectl get ns
```

List all of objects in default namespace

```
kubectl get all -o wide
```

Create namespace alpha if missing 

```
kubectl create ns alpha
```


**1.Create a pod named web using image nginx:1.11.9-alpine, on port 80 and 443.** 

```kubectl get pod -n alpha```

<pre>
NAME   READY   STATUS    RESTARTS   AGE
web    1/1     Running   0          49s
</pre>

CHECK

```
kubectl get pod web -n alpha |grep Running
```

```
kubectl get pod web -n alpha -o yaml  |grep 'image: nginx:1.11.9-alpine' && echo "done"

```
CHECK


<details>
<summary><b>Answer for Question 01 </b></summary>

Any folded content here. It requires an empty line just above it.

```bash
kubectl run web --image=nginx:1.11.9-alpine --port 80  -o yaml --dry-run=client > 01-web-pod.yaml
```
edit

```
vim 01-web-pod.yaml
```

add port 443

```bash
kubectl apply -f 01-web-pod.yaml -n alpha

kubectl expose pod/web --name=webservice -n alpha -o yaml --dry-run=client > 01-webservice-service.yaml
```


The complete yaml manifest

```yaml
cat <<EOF | kubectl apply -f -
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: web
  name: web
  namespace: alpha 
spec:
  containers:
  - image: nginx:1.11.9-alpine
    name: web
    ports:
    - containerPort: 80
    - containerPort: 443
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
EOF
```

</details>




**2.Create a service to expose that pod, named as webservice**

`kubectl get pod,svc,ep -n alpha`{{execute}}

<pre>

NAME      READY   STATUS    RESTARTS   AGE
pod/web   1/1     Running   0          51s

NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/webservice   ClusterIP   10.103.154.206   <none>        80/TCP,443/TCP   15s

NAME            ENDPOINTS                      AGE
endpoints/webservice   10.244.1.4:80,10.244.1.4:443   15s

</pre>

CHECK
`kubectl get svc webservice -n alpha |grep 80 && kubectl get svc webservice -n alpha |grep 443 &&  echo "done" `{{execute}}
CHECK

<details>
<summary><b>Answer for Question 02 </b></summary>




```bash
kubectl expose pod/web --name=webservice -n alpha -o yaml --dry-run=client > 01-webservice-service.yaml
```

The complete yaml manifest

```yaml
cat <<EOF | kubectl apply -f -
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: web
  name: webservice
  namespace: alpha
spec:
  ports:
  - name: port-1
    port: 80
    protocol: TCP
    targetPort: 80
  - name: port-2
    port: 443
    protocol: TCP
    targetPort: 443
  selector:
    run: web
status:
  loadBalancer: {}
EOF
```

</details>

**3.Create a pod named postgresql using image postgres:12.4 on port 5432.**

Something bad is going on

`kubectl get pod postgresql -n alpha`{{execute}}

`kubectl logs postgresql -n alpha`{{execute}}

<pre>
Error: Database is uninitialized and superuser password is not specified.
       You must specify POSTGRES_PASSWORD to a non-empty value for the
       superuser. For example, "-e POSTGRES_PASSWORD=password" on "docker run".

       You may also use "POSTGRES_HOST_AUTH_METHOD=trust" to allow all
       connections without a password. This is *not* recommended.

       See PostgreSQL documentation about "trust":
       https://www.postgresql.org/docs/current/auth-trust.html

</pre>


CHECK
`kubectl get pod postgresql -n alpha && echo "done"`{{execute}}

`kubectl get pod postgresql -n alpha -o yaml |grep " containerPort: 5432" && echo "done"`{{execute}}

`kubectl get pod postgresql -n alpha -o yaml  |grep 'image: postgres:12.4' && echo "done"`{{execute}}

CHECK

<details>
<summary><b>Answer for Question 03 </b></summary>

Any folded content here. It requires an empty line just above it.
TODO

</details>


Now we have to set database configuration values


**4.Create a pod named postgresql-env using image postgres:12.4 on port 5432.**

add to pod environment variables

POSTGRES_DB: postgresdb
POSTGRES_USER: postgresadmin
POSTGRES_PASSWORD: admin123

`kubectl get pod postgresql-env -n alpha`{{execute}}

`kubectl logs postgresql-env -n alpha`{{execute}}

<pre>
...
2020-09-24 22:10:12.464 UTC [1] LOG:  database system is ready to accept connections
...
</pre>


CHECK

`kubectl get pod postgresql-env -n alpha |grep Running && echo "done"`{{execute}}

`kubectl get pod postgresql-env -n alpha -o yaml |grep "containerPort: 5432"&& echo "done"`{{execute}}

`kubectl get pod postgresql-env -n alpha -o yaml  |grep 'image: postgres:12.4'&& echo "done"`{{execute}}

CHECK


<details>
<summary><b>Answer for Question 04 </b></summary>

Any folded content here. It requires an empty line just above it.
TODO

</details>


**5.Create a configmap named postgresql-configmap with following content**

POSTGRES_DB: postgresdb
POSTGRES_USER: postgresadmin
POSTGRES_PASSWORD: admin123


CHECK
`kubectl get cm postgresql-configmap -n alpha && echo "done"`{{execute}}

`kubectl get cm postgresql-configmap -n alpha -o yaml | grep "POSTGRES_DB: postgresdb" && echo "done"`{{execute}}

`kubectl get cm postgresql-configmap -n alpha -o yaml | grep "POSTGRES_USER: postgresadmin" && echo "done"`{{execute}}

`kubectl get cm postgresql-configmap -n alpha -o yaml | grep "POSTGRES_PASSWORD: admin123" && echo "done"`{{execute}}

CHECK


<details>
<summary><b>Answer for Question 04 </b></summary>

Any folded content here. It requires an empty line just above it.
TODO

</details>


**6.Create a pod named postgresql-cm using image postgres:12.4 on port 5432.**

But instead of environment variables use configmap postgresql-configmap


Hint!
use:

`kubectl explain pod.spec.containers.envFrom`{{execute}}

CHECK

`kubectl get pod postgresql-cm -n alpha | grep Running && echo "done"`{{execute}}

`kubectl get pod postgresql-cm -n alpha -o yaml |grep configMapRef -A1 | grep postgresql-cm && echo "done"`{{execute}}

CHECK

<details>
<summary><b>Answer for Question 06 </b></summary>

Any folded content here. It requires an empty line just above it.
TODO

</details>


**7.Create a configmap named postgresql-configmap-nopass with following content**

POSTGRES_DB: postgresdb
POSTGRES_USER: postgresadmin


CHECK

`kubectl get cm postgresql-configmap-nopass -n alpha && echo "done"`{{execute}}

`kubectl get cm postgresql-configmap-nopass -n alpha -o yaml | grep "POSTGRES_DB: postgresdb" && echo "done"`{{execute}}

`kubectl get cm postgresql-configmap-nopass -n alpha -o yaml | grep "POSTGRES_USER: postgresadmin" && echo "done"`{{execute}}

CHECK



<details>
<summary><b>Answer for Question 07 </b></summary>

Any folded content here. It requires an empty line just above it.
TODO

</details>



**8.Create a secret named postgresql-secret with following content**

POSTGRES_PASSWORD: admin123

CHECK

`kubectl get secret postgresql-secret -n alpha && echo "done"`{{execute}}

`kubectl get secret postgresql-secret -n alpha -o yaml | grep POSTGRES_PASSWORD: && echo "done"`{{execute}}

CHECK


<details>
<summary><b>Answer for Question 08 </b></summary>

Any folded content here. It requires an empty line just above it.
TODO

</details>



**9.Create a pod named postgresql-cm-secret using image postgres:12.4, on port 5432.**

use 
configmap postgresql-configmap-nopass
and
secret postgresql-secret

CHECK

`kubectl get pod postgresql-cm-secret -n alpha | grep Running && echo "done"`{{execute}}

`kubectl get pod postgresql-cm-secret -n alpha -o yaml |grep configMapRef -A1 | grep postgresql-configmap-nopass && echo "done"`{{execute}}

`kubectl get pod postgresql-cm-secret -n alpha -o yaml |grep secretRef: -A1| grep postgresql-secret && echo "done"`{{execute}}

CHECK


<details>
<summary><b>Answer for Question 09 </b></summary>

Any folded content here. It requires an empty line just above it.
TODO

</details>


**10.Create a service as ClusterIP to expose pod postgresql-cm-secret, named as postgresql-webservice**

CHECK

`kubectl get svc postgresql-webservice -n alpha |grep 5432 && echo "done" `{{execute}}

CHECK

`kubectl get pod,cm,secret,svc,ep -n alpha`{{execute}}
<pre>

NAME                       READY   STATUS             RESTARTS   AGE
pod/postgresql             0/1     CrashLoopBackOff   8          17m
pod/postgresql-cm          1/1     Running            0          5m6s
pod/postgresql-cm-secret   1/1     Running            0          67s
pod/postgresql-env         1/1     Running            0          13m
pod/web                    1/1     Running            0          27m

NAME                                    DATA   AGE
configmap/postgresql-configmap          3      11m
configmap/postgresql-configmap-nopass   2      4m24s

NAME                         TYPE                                  DATA   AGE
secret/default-token-hbtjl   kubernetes.io/service-account-token   3      29m
secret/postgresql-secret     Opaque                                1      3m46s

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/postgresql-webservice   ClusterIP   10.103.18.155   <none>        5432/TCP         4s
service/webservice             ClusterIP   10.110.75.85    <none>        80/TCP,443/TCP   22m

NAME                             ENDPOINTS                                          AGE
endpoints/postgresql-webservice   10.244.1.10:5432,10.244.1.6:5432,10.244.1.8:5432   4s
endpoints/webservice             10.244.1.3:80,10.244.1.3:443                       22m
</pre>


<details>
<summary><b>Answer for Question 10 </b></summary>

Any folded content here. It requires an empty line just above it.
TODO

</details>

