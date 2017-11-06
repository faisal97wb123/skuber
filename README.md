# Skuber

Skuber is a Scala client library for [Kubernetes](http://kubernetes.io). It provides a fully featured, high-level and strongly typed Scala API for managing Kubernetes cluster resources (such as Pods, Services, Deployments, ReplicaSets, Ingresses  etc.) via the Kubernetes REST API server.

## Features

- Comprehensive support for Kubernetes API model represented as Scala case classes
- Support for core, extensions and other Kubernetes API groups
- Full support for converting resources between the case class and standard JSON representations 
- Client API for creating, reading, updating, removing, listing and watching resources on a Kubernetes cluster
- The API is asynchronous and strongly typed e.g. `k8s get[Deployment]("nginx")` returns a value of type `Future[Deployment]`
- Fluent API for creating and updating specifications of Kubernetes resources
- Uses standard `kubeconfig` files for configuration - see the [Configuration guide](docs/Configuration.md) for details

See the [programming guide](docs/GUIDE.md) for more details.

## Example Usage

This example creates a nginx service (accessed via port 30001 on each Kubernetes cluster node) that is backed by five nginx replicas.

    import skuber._
    import skuber.json.format._

    val nginxSelector  = Map("app" -> "nginx")
    val nginxContainer = Container("nginx",image="nginx").exposePort(80)
    val nginxController= ReplicationController("nginx",nginxContainer,nginxSelector)
    	.withReplicas(5)
    val nginxService = Service("nginx")
    	.withSelector(nginxSelector)
    	.exposeOnNodePort(30001 -> 80) 

    import scala.concurrent.ExecutionContext.Implicits.global

    val k8s = k8sInit

    val createOnK8s = for {
      svc <- k8s create nginxService
      rc  <- k8s create nginxController
    } yield (rc,svc)

    createOnK8s onComplete {
      case Success(_) => System.out.println("Successfully created nginx replication controller & service on Kubernetes cluster")
      case Failure(ex) => System.err.println("Encountered exception trying to create resources on Kubernetes cluster: " + ex)
    }

    k8s.close


## Prerequisites

A Kubernetes cluster is needed at runtime, The client has been tested against various releases of Kubernetes, from v1.0 through to v1.7. For local development purposes, minikube is recommended.

You need Java 8 to run Skuber.

## Release

The most up-to-date stable version is 1.7.1, which you can add to your build using:

     libraryDependencies += "io.github.doriordan" % "skuber_2.11" % "1.7.1-RC6"
	
Although that release is labelled as a release candidate (RC6), is is considered stable and only minor changes are likely before it becomes an official release.

Note Skuber v1.x is only available for Scala 2.11. If you need Scala 2.12 support then you must use v2.0. 

## Roadmap

Release v2.0 of Skuber is in progress and an early release candidate is available. Skuber v2.0 is largely compatible with release 1.7.1, but refactors Skuber internals to use Akka Http instead of Play 2.4 as its http client, and brings support for Scala 2.12. It is planned to officially release Skuber v2.0 before the end of 2017. New projects should look to adopt this rather than v1.7.1 if that fits their timeframes as it will be better supported with new features and for changes in new versions of Kubernetes going forward. More details and updated docs (including a migration guide for v1.x users) on the [release branch](https://github.com/doriordan/skuber/tree/release_2.0).

You can test drive the current release 2.0 candidate (for Scala 2.12) by adding to your build:

    libraryDependencies += "io.skuber" % "skuber_2.12" % "2.0.0-RC1"
    
(Note that the organisation setting for the release has changed from "io.github.doriordan" to "io.skuber" going forward)

However for significant evaluation it is currently recommended just to build from the latest release_2.0 branch commit, as it contains a couple of important fixes and enhancements that will be released as RC2 shortly.

## Building

Building the library from source is very straightforward. Simply run `sbt test`in the root directory of the project to build the library (and examples) and run the unit tests to verify the build.

## Quick Start

The quickest way to get started with Skuber:

- If you don't already have Kubernetes installed, then follow the instructions [here](https://github.com/kubernetes/minikube) to install minikube, which is now the recommended way to run Kubernetes locally.

- Ensure Skuber configures itself from the default Kubeconfig file (`$HOME/.kube/config`) : 

	`export SKUBER_CONFIG=file` 

- Try one or more of the examples: if you have cloned this repository run `sbt` in the top-level directory to start sbt in interactive mode and then:

```
    > project examples

    > run
    [warn] Multiple main classes detected.  Run 'show discoveredMainClasses' to see the list

    Multiple main classes detected, select one to run:
    
     [1] skuber.examples.customresources.CreateCRD
     [2] skuber.examples.deployment.DeploymentExamples
     [3] skuber.examples.fluent.FluentExamples
     [4] skuber.examples.guestbook.Guestbook
     [5] skuber.examples.ingress.NginxIngress
     [6] skuber.examples.job.PrintPiJob
     [7] skuber.examples.list.ListExamples
     [8] skuber.examples.scale.ScaleExamples

    Enter number: 
```

For other Kubernetes setups, see the [Configuration guide](docs/Configuration.md) for details on how to tailor the configuration for your clusters security, namespace and connectivity requirements.



## License

This code is licensed under the Apache V2.0 license, a copy of which is included [here](LICENSE.txt).
