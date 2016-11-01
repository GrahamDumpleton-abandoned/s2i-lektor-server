# Lektor Site Generator

This repository implements a [Lektor](https://www.getlektor.com) site generator and hosting service. It is implemented as a [Source-to-Image](https://github.com/openshift/source-to-image) (S2I) builder and can be used to generate a standalone Docker-formatted image which you can run with any container runtime supporting Docker-formatted images. Alternatively the S2I builder can be used in conjunction with OpenShift to handle both the build and deployment of the generated Lektor formatted site.

## Integration With OpenShift

To use this with OpenShift, it is a simple matter of creating a new application within OpenShift, pointing the S2I builder at the Git repository containing your Lektor document source.

As an example, to build and host the the [Lektor site](https://github.com/lektor/lektor-website), you only need run:

```
oc new-app getwarped/s2i-lektor-server:2.3~https://github.com/lektor/lektor-website.git --name lektor-site

oc expose svc/lektor-site
```

To have any changes to your document source automatically redeployed when changes are pushed back up to your Git repository, you can use the [web hooks integration](https://docs.openshift.com/container-platform/latest/dev_guide/builds.html#webhook-triggers) of OpenShift to create a link from your Git repository hosting service back to OpenShift.

To make it easier to deploy a Lektor site, a template for OpenShift is also included. This can be loaded into your project using:

```
oc create -f https://raw.githubusercontent.com/getwarped/s2i-lektor-server/master/template.json
```

Once loaded, select the ``lektor-server`` template from the web console when wanting to add a new site to the project.

The template details are:

```
Name:		lektor-server
Description:	Lektor Server
Annotations:	tags=instant-app,lektor

Parameters:
    Name:		APPLICATION_NAME
    Description:	The name of the application.
    Required:		true
    Value:		lektor-server

    Name:		SOURCE_REPOSITORY
    Description:	Git repository for source.
    Required:		true
    Value:		<none>
    
    Name:		SOURCE_DIRECTORY
    Description:	Sub-directory of repository for source files.
    Required:		false
    Value:		<none>
    
    Name:		DOCUMENT_ROOT
    Description:	Sub-directory of source directory for documents.
    Required:		false
    Value:		<none>
    
    Name:		BUILD_MEMORY_LIMIT
    Description:	Build time memory limit.
    Required:		true
    Value:		512Mi

Objects:
    ImageStream		${APPLICATION_NAME}
    ImageStream		${APPLICATION_NAME}-s2i
    DeploymentConfig	${APPLICATION_NAME}
    Service		${APPLICATION_NAME}
    Route		${APPLICATION_NAME}
```

The ``APPLICATION_NAME`` and ``SOURCE_REPOSITORY`` must be specified. Override the ``BUILD_MEMORY_LIMIT`` and set it to a higher memory value if the ``lektor`` application keeps getting killed when processing the site data.

## Standalone Docker Images

To create a standalone Docker-formatted image, you need to [install](https://github.com/openshift/source-to-image/releases) the ``s2i`` program from the Source-to-Image (S2I) project locally. Once you have this installed, you would run within your Git repository:

```
s2i build . getwarped/s2i-lektor-server:2.3 mylektorsite
```

In this case this will create a Docker-formatted image called ``mylektorsite``. You can then run the image using:

```
docker run --rm -p 8080:8080 mylektorsite
```