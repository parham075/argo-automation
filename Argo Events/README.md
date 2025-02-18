In this section I will provide a tutorial to implement argo events using [Quick Start](https://argoproj.github.io/argo-events/installation/)

- ## Introduction:
    Argo Events is an event-driven workflow automation framework for Kubernetes which helps you trigger K8s objects, Argo Workflows, Serverless workloads, etc. on events from a variety of sources like webhooks, S3, schedules, messaging queues, gcp pubsub, sns, sqs, etc.
- ## Architecture:
    Main components of Argo Events are:
    - Event Source: An EventSource defines the configurations required to consume events from external sources like AWS SNS, SQS, GCP PubSub, Webhooks, etc. It further transforms the events into the cloudevents and dispatches them over to the eventbus.
    - Sensor: Sensor defines a set of event dependencies (inputs) and triggers (outputs). It listens to events on the eventbus and acts as an event dependency manager to resolve and execute the triggers.
    - Eventbus: The EventBus acts as the transport layer of Argo-Events by connecting the EventSources and Sensors.EventSources publish the events while the Sensors subscribe to the events to execute triggers. There are three implementations of the EventBus: NATS (deprecated), Jetstream, and Kafka.
    - Trigger: A Trigger is the resource/workload executed by the sensor once the event dependencies are resolved.
    The architecture of Argo Events can is illustrated as below:
    
    <p align="center" ><img src="https://argoproj.github.io/argo-events/assets/argo-events-architecture.png" alt="Argo Events' architecture" width="80%" height="100%"style=" display: block; margin: 20 auto;"/></p>
- ## Installation:
    - Create namespace:
        ```
        kubectl create ns argo-events
        ```
    - Deploy Argo Events SA, ClusterRoles, and Controller for Sensor, EventBus, and EventSource.
        ```
        kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/namespace-install.yaml
        ```

        > NOTE:
        > ```
        >  * On GKE, you may need to grant your account the ability to create new custom    resource definitions and clusterroles
        >       kubectl create clusterrolebinding YOURNAME-cluster-admin-binding --clusterrole=cluster-admin --user=YOUREMAIL@gmail.com
        >  * On OpenShift:
        >        - Make sure to grant `anyuid` scc to the service accounts.
        >
        >            oc adm policy add-scc-to-user anyuid system:serviceaccount:argo-events:argo-events-sa system:serviceaccount:argo-events:argo-events-webhook-sa
        >
        >        - Add update permissions for the `deployments/finalizers` and `clusterroles/finalizers` of the argo-events-webhook ClusterRole(this is necessary for the validating admission controller)
        >
        >            - apiGroups:
        >            - rbac.authorization.k8s.io
        >            resources:
        >            - clusterroles/finalizers
        >            verbs:
        >            - update
        >            - apiGroups:
        >            - apps
        >            resources:
        >            - deployments/finalizers
        >            verbs:
        >            - update
        >
        > ```
    
    - Deploy the eventbus
        ```
        kubectl apply -n argo-events -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/eventbus/native.yaml
        ```