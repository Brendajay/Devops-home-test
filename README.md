
#1.) 
To deploy two containers that run a small API returning users and shifts from a database, we will create a Kubernetes deployment.One for the users container and another for the shifts container. Each container will run on a separate pod, and each pod will have its own isolated database.  
Also we will use a Kubernetes StatefulSet to manage the database state for each container.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  labels:
    app: api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: users
        image: <users-image>
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 100m
            memory: 256Mi
        ports:
        - containerPort: 80
        env:
        - name: DB_HOST
          value: <users-db-host>
        - name: DB_NAME
          value: users
        - name: DB_USER
          value: <users-db-user>
        - name: DB_PASSWORD
          value: <users-db-password>
      - name: shifts
        image: <shifts-image>
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 100m
            memory: 256Mi
        ports:
        - containerPort: 80
        env:
        - name: DB_HOST
          value: <shifts-db-host>
        - name: DB_NAME
          value: shifts
        - name: DB_USER
          value: <shifts-db-user>
        - name: DB_PASSWORD
          value: <shifts-db-password>
      imagePullSecrets:
      - name: <registry-secret-name>
      
In the manifest, we define two containers which are (users and shifts) each with its own CPU and memory resource limits and requests. We also define the environment variables needed to connect to each container's database. 

        
#2.) 
To auto-scale the deployment based on CPU usage, we can use a Kubernetes Horizontal Pod Autoscaler (HPA) object. The HPA will automatically adjust the number of replicas based on the average CPU utilization of the pods which happens to be 70% in this case.

apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 70

In the manifest, we define an HPA object that targets the api-deployment deployment and sets the minimum and maximum number of replicas. The HPA will adjust the number of replicas based on the average CPU utilization of the pods. 
        


#3.) Ensure the deployment can handle rolling deployments and rollbacks.
To handle rolling deployments and rollbacks, we can use Kubernetes Deployments. We can set the strategy field in the Deployment spec to RollingUpdate, which will     ensure that updates are applied gradually, one replica at a time, and that the previous version is still running until the new version is fully deployed. We can also set the revisionHistoryLimit field to limit the number of old replicas we keep.
•	We will use Kubernetes Deployments to manage our container updates, which support rolling updates and rollbacks.
•	When we update the container image, Kubernetes will create a new ReplicaSet with the updated image, and gradually shift traffic to the new replicas while terminating the old ones.
•	If the update causes issues, we can quickly roll back to the previous version by reverting the Deployment to the previous revision.




#4.) Your development team should not be able to run certain commands on your k8s cluster, but you want them to be able to deploy and roll back. What types of IAM controls do you put in place?
To prevent the development team from running certain commands on the Kubernetes cluster, we can use RBAC (Role-Based Access Control) to create a custom role with specific permissions. For example, we can create a role that only allows the deployment and rollback of the application, but does not allow access to other Kubernetes resources like pods or services. We can then assign this role to the development team's service account or user account.

Bonus
# How would you apply the configs to multiple environments (staging vs production)?
we can use Kubernetes namespaces to separate the environments. We can create a namespace for each environment and deploy the resources to their respective namespace. For example, we can create a "staging" namespace and a "production" namespace, and deploy the resources to the appropriate namespace.


#How would you auto-scale the deployment based on network latency instead of CPU?

 To auto-scale based on network latency, we can use Kubernetes custom metrics. We can create a custom metric server that monitors the network latency between the users and shifts containers, and expose the metric as a custom metric in Kubernetes. We can then create a HorizontalPodAutoscaler that uses the custom metric as the scaling metric, and set the target value to the desired network latency. For example, we







