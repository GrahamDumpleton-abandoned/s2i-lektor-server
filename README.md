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

## Standalone Docker Images

To create a standalone Docker-formatted image, you need to [install](https://github.com/openshift/source-to-image/releases) the ``s2i`` program from the Source-to-Image (S2I) project locally. Once you have this installed, you would run within your Git repository:

```
s2i build . getwarped/s2i-lektor-server:2.3 mylektorsite
```

In this case this will create a Docker-formatted image called ``mylektorsite``. You can then run the image using:

```
docker run --rm -p 8080:8080 mylektorsite
```