## Advanced Kubernetes Deployments

**Objective:** Create a more advanced deployment, using kubectl to update with apply, rollout, and deploy. Plus, work with ReplicationControllers for the first time.<br>
**Preparation:** You need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using Minikube. Also, navigate to the appropriate lab folder and confirm the below mentioned files are present.<br>
**Outcome:** Once complete the participant will have experience in creating deployments in YAML files, lauching them with kubectl and then updating them.<br>
**Data Files:**  deployment.yaml, deployment-scale.yaml, deployment-update.yaml, nginx-deployment.yaml<br>

You need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using Minikube.
Creating and exploring an nginx deployment

You can run an application by creating a Kubernetes Deployment object, and you can describe a Deployment in a YAML file. For example, this YAML file describes a Deployment that runs the nginx:1.7.9 Docker image:<br>

###### deployment.yaml

```.yaml
apiVersion: extensions/v1beta1 
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2 # tells deployment to run 2 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      # unlike pod-nginx.yaml, the name is not included in the meta data as a unique name is
      # generated from the deployment name
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

#### Step 1. Create a Stateful Apllication Deploymnet

1. Create a Deployment based on the YAML file:

```
$ kubectl create -f <path-to-deployment.yaml>
```

2. Display information about the Deployment:

```
$ kubectl describe deployment nginx-deployment
```

Output:

```
 Name:     nginx-deployment
 Namespace:    default
 CreationTimestamp:  Tue, 30 Aug 2016 18:11:37 -0700
 Labels:     app=nginx
 Selector:   app=nginx
 Replicas:   2 updated | 2 total | 2 available | 0 unavailable
 StrategyType:   RollingUpdate
 MinReadySeconds:  0
 RollingUpdateStrategy:  1 max unavailable, 1 max surge
 Conditions:
   Type          Status  Reason
   ----          ------  ------
   Available     True    MinimumReplicasAvailable
 OldReplicaSets:   <none>
 NewReplicaSet:    nginx-deployment-1771418926 (2/2 replicas created)
 No events.
```

3. List the pods created by the deployment:

```
$ kubectl get pods -l app=nginx
```

Output:

```
 NAME                                READY     STATUS    RESTARTS   AGE
 nginx-deployment-1771418926-7o5ns   1/1       Running   0          16h
 nginx-deployment-1771418926-r18az   1/1       Running   0          16h
```

4. Display information about a pod:

```
$ kubectl describe pod <pod-name>
```

>NOTE:`<pod-name>` is the name of one of your pods.

#### Step 2. Updating the Deployment

You can update the deployment by applying a new YAML file. This YAML file specifies that the deployment should be updated to use nginx 1.8.<br>

###### deployment-update.yaml

```.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8 # Update the version of nginx from 1.7.9 to 1.8
        ports:
        - containerPort: 80
```

1. Apply the new YAML file:

```
$ kubectl apply -f <path-to-deployment-update.yaml>
```

2. Watch the deployment create pods with new names and delete the old pods:

```
$ kubectl get pods -l app=nginx
```

>NOTE: Continue to repeat this command (#2) and watch the changes take effect. 
 
#### Step 3. Scaling Application by Increasing the Replica Count

You can increase the number of pods in your Deployment by applying a new YAML file. This YAML file sets replicas to 4, which specifies that the Deployment should have four pods:<br>

###### deployment-scale.yaml<br> 

```.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 4   # Update the replicas from 2 to 4
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8
        ports:
        - containerPort: 80
```

1. Apply the new YAML file:

```
$ kubectl apply -f <path-to-deployment-scale.yaml>
```

2. Verify that the Deployment has four pods:

```
$ kubectl get pods -l app=nginx
```
The output is similar to this:

```
 NAME                               READY     STATUS    RESTARTS   AGE
 nginx-deployment-148880595-4zdqq   1/1       Running   0          25s
 nginx-deployment-148880595-6zgi1   1/1       Running   0          25s
 nginx-deployment-148880595-fxcez   1/1       Running   0          2m
 nginx-deployment-148880595-rwovn   1/1       Running   0          2m
