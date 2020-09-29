# pizza-operator project

This project uses Quarkus, the Supersonic Subatomic Java Framework to create a Kubernetes Operator following 
[@Alex Soto](https://twitter.com/alexsotob) cheat sheet [Writing a Kubernetes Operator in Java](https://t.co/4m7kSKUPj9?amp=1)


If you want to learn more about Quarkus, please visit its website: https://quarkus.io/ .

## Creating the .mvn folder for the Maven Wrapper

This project is based on the Maven Wrapper, that needs a .mvn folder created with the wrapper jar inside

Execute following command the first time you are going to build the project :
```
mvn -N io.takari:maven:wrapper
```

## Running the application in dev mode

You can run your application in dev mode that enables live coding using:
```
./mvnw quarkus:dev
```

## Packaging and running the application

The application can be packaged using `./mvnw package`.
It produces the `pizza-operator-1.0-SNAPSHOT-runner.jar` file in the `/target` directory.
Be aware that it’s not an _über-jar_ as the dependencies are copied into the `target/lib` directory.

The application is now runnable using `java -jar target/pizza-operator-1.0-SNAPSHOT-runner.jar`.

## Creating a native executable

You can create a native executable using: `./mvnw package -Pnative`.

Or, if you don't have GraalVM installed, you can run the native executable build in a container using: `./mvnw package -Pnative -Dquarkus.native.container-build=true`.

You can then execute your native executable with: `./target/pizza-operator-1.0-SNAPSHOT-runner`

If you want to learn more about building native executables, please consult https://quarkus.io/guides/building-native-image.

## Publishing the operator on Quay

The different deployment yaml files are using a public accessible container image of our operator

To create the container image we should do :
```
docker build -f src/main/docker/Dockerfile.native -t quay.io/{your-quay-user}/pizza-operator-native .
```

And now it's the turn to push this image to Quay ( quay.io)

```
docker login quay.io
docker push quay.io/{your-quay-user}/pizza-operator-native
```

## Testing

By default, Pods in Kubernetes do not have the permission to list other pods. Therefore, we need to create a cluster role, a service account, and a cluster role binding.

    kubectl apply -f k8s_files/operator.clusterrole.yaml
    kubectl apply -f k8s_files/operator.serviceaccount.yaml
    kubectl apply -f k8s_files/operator.clusterrolebinding.yaml

Now you can run the `kubectl apply -f k8s_files/operator.crd.yaml` command to register the CRD in the cluster. 

Now it's the time to modify the deployment file `operator-deployment.yaml` to point to your image

Changing
```
        - image: quarkus/pizza-operator-native
```
by
```
        - image: {your-quay-user}/pizza-operator-native
```

Run the `kubectl apply -f k8s_files/operator.deployment.yaml` command to register the operator.


### Running the example
Apply the custom resource by running: `kubectl apply -f meat-pizza.yaml` and check the output of 
`kubectl get pods` command.

```
> k get pods
NAME                                        READY   STATUS      RESTARTS   AGE
meats-pod                               0/1     Completed   0          7m10s
quarkus-operator-example-554b8f45fc-mcgqn   1/1     Running     0          7m25s
```

If you check the log in the `meats-pod` you can see how the Operator created and delivered our pizza ;-)
```
│ __  ____  __  _____   ___  __ ____  ______                                                                                        │
│  --/ __ \/ / / / _ | / _ \/ //_/ / / / __/                                                                                        │
│  -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \                                                                                          │
│ --\___\_\____/_/ |_/_/|_/_/|_|\____/___/                                                                                          │
│ 2020-05-21 14:17:32,324 INFO  [io.quarkus] (main) pizza-maker 1.0-SNAPSHOT (powered by Quarkus 1.4.0.CR1) started in 0.027s.      │
│ 2020-05-21 14:17:32,324 INFO  [io.quarkus] (main) Profile prod activated.                                                         │
│ 2020-05-21 14:17:32,324 INFO  [io.quarkus] (main) Installed features: [cdi]                                                       │
│ Doing The Base                                                                                                                    │
│ Adding Sauce bbq                                                                                                                  │
│ Adding Toppings [mozzarella,pepperoni,tuna,mushrooms]                                                                             │
│ Baking                                                                                                                            │
│ Baked                                                                                                                             │
│ Ready For Delivery                                                                                                                │
│ 2020-05-21 14:17:32,825 INFO  [io.quarkus] (main) pizza-maker stopped in 0.001s                                                   │
│
```

