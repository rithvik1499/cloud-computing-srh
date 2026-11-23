# Session 4: the generator on GKE

## kubectl get pods -o wide
```
NAME                         READY   STATUS    RESTARTS   AGE     IP           NODE                             NOMINATED NODE   READINESS GATES
generator-7655b67568-65xcc   1/1     Running   0          2m44s   10.16.0.19   gke-freight-main-eefcad73-nmp0   <none>           <none>
```

## kubectl rollout history deployment/generator
```
deployment.apps/generator 
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```

## What happened when I deleted the pod
I deleted the generator pod and Kubernetes created a replacement with a new name within a second. The Deployment declares one replica as the desired state. When the pod was gone, the actual state (zero pods) no longer matched it, so the Deployment's controller started a new one. I never told Kubernetes to restart anything.