```

3. Delete the deployment by name:

```
$ kubectl delete deployment nginx-deployment
```

#### Step 4. Creating a Advanced Deployment

Here is an example Deployment. It creates a ReplicaSet to bring up 3 nginx Pods.<br>

###### nginx-deployment.yaml

```.yaml
apiVersion: extensions/v1beta1 
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

1. Run the example by downloading the example file and then running this command:

```
$ kubectl create -f nginx-deployment.yaml --record
```

Output:

```
deployment "nginx-deployment" created
```

Setting the kubectl flag --record to true allows you to record current command in the annotations of the resources being created or updated. It will be useful for future introspection; for example, to see the commands executed in each Deployment revision.

2. Then, running get immediately will give:

```
$ kubectl get deployments
```

Output:

```
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         0         0            0           1s
```

This indicates that the Deployment’s number of desired replicas is 3 (according to deployment’s `.spec.replicas`), the number of current replicas (`.status.replicas`) is 0, the number of up-to-date replicas (`.status.updatedReplicas`) is 0, and the number of available replicas (`.status.availableReplicas`) is also 0.

3. To see the Deployment rollout status, simply run:

```
$ kubectl rollout status deployment/nginx-deployment
```
Output:

```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
```

4. Running the get again a few seconds later, should give:

```
$ kubectl get deployments
```

Output:

```
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           18s
```

5. This indicates that the Deployment has created all three replicas, and all replicas are up-to-date (contains the latest pod template) and available (pod status is ready for at least Deployment’s `.spec.minReadySeconds`). Running `kubectl get rs` and `kubectl get pods` will show the ReplicaSet (RS) and Pods created.

```
$ kubectl get rs
```

Output:

```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-2035384211   3         3         3       18s
```

6. You may notice that the name of the ReplicaSet is always <the-name-of-the-Deployment>-<hash-value-of-the-pod-template>.

```
$ kubectl get pods --show-labels
```

Output:

```
NAME                                READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-2035384211-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=2035384211
nginx-deployment-2035384211-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=2035384211
nginx-deployment-2035384211-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=2035384211
```

The created ReplicaSet will ensure that there are three nginx Pods at all times.

> Note: You must specify appropriate selector and pod template labels of a Deployment (in this case, app = nginx), i.e. don’t overlap with other controllers (including Deployments, ReplicaSets, ReplicationControllers, etc.) Kubernetes won’t stop you from doing that, and if you end up with multiple controllers that have overlapping selectors, those controllers will fight with each other and won’t behave correctly.

#### Step 5. Updating a Deployment

A Deployment’s rollout is triggered if and only if the Deployment’s pod template (i.e. .spec.template) is changed, e.g. updating labels or container images of the template. Other updates, such as scaling the Deployment, will not trigger a rollout.

1. Suppose that we now want to update the nginx Pods to start using the `nginx:1.9.1` image instead of the `nginx:1.7.9` image.

```
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
```
Output:

```
deployment "nginx-deployment" image updated
```

2. Alternatively, we can edit the Deployment and change `.spec.template.spec.containers[0].image` from `nginx:1.7.9` to `nginx:1.9.1`:

```
$ kubectl edit deployment/nginx-deployment
```

>NOTE: Katacoda uses Vim, so exit useing (esc), (:q). Ask instructor if you cannot exit successfully.

Output:

```
deployment "nginx-deployment" edited
```

3. To see its rollout status, simply run:

```
$ kubectl rollout status deployment/nginx-deployment
```

Output:

```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
```

4. After the rollout succeeds, you may want to get the Deployment:

```
$ kubectl get deployments
```

Output:

