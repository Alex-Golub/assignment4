<h1 id="home">K8S Assignment (lab4)</h1>

## Prerequisites

- `kubectl`
    - Client Version >= v1.25.2
    - Server Version >= v1.24.3
- `minikube` version >= v1.26.1
- Save valuable time and run `alias k=kubectl`
- [Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)

## Solutions

[Part 1](#1)  
[Pod Design Questions](#2)  
[Deployments](#3)  
[Config Map](#4)

<h3 id="1">Part 1</h3>

1. Deploy a pod named `nginx-pod` using the `nginx:alpine` image

   ```commandline
   k run nginx-pod --image=nginx:latest
   ```

<div align="right"><a href="#home">Back to top 👆</a></div>

2. Deploy a messaging pod using the `redis:alpine` image with the labels set to `tier=msg`

   ```commandline
   k run messaging \
   --image=redis:alpine \
   --labels=tier=msg
   
   # To follow pod status 
   k get pods --show-labels=true --watch=true
   
   NAME        READY   STATUS              RESTARTS   AGE   LABELS
   messaging   0/1     ContainerCreating   0          1s    tier=msg
   messaging   1/1     Running             0          3s    tier=msg
   ```

<div align="right"><a href="#home">Back to top 👆</a></div>

3. Create a `namespace` named `apx-x998-{{your-name}}`

   ```commandline
   k create namespace apx-x998-alex
   
   # Verify namespace was created with the supplied label
   k get namespace --show-labels=true
   
   NAME              STATUS   AGE   LABELS
   apx-x998-alex     Active   10s   kubernetes.io/metadata.name=apx-x998-alex
   ```

<div align="right"><a href="#home">Back to top 👆</a></div>

4. Get the list of `nodes` in `JSON` format and store it in a file at `/tmp/nodes-{{your-name}}`

   ```commandline
   k get nodes --output=json > /tmp/nodes-alex.json
   ```

   NOTE: On Windows, output file is located in `C:\Users\%USERNAME%\AppData\Local\Temp`

<div align="right"><a href="#home">Back to top 👆</a></div>

5. Create a `service` named `messaging-service` to expose the messaging application within the cluster on port 6379 (imperative)

   ```commandline
   k expose pod messaging \
   --name=messaging-service \
   --port=6379 \
   --type=ClusterIP
   ```

   NOTE: By default services type are set to `ClusterIP`

<div align="right"><a href="#home">Back to top 👆</a></div>

6. Create a `service` named `messaging-service.yaml` to expose the messaging application within the cluster on port 6379 (declarative)

   ```yaml
   apiVersion: v1 
   kind: Service
   metadata:
      name: messaging-service
      labels:
        tier: msg
   spec:
     type: ClusterIP
     selector:
        tier: msg
     ports:
       - port: 6379
         protocol: TCP
         targetPort: 6379 
    ```

   Now we can create the service executing

   ```commandline
   k create --filename=messaging-service.yaml
   ```

<div align="right"><a href="#home">Back to top 👆</a></div>

7. Create a `deployment` named `hr-web-app` using the image `kodekloud/webapp-color` with 2 replicas

   ```commandline
   k create deployment hr-web-app \
   --image=kodekloud/webapp-color \
   --replicas=2
   ```

<div align="right"><a href="#home">Back to top 👆</a></div>

8. Create a static pod named `static-busybox` on the master node that uses the `busybox` image and the command `sleep 1000`
    - [Ref Docs: static-pod-creation](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/#static-pod-creation)
    - [Ref Docs: how-does-cat-eof-work-in-bash](https://stackoverflow.com/questions/2500436/how-does-cat-eof-work-in-bash)

    ```commandline
    # Generate pod template with required parameters
    k run static-busybox \
    --image=busybox \
    --dry-run=client \
    --output=yaml \
    --command -- sleep 1000 > static-busybox.yaml
    
    # Login into master node
    ssh minikube
    
    # Static pods controlled by kubelet (should be configured for it) under /etc/kubernetes/manifests directory
    cat <<EOF > /etc/kubernetes/manifests/static-busybox.yaml
    apiVersion: v1
    kind: Pod
    metadata:
    labels:
      run: static-busybox
    name: static-busybox
    spec:
    containers:
      - command:
          - sleep
          - "1000"
        image: busybox
        name: static-busybox
        imagePullPolicy: IfNotPresent
        resources: { }
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    EOF
    
    # Back to master
    logout
    ```

   Another solution without login into node

    ```commandline
    k run static-busybox \
    --image=busybox \
    --dry-run=client \
    --output=yaml \
    --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

9. Create a pod in the `finance-yourname` namespace named `temp-bus` with the image `redis:alpine`

   ```commandline
   k create namespace finance-alex
   
   k run temp-bus \
   --image=redis:alpine \
   --namespace=finance-alex
   
   k get pods --namespace=finance-alex
   ```

<div align="right"><a href="#home">Back to top 👆</a></div>

10. Create a Persistent Volume with the given specification
    - Volume Name: pv-analytics
    - Storage: 100Mi
    - Access modes: ReadWriteMany
    - Host Path: /pv/data-analytics
    - [Ref Docs: volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
    - [Ref Docs: types-of-persistent-volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes)

    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv-analytics
    spec:
      capacity:
        storage: 100Mi
      accessModes:
        - ReadWriteMany
      hostPath:
        path: /pv/data-analytics
    ```

    ```commandline
    k create --filename=pv-analytics.yaml
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

11. Create a Pod called `redis-storage-{{yourname}}` with image: `redis:alpine` with a Volume of type `emptyDir` that lasts for the life of the Pod
    - Pod named `redis-storage-{{yourname}}`
    - Pod `redis-storage-{{yourname}}` uses Volume type of `emptyDir`
    - Pod `redis-storage-{{yourname}}` uses `volumeMounts` with `mountPath = /data/redis`
    - [Ref Docs: configure-a-volume-for-a-pod](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/#configure-a-volume-for-a-pod)

    ```commandline
    k run redis-storage-alex \
    --image=redis:alpine \
    --dry-run=client \
    --output=yaml > redis-storage-alex.yaml
    ```

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        run: redis-storage-alex
      name: redis-storage-alex
    spec:
      containers:
        - image: redis:alpine
          name: redis-storage-alex
          volumeMounts:
            - mountPath: /data/redis
              name: redis-volume
      volumes:
        - name: redis-volume
          emptyDir: { }
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    ```

    ```commandline
    k apply --filename=redis-storage-alex.yaml
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

12. Create this pod and attached it a persistent volume called `pv-1`
    - [Ref Docs: persistentvolumeclaims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)

    ```commandline
    k run pv-alex \
    --image=nginx \
    --dry-run=client \
    --output=yaml > pv-alex.yaml
    ```

    Modify Pod template to use persistent volume claim

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        run: pv-alex
      name: pv-alex
    spec:
      volumes:
        - name: pv-1
          persistentVolumeClaim:
            claimName: pv-claim
      containers:
        - image: nginx
          name: pv-alex
          volumeMounts:
            - mountPath: /data
              name: pvc-alex
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    ```

    Create `PersistentVolumeClaim` resource

    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: pvc-alex
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 100Mi
    ```

    Apply new configuration to pod and run it

    ```commandline
    k apply --filename=pv-alex.yaml
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

13. Create a new `deployment` called `nginx-deploy`, with image `nginx:1.16` and 1 replica.  
    Record the version.  
    Next upgrade the deployment to version 1.17 using rolling update.  
    Make sure that the version upgrade is recorded in the resource annotation.

    ```commandline
    k create deployment nginx-deploy \
    --image=nginx:1.16 \
    --replicas=1 \
    --dry-run=client \
    --output=yaml > nginx-deploy.yaml
    
    k apply --filename=nginx-deploy.yaml --record
    
    k rollout history deployment
    REVISION  CHANGE-CAUSE
    1         kubectl.exe create --filename=nginx-deploy.yaml --record=true
    
    k set image deployment/nginx-deploy nginx=nginx:1.17 --record
    
    k rollout history deployment
    REVISION  CHANGE-CAUSE
    1         kubectl.exe create --filename=nginx-deploy.yaml --record=true
    2         kubectl.exe set image deployment/nginx-deploy nginx=nginx:1.17 --record=true
    
    k get deploy nginx-deploy --output=yaml
    ```

    ```yaml
    apiVersion: v1
    items:
      - apiVersion: apps/v1
        kind: Deployment
        metadata:
          annotations:
            deployment.kubernetes.io/revision: "2"
            kubernetes.io/change-cause: kubectl.exe set image deployment/nginx-deploy nginx=nginx:1.17
              --record=true
    ...
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

14. Create an nginx pod called `nginx-resolver` using image `nginx`, expose it internally with a
    service called `nginx-resolver-service`.  
    Test that you are able to look up the service and pod names from within the cluster.  
    Use the image: `busybox:1.28` for dns lookup.  
    Record results in `/root/nginx-yourname.svc` and `/root/nginx-yourname.pod`
    - [Ref Docs: jsonpath](https://kubernetes.io/docs/reference/kubectl/jsonpath/)

    ```commandline
    k run nginx-resolver \
    --image=nginx \
    --labels=app=nginx-resolver
    ```

    Create required service

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-resolver-service
    spec:
      selector:
        app: nginx-resolver
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
    ```

    ```commandline
    k apply --filename=nginx-resolver-service.yaml
    
    k run dns-lookup --image=busybox:1.28 -- sleep 2000
    CLUSTER_IP=$(k get svc nginx-resolver-service --output=jsonpath='{.spec.clusterIP}')
    k exec -it dns-lookup -- nslookup $CLUSTER_IP > /root/nginx-alex.svc
    
    POD_IP=$(k get pods nginx-resolver -o=jsonpath='{.status.podIP}')
    k exec -it dns-lookup -- nslookup $POD_IP > /root/nginx-alex.pod
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

15. Create a static pod on `node01` called `nginx-critical` with image `nginx`.  
    Create this pod on `node01` and make sure that it is recreated/restarted automatically in case of a failure.

    ```commandline
    # Generate pod template with required image 
    k run nginx-critical --image=nginx --dry-run=client --output=yaml > nginx-critical.yaml
    
    # Connect to node
    ssh node01
    
    # Static pods controlled by kubelet (should be configured for it) under /etc/kubernetes/manifests directory 
    cat <<EOF > /etc/kubernetes/manifests/nginx-critical.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        run: nginx-critical
      name: nginx-critical
    spec:
      containers:
      - image: nginx
        name: nginx-critical
        resources: {}
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    status: {}
    EOF
    
    # Back to master
    logout
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

16. Create a pod called `multi-pod` with two containers.  
    Container 1, name: alpha, image: nginx  
    Container 2: beta, image: busybox, command sleep 4800.  
    a. Environment Variables:
    i. container 1:
    ii. name: alpha
    iii. Container 2:
    iv. name: beta

    ```commandline
    # Generate pod with single container
    k run alpha --image=nginx --dry-run=client --output=yaml > multi-pod.yaml
    ```

    Edit `multi-pod.yaml`

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: multi-pod
    spec:
      containers:
        - image: nginx
          name: alpha
          imagePullPolicy: IfNotPresent
          env:
            - name: alpha
        - image: busybox
          name: beta
          imagePullPolicy: IfNotPresent
          env:
            - name: beta
          command: [ "sleep", "4800" ]
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    
    ```

    ```commandline
    k apply --filename=multi-pod.yaml
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

---

<h2 id="2">Pod Design Questions</h3>

1. Type the command for: Get pods with label information

    ```commandline
    k get pods --show-labels=true
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

2. Create 5 `nginx` pods in which two of them is labeled `env=prod` and three of them is labeled `env=dev`

    ```commandline
    k run nginx-dev01 --image=nginx --labels=env=dev
    k run nginx-dev02 --image=nginx --labels=env=dev
    k run nginx-dev03 --image=nginx --labels=env=dev
   
    k run nginx-prod04 --image=nginx --labels=env=prod
    k run nginx-prod05 --image=nginx --labels=env=prod
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

3. Verify all the pods are created with correct labels

    ```commandline
    k get pods --show-labels=true
    
    NAME           READY   STATUS    RESTARTS   AGE   LABELS  
    nginx-dev01    1/1     Running   0          38s   env=dev 
    nginx-dev02    1/1     Running   0          38s   env=dev 
    nginx-dev03    1/1     Running   0          37s   env=dev 
    nginx-prod04   1/1     Running   0          37s   env=prod
    nginx-prod05   1/1     Running   0          36s   env=prod
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

4. Get the pods with label `env=dev`

    ```commandline
    k get pods --selector=env=dev
    
    NAME          READY   STATUS    RESTARTS   AGE
    nginx-dev01   1/1     Running   0          5m5s
    nginx-dev02   1/1     Running   0          5m5s
    nginx-dev03   1/1     Running   0          5m4s
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

5. Get the pods with label `env=dev` and also output the labels

    ```commandline
    k get pods --selector=env=dev --show-labels=true
    
    NAME          READY   STATUS    RESTARTS   AGE     LABELS
    nginx-dev01   1/1     Running   0          6m56s   env=dev
    nginx-dev02   1/1     Running   0          6m56s   env=dev
    nginx-dev03   1/1     Running   0          6m55s   env=dev
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

6. Get the pods with label `env=prod`

    ```commandline
    k get pods --selector=env=prod
    
    NAME           READY   STATUS    RESTARTS   AGE
    nginx-prod04   1/1     Running   0          7m41s
    nginx-prod05   1/1     Running   0          7m40s
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

7. Get the pods with label `env=prod` and also output the labels

    ```commandline
    k get pods --selector=env=prod --show-labels=true
    
    NAME           READY   STATUS    RESTARTS   AGE     LABELS
    nginx-prod04   1/1     Running   0          8m21s   env=prod
    nginx-prod05   1/1     Running   0          8m20s   env=prod
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

8. Get the pods with label `env`

    ```commandline
    k get pods --label-columns=env
    
    NAME           READY   STATUS    RESTARTS   AGE   ENV
    nginx-dev01    1/1     Running   0          11m   dev
    nginx-dev02    1/1     Running   0          11m   dev
    nginx-dev03    1/1     Running   0          11m   dev
    nginx-prod04   1/1     Running   0          11m   prod
    nginx-prod05   1/1     Running   0          11m   prod
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

9. Get the pods with labels `env=dev` and `env=prod`

    ```commandline
    k get pods --selector='env in (dev,prod)'
    
    NAME           READY   STATUS    RESTARTS   AGE
    nginx-dev01    1/1     Running   0          16m
    nginx-dev02    1/1     Running   0          16m
    nginx-dev03    1/1     Running   0          16m
    nginx-prod04   1/1     Running   0          16m
    nginx-prod05   1/1     Running   0          16m
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

10. Get the pods with labels `env=dev` and `env=prod` and output the labels as well

    ```commandline
    k get pods --selector='env in (dev,prod)' --show-labels
    
    NAME           READY   STATUS    RESTARTS   AGE   LABELS
    nginx-dev01    1/1     Running   0          17m   env=dev
    nginx-dev02    1/1     Running   0          17m   env=dev
    nginx-dev03    1/1     Running   0          17m   env=dev
    nginx-prod04   1/1     Running   0          17m   env=prod
    nginx-prod05   1/1     Running   0          17m   env=prod
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

11. Change the label for one of the pod to `env=uat` and list all the pods to verify

    ```commandline
    k label pod nginx-dev01 env=uat --overwrite
    
    k get pods --show-labels
    NAME           READY   STATUS    RESTARTS   AGE   LABELS
    nginx-dev01    1/1     Running   0          19m   env=uat
    nginx-dev02    1/1     Running   0          19m   env=dev
    nginx-dev03    1/1     Running   0          19m   env=dev
    nginx-prod04   1/1     Running   0          19m   env=prod
    nginx-prod05   1/1     Running   0          19m   env=prod
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

12. Remove the labels for the pods that we created now and verify all the labels are removed

    ```commandline
    k label pod nginx-dev{01..03} env-
    k label pod nginx-prod{04..05} env-
    
    k get pods --show-labels
    NAME           READY   STATUS    RESTARTS   AGE   LABELS
    nginx-dev01    1/1     Running   0          23m   <none>
    nginx-dev02    1/1     Running   0          23m   <none>
    nginx-dev03    1/1     Running   0          23m   <none>
    nginx-prod04   1/1     Running   0          23m   <none>
    nginx-prod05   1/1     Running   0          23m   <none>
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

13. Let’s add the label `app=nginx` for all the pods and verify (using kubectl)

    ```commandline
    k label pod nginx-dev{01..03} app=nginx
    k label pod nginx-prod{04..05} app=nginx
    
    k get pods --show-labels
    NAME           READY   STATUS    RESTARTS   AGE   LABELS
    nginx-dev01    1/1     Running   0          25m   app=nginx
    nginx-dev02    1/1     Running   0          25m   app=nginx
    nginx-dev03    1/1     Running   0          25m   app=nginx
    nginx-prod04   1/1     Running   0          25m   app=nginx
    nginx-prod05   1/1     Running   0          25m   app=nginx
    
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

14. Get all the nodes with labels (if using minikube you would get only master node)

    ```commandline
    k get nodes --show-labels
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

15. Label the worker node `nodeName=nginxnode`

    ```commandline
    k label node minikube nodeName=nginxnode
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

16. Create a Pod that will be deployed on the worker node with the label `nodeName=nginxnode`

    ```commandline
    k run nginx \
    --image=nginx \
    --dry-run=client \
    --output=yaml > nginx.yaml
    ```

    Create label `nginxnode` is assigned to this pod

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        run: nginx
      name: nginx
    spec:
      nodeSelector:
        nodeName: nginxnode
      containers:
        - image: nginx
          name: nginx
          resources: { }
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    ```

    ```commandline
    k apply --filename=nginx.yaml
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

17. Verify the pod that it is scheduled with the node selector on the right node… fix it if it’s not behind scheduled.

    ```commandline
    k describe pod nginx | grep Node-Selector
    
    Node-Selectors:              nodeName=nginxnode
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

18. Verify the pod nginx that we just created has this label

    ```commandline
    k describe pod nginx | grep Labels
    Labels:           run=nginx
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

---

<h2 id="3">Deployments</h3>

1. Create a deployment called `webapp` with image `nginx` with 5 replicas

    ```commandline
    k create deployment webapp \
    --image=nginx \
    --dry-run=client \
    --output=yaml > webapp.yaml
    ```

   Modifying `webapp.yaml` to use 5 replicas

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: webapp
      name: webapp
    spec:
      replicas: 5 # change from 1 to 5
      selector:
        matchLabels:
          app: webapp
      strategy: { }
      template:
        metadata:
          labels:
            app: webapp
        spec:
          containers:
            - image: nginx
              name: nginx
              resources: { }
    ```

   Create deployment

    ```commandline
    k create --filename=webapp.yaml
    ```

   NOTE: can be done in single cli command

    ```commandline
    k create deployment webapp \
    --image=nginx \
    --replicas=5 \
    --output=yaml > webapp.yaml
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

2. Get the deployment `rollout` status

    ```commandline
    k rollout status deploy webapp
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

3. Get the replicaset that created with this deployment

    ```commandline
    k get replicasets --output=wide
    
    NAME                DESIRED   CURRENT   READY   AGE    CONTAINERS   IMAGES   SELECTOR
    webapp-678fd589dc   5         5         5       3m4s   nginx        nginx    app=webapp,pod-template-hash=678fd589dc 
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

4. EXPORT the yaml of the replicaset and pods of this deployment

    ```commandline
    k get replicasets --selector=app=webapp --output=yaml
    k get pods --selector=app=webapp --output=yaml
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

5. Delete the deployment you just created and watch all the pods are also being deleted

    ```commandline
    k delete deploy webapp
    k get pods --selector=app=webapp --watch=true
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

6. Create a deployment of `webapp` with image `nginx:1.17.1` with container port 80 and verify the image version

    ```commandline
    k create deploy webapp \
    --image=nginx:1.17.1 \
    --dry-run=client \
    --output=yaml > webapp.yaml
    ```

   Add `ports: list` property to nginx container

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: webapp
      name: webapp
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: webapp
      strategy: { }
      template:
        metadata:
          labels:
            app: webapp
        spec:
          containers:
            - image: nginx:1.17.1
              name: nginx
              ports:
                - containerPort: 80
              resources: { }
    ```

   Create deployment resource from `webapp.yaml`

    ```commandline
    k create --filename=webapp.yaml
    ```

   Describe deployment to verify the correct image is used

    ```commandline
    k describe deploy webapp | grep Image
    
        Image:        nginx:1.17.1
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

7. Update the deployment with the image version `1.17.4` and verify

    ```commandline
    k set image deploy webapp nginx=nginx:1.17.4
    k describe deploy webapp | grep Image
        Image:        nginx:1.17.4
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

8. Check the `rollout` history and make sure everything is ok after the update

    ```commandline
    k rollout history deploy webapp
    
    k get all --selector=app=webapp
    
    NAME                          READY   STATUS    RESTARTS   AGE
    pod/webapp-5657d889bc-mbkg4   1/1     Running   0          2m14s
    
    NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/webapp   1/1     1            1           9m5s
    
    NAME                                DESIRED   CURRENT   READY   AGE
    replicaset.apps/webapp-5657d889bc   1         1         1       2m14s
    replicaset.apps/webapp-7655c974dd   0         0         0       9m5s
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

9. Undo the deployment to the previous version 1.17.1 and verify Image has the previous version

    ```commandline
    k rollout undo deploy webapp
    k describe deploy webapp | grep Image
        Image:        nginx:1.17.1
    
    k rollout history deploy webapp
    
    deployment.apps/webapp 
    REVISION  CHANGE-CAUSE
    2         <none>
    3         <none>
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

10. Update the deployment with the wrong image version `1.100` and-verify something is wrong with the deployment

    ```commandline
    k set image deploy webapp nginx=nginx:1.100
    
    k rollout status deploy webapp
    Waiting for deployment "webapp" rollout to finish: 1 old replicas are pending termination...
    
    k get pods --selector=app=webapp
    NAME                      READY   STATUS             RESTARTS   AGE
    webapp-7655c974dd-tffvm   1/1     Running            0          2m12s
    webapp-7f565f774d-8cjxq   0/1     ErrImagePull       0          24s
    
    # Checking Events we can see that 'Failed to pull image "nginx:1.100":'
    k describe pod webapp-7655c974dd-tffvm
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

11. Apply the autoscaling to this deployment with minimum 10 and maximum 20 replicas and target CPU of 85% and verify hpa is created and replicas are increased to 10 from 1

    ```commandline
    k autoscale deploy webapp --min=10 --max=20 --cpu-percent=85
    k get hpa
    k get pod --selector=app=webapp
    k get deploy webapp --output=yaml > webapp-check.yaml
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

13. Clean the cluster by deleting deployment and hpa you just created

    ```commandline
    k delete all --selector=app=webapp
    k delete hpa webapp
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

14. Create a job and make it run 10 times one after one

    ```commandline
    k create job hello-job \
    --image=busybox \
    --dry-run=client \
    --output=yaml -- echo "Hello I am from job" > hello-job.yaml
    ```

    Edit job resource

    ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: hello-job
    spec:
      completions: 10
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
            - command:
                - echo
                - Hello I am from job
              image: busybox
              name: hello-job
              resources: { }
          restartPolicy: Never
    ```

    Create job, watch pod creation and finally terminate the job

    ```commandline
    k create --filename=hello-job.yaml
    k get job --watch=true                            
    
    # NAME        COMPLETIONS   DURATION   AGE
    # hello-job   0/10          1s         1s 
    # hello-job   1/10          8s         8s
    # hello-job   2/10          15s        15s
    # hello-job   3/10          23s        23s
    # hello-job   4/10          31s        31s
    # hello-job   5/10          39s        39s
    # hello-job   6/10          48s        48s
    # hello-job   7/10          56s        56s
    # hello-job   8/10          64s        64s
    # hello-job   9/10          72s        72s
    # hello-job   10/10         80s        80s
    
    k get pods
    # NAME              READY   STATUS      RESTARTS   AGE 
    # hello-job-dgn6r   0/1     Completed   0          68s 
    # hello-job-gc499   0/1     Completed   0          85s 
    # hello-job-gvrlb   0/1     Completed   0          93s 
    # hello-job-k592l   0/1     Completed   0          52s 
    # hello-job-pp5ns   0/1     Completed   0          76s 
    # hello-job-spqml   0/1     Completed   0          2m4s
    # hello-job-wfwxx   0/1     Completed   0          60s 
    # hello-job-x9qzj   0/1     Completed   0          116s
    # hello-job-xdpmf   0/1     Completed   0          109s
    # hello-job-zbcmn   0/1     Completed   0          101s
    
    # delete job
    k delete job hello-job
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

---

<h3 id="4">Config Map</h3>

1. Create a file called `config.txt` with two values `key1=value1` and `key2=value2` and verify the file

    ```commandline
    cat >> config.txt << EOF
    key1=value1
    key2=value2
    EOF
    cat config.txt
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

2. Create a configmap named `keyvalcfgmap` and read data from the file `config.txt` and verify that configmap is created correctly

    ```commandline
    k create configmap keyvalcfgmap --from-file=config.txt
    k get configmap keyvalcfgmap --output=yaml
    ```

   Generated ConfigMap resource

    ```yaml
    apiVersion: v1
    data:
      config.txt: |
        key1=value1
        key2=value2
    kind: ConfigMap
    metadata:
      creationTimestamp: "2022-10-15T17:00:31Z"
      name: keyvalcfgmap
      namespace: default
      resourceVersion: "7788"
      uid: 2c6540f6-7674-4bd9-ba59-231dce663abc
    ```

<div align="right"><a href="#home">Back to top 👆</a></div>

3. Create an `nginx` pod and load environment values from the above configmap `keyvalcfgmap` and exec into the pod and verify the environment variables and delete the pod

    ```commandline
    k run nginx \
    --image=nginx \
    --dry-run=client \
    --output=yaml > nginx-pod.yaml
    ```

   Modify pod to use the `configMapRef` property

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        run: nginx
      name: nginx
    spec:
      containers:
        - image: nginx
          name: nginx
          resources: { }
          envFrom:
            - configMapRef:
                name: keyvalcfgmap
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    ```

   Create pod and verify environment variables within it

    ```commandline
    k create --filename=nginx-pod.yaml
    k exec -it nginx -- env
    ```