```
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           36s
```
The number of up-to-date replicas indicates that the Deployment has updated the replicas to the latest configuration. The current replicas indicates the total replicas this Deployment manages, and the available replicas indicates the number of current replicas that are available.

5. We can run `kubectl get rs` to see that the Deployment updated the Pods by creating a new `ReplicaSet` and scaling it up to 3 replicas, as well as scaling down the old `ReplicaSet` to 0 replicas.

```
$ kubectl get rs
```

Output:

```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   3         3         3       6s
nginx-deployment-2035384211   0         0         0       36s
```

6. Running get pods should now show only the new Pods:

```
$ kubectl get pods
```

Output:

```
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-1564180365-khku8   1/1       Running   0          14s
nginx-deployment-1564180365-nacti   1/1       Running   0          14s
nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
```

Next time we want to update these Pods, we only need to update the Deployment’s pod template again.

7. Now, let's check out deployments:

```
$ kubectl describe deployments
```

Output:

```
Name:           nginx-deployment
Namespace:      default
CreationTimestamp:  Tue, 15 Mar 2016 12:01:06 -0700
Labels:         app=nginx
Selector:       app=nginx
Replicas:       3 updated | 3 total | 3 available | 0 unavailable
StrategyType:       RollingUpdate
MinReadySeconds:    0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
OldReplicaSets:     <none>
NewReplicaSet:      nginx-deployment-1564180365 (3/3 replicas created)
Events:
  FirstSeen LastSeen    Count   From                     SubobjectPath   Type        Reason              Message
  --------- --------    -----   ----                     -------------   --------    ------              -------
  36s       36s         1       {deployment-controller }                 Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
  23s       23s         1       {deployment-controller }                 Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
  23s       23s         1       {deployment-controller }                 Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
  23s       23s         1       {deployment-controller }                 Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
  21s       21s         1       {deployment-controller }                 Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
  21s       21s         1       {deployment-controller }                 Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
```

Here we see that when we first created the Deployment, it created a `ReplicaSet` (`nginx-deployment-2035384211`) and scaled it up to 3 replicas directly. When we updated the Deployment, it created a new ReplicaSet (`nginx-deployment-1564180365`) and scaled it up to 1 and then scaled down the old `ReplicaSet` to 2, so that at least 2 Pods were available and at most 4 Pods were created at all times. It then continued scaling up and down the new and the old `ReplicaSet`, with the same rolling update strategy. Finally, we’ll have 3 available replicas in the new `ReplicaSet`, and the old `ReplicaSet` is scaled down to 0.

8. Suppose that we made a typo while updating the Deployment, by putting the image name as `nginx:1.91` instead of `nginx:1.9.1`:

```
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.91
```

Output:

```
deployment "nginx-deployment" image updated
```

9. The rollout will be stuck.

```
$ kubectl rollout status deployments nginx-deployment
```

Output:

```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
```

Press `Ctrl-C` to stop the above rollout status watch. 

10. You will also see that both the number of old replicas (`nginx-deployment-1564180365` and `nginx-deployment-2035384211`) and new replicas (`nginx-deployment-3066724191`) are 2.

```
$ kubectl get rs
```

Output:

```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   2         2         0       25s
nginx-deployment-2035384211   0         0         0       36s
nginx-deployment-3066724191   2         2         2       6s
```

11. Looking at the Pods created, you will see that the 2 Pods created by new ReplicaSet are stuck in an image pull loop.

```
$ kubectl get pods
```

Output:

```
NAME                                READY     STATUS             RESTARTS   AGE
nginx-deployment-1564180365-70iae   1/1       Running            0          25s
nginx-deployment-1564180365-jbqqo   1/1       Running            0          25s
nginx-deployment-3066724191-08mng   0/1       ImagePullBackOff   0          6s
nginx-deployment-3066724191-eocby   0/1       ImagePullBackOff   0          6s
```

> Note: The Deployment controller will stop the bad rollout automatically, and will stop scaling up the new ReplicaSet. This depends on the rollingUpdate parameters (maxUnavailable specifically) that you have specified. Kubernetes by default sets the value to 1 and spec.replicas to 1 so if you haven’t cared about setting those parameters, your Deployment can have 100% unavailability by default! This will be fixed in Kubernetes in a future version.

12. Again, let's check our deployment:

```
$ kubectl describe deployment
```

Output:

```
Name:           nginx-deployment
Namespace:      default
CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
Labels:         app=nginx
Selector:       app=nginx
Replicas:       2 updated | 3 total | 2 available | 2 unavailable
StrategyType:       RollingUpdate
MinReadySeconds:    0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
OldReplicaSets:     nginx-deployment-1564180365 (2/2 replicas created)
NewReplicaSet:      nginx-deployment-3066724191 (2/2 replicas created)
Events:
  FirstSeen LastSeen    Count   From                    SubobjectPath   Type        Reason              Message
  --------- --------    -----   ----                    -------------   --------    ------              -------
  1m        1m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
  21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
  21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
  13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
  13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-1564180365 to 2
  13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 2
```

To fix this, we need to rollback to a previous revision of Deployment that is stable.

#### Step 6. Checking Rollout History of a Deployment

1. First, check the revisions of this deployment:

```
$ kubectl rollout history deployment/nginx-deployment
```

Output:

```
deployments "nginx-deployment"
REVISION    CHANGE-CAUSE
1           kubectl create -f docs/user-guide/nginx-deployment.yaml --record
2           kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
3           kubectl set image deployment/nginx-deployment nginx=nginx:1.91
```

Because we recorded the command while creating this Deployment using --record, we can easily see the changes we made in each revision.

2. To further see the details of each revision, run:

```
$ kubectl rollout history deployment/nginx-deployment --revision=2
```

Output:

```
deployments "nginx-deployment" revision 2
  Labels:       app=nginx
          pod-template-hash=1159050644
  Annotations:  kubernetes.io/change-cause=kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
  Containers:
   nginx:
    Image:      nginx:1.9.1
    Port:       80/TCP
     QoS Tier:
        cpu:      BestEffort
        memory:   BestEffort
    Environment Variables:      <none>
  No volumes.
  ```
  
#### Step 7.  Rolling Back to a Previous Revision
  
1. Now we’ve decided to undo the current rollout and rollback to the previous revision:

```
$ kubectl rollout undo deployment/nginx-deployment
```

Output:

```
deployment "nginx-deployment" rolled back
```

2. Alternatively, you can rollback to a specific revision by specify that in `--to-revision`:

```
$ kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

Output:

```
deployment "nginx-deployment" rolled back
```

3. The Deployment is now rolled back to a previous stable revision. As you can see, a DeploymentRollback event for rolling back to revision 2 is generated from Deployment controller. Let's verify deployment:

```
$ kubectl get deployment
```

Output:

```
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           30m
```

4. Get a detailed account:

```
$ kubectl describe deployment
```

Output:

```
Name:           nginx-deployment
Namespace:      default
CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
Labels:         app=nginx
Selector:       app=nginx
Replicas:       3 updated | 3 total | 3 available | 0 unavailable
StrategyType:       RollingUpdate
MinReadySeconds:    0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
OldReplicaSets:     <none>
NewReplicaSet:      nginx-deployment-1564180365 (3/3 replicas created)
Events:
  FirstSeen LastSeen    Count   From                    SubobjectPath   Type        Reason              Message
  --------- --------    -----   ----                    -------------   --------    ------              -------
  30m       30m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 2
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
  29m       29m         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-1564180365 to 2
  2m        2m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-3066724191 to 0
  2m        2m          1       {deployment-controller }                Normal      DeploymentRollback  Rolled back deployment "nginx-deployment" to revision 2
  29m       2m          2       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
```

#### Step 8. Scaling a Deployment

1. You can scale a Deployment by using the following command:

```
$ kubectl scale deployment nginx-deployment --replicas 10
```

Output:

```
deployment "nginx-deployment" scaled
```

2. Assuming horizontal pod autoscaling is enabled in your cluster, you can setup an autoscaler for your Deployment and choose the minimum and maximum number of Pods you want to run based on the CPU utilization of your existing Pods.

```
$ kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80
```

Output:

```
deployment "nginx-deployment" autoscaled
```


#### Step 9. Proportional scaling

RollingUpdate Deployments support running multiple versions of an application at the same time. When you or an autoscaler scales a RollingUpdate Deployment that is in the middle of a rollout (either in progress or paused), then the Deployment controller will balance the additional replicas in the existing active ReplicaSets (ReplicaSets with Pods) in order to mitigate risk. This is called proportional scaling.

1. For example, you are running a Deployment with 10 replicas, maxSurge=3, and maxUnavailable=2.

```
$ kubectl get deploy
```

Output:

```
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     10        10        10           10          50s
```

2. You update to a new image which happens to be unresolvable from inside the cluster.

```
$ kubectl set image deploy/nginx-deployment nginx=nginx:sometag
```

Output:

```
deployment "nginx-deployment" image updated
```

3. The image update starts a new rollout with ReplicaSet ```nginx-deployment-1989198191``` but it’s blocked due to the maxUnavailable requirement that we mentioned above.

```
$ kubectl get rs
```

Output:

```
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   5         5         0         9s
nginx-deployment-618515232    8         8         8         1m
```

Then a new scaling request for the Deployment comes along. The autoscaler increments the Deployment replicas to 15. The Deployment controller needs to decide where to add these new 5 replicas. If we weren’t using proportional scaling, all 5 of them would be added in the new ReplicaSet. With proportional scaling, we spread the additional replicas across all ReplicaSets. Bigger proportions go to the ReplicaSets with the most replicas and lower proportions go to ReplicaSets with less replicas. Any leftovers are added to the ReplicaSet with the most replicas. ReplicaSets with zero replicas are not scaled up.

4. In our example above, 3 replicas will be added to the old ReplicaSet and 2 replicas will be added to the new ReplicaSet. The rollout process should eventually move all replicas to the new ReplicaSet, assuming the new replicas become healthy.

```
$ kubectl get deploy
```

Output:

```
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     15        18        7            8           7m
```

5. Now...

```
$ kubectl get rs
```

Output:

```
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   7         7         0         7m
nginx-deployment-618515232    11        11        11        7m
```

#### Step 10. Pausing and Resuming a Deployment

You can pause a Deployment before triggering one or more updates and then resume it. This will allow you to apply multiple fixes in between pausing and resuming without triggering unnecesarry rollouts.

1. For example, with a Deployment that was just created:

```
$ kubectl get deploy
```

Output:

```
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     3         3         3            3           1m
```

2. Then...

```
$ kubectl get rs
```

Output:

```
NAME               DESIRED   CURRENT   READY     AGE
nginx-2142116321   3         3         3         1m
```

3. Pause by running the following command:

```
$ kubectl rollout pause deployment/nginx-deployment
```

Output:

```
deployment "nginx-deployment" paused
```

4. Then update the image of the Deployment:

```
$ kubectl set image deploy/nginx-deployment nginx=nginx:1.9.1
```

Output:

```
deployment "nginx-deployment" image updated
```

5. Notice that no new rollout started:

```
$ kubectl rollout history deploy/nginx
```

Output:

```
deployments "nginx"
REVISION  CHANGE-CAUSE
1   <none>
```

6. We run it again...

```
$ kubectl get rs
```

Output:

```
NAME               DESIRED   CURRENT   READY     AGE
nginx-2142116321   3         3         3         2m
```

7. You can make as many updates as you wish, for example, update the resources that will be used:

```
$ kubectl set resources deployment nginx -c=nginx --limits=cpu=200m,memory=512Mi
```

Output:

```
deployment "nginx" resource requirements updated
```

The initial state of the Deployment prior to pausing it will continue its function, but new updates to the Deployment will not have any effect as long as the Deployment is paused.

8. Eventually, resume the Deployment and observe a new ReplicaSet coming up with all the new updates:

```
$ kubectl rollout resume deploy nginx
```

Output:

```
deployment "nginx" resumed
```

9. Let's use the watch (-w) option this time, like this:

```
$ kubectl get rs -w
```

Output:

```
NAME               DESIRED   CURRENT   READY     AGE
nginx-2142116321   2         2         2         2m
nginx-3926361531   2         2         0         6s
nginx-3926361531   2         2         1         18s
nginx-2142116321   1         2         2         2m
nginx-2142116321   1         2         2         2m
nginx-3926361531   3         2         1         18s
nginx-3926361531   3         2         1         18s
nginx-2142116321   1         1         1         2m
nginx-3926361531   3         3         1         18s
nginx-3926361531   3         3         2         19s
nginx-2142116321   0         1         1         2m
nginx-2142116321   0         1         1         2m
nginx-2142116321   0         0         0         2m
nginx-3926361531   3         3         3         20s
...
^C
```

10. Following up with:

```
$ kubectl get rs
```

Output:

```
NAME               DESIRED   CURRENT   READY     AGE
nginx-2142116321   0         0         0         2m
nginx-3926361531   3         3         3         28s
```

>NOTE: You cannot rollback a paused Deployment until you resume it.

#### Step 11. Deployment Status

A Deployment enters various states during its lifecycle. It can be progressing while rolling out a new ReplicaSet, it can be complete, or it can fail to progress.

You can check if a Deployment has completed by using kubectl rollout status. If the rollout completed successfully, kubectl rollout status returns a zero exit code.

```
$ kubectl rollout status deploy/nginx
```

Output:

```
Waiting for rollout to finish: 2 of 3 updated replicas are available...
deployment "nginx" successfully rolled out
$ echo $?
0
``` 

The following kubectl command sets the spec with progressDeadlineSeconds to make the controller report lack of progress for a Deployment after 10 minutes:

```
$ kubectl patch deployment/nginx-deployment -p '{"spec":{"progressDeadlineSeconds":600}}'
```

Output:

```
"nginx-deployment" patched
```

You may experience transient errors with your Deployments, either due to a low timeout that you have set or due to any other kind of error that can be treated as transient. For example, let’s suppose you have insufficient quota. If you describe the Deployment you will notice the following section:

```
$ kubectl describe deployment nginx-deployment
```

Output:

```
<...>
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     True    ReplicaSetUpdated
  ReplicaFailure  True    FailedCreate
<...>
```

If you run ```kubectl get deployment nginx-deployment -o yaml```, the Deployement status might look like this:

```
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: Replica set "nginx-deployment-4262182780" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  - lastTransitionTime: 2016-10-04T12:25:42Z
    lastUpdateTime: 2016-10-04T12:25:42Z
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: 'Error creating: pods "nginx-deployment-4262182780-" is forbidden: exceeded quota:
      object-counts, requested: pods=1, used: pods=3, limited: pods=2'
    reason: FailedCreate
    status: "True"
    type: ReplicaFailure
  observedGeneration: 3
  replicas: 2
  unavailableReplicas: 2
```

Eventually, once the Deployment progress deadline is exceeded, Kubernetes updates the status and the reason for the Progressing condition:

```
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     False   ProgressDeadlineExceeded
  ReplicaFailure  True    FailedCreate
```

You can check if a Deployment has failed to progress by using kubectl rollout status. kubectl rollout status returns a non-zero exit code if the Deployment has exceeded the progression deadline.

```
$ kubectl rollout status deploy/nginx
```

Output:

```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
error: deployment "nginx" exceeded its progress deadline
$ echo $?
1
```
