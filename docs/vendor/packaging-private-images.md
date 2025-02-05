# Connecting to an Image Registry

You can use the Replicated private registry or any supported
external private or public registry with your application.

## Use the Replicated private registry

You can push the private image for your application to the
Replicated private registry.

For more information about building, tagging and pushing docker images, see the
[Docker CLI documentation](https://docs.docker.com/engine/reference/commandline/cli/).

To use the Replicated private registry:

1. Do one of the following to to connect with the `registry.replicated.com` container registry:
   * **Log in with a service account or team token**: Use `docker login registry.replicated.com` with a Replicated vendor portal service account or team token as the password. You can use any string as the username. For more information, see [Service accounts](../reference/replicated-cli-tokens#service-accounts) and [Team tokens](../reference/replicated-cli-tokens#team-tokens) in _Using Vendor API tokens_.
   * **Log in with a user token**: Use `docker login registry.replicated.com` with your vendor portal email as the username and a vendor portal user token as the password. For more information, see [User tokens](../reference/replicated-cli-tokens#user-tokens) in _Using Vendor API tokens_.
   * **Log in with your credentials**: Use `docker login registry.replicated.com` with your vendor portal email and password as the credentials.

1. Tag your private image with the Replicated registry hostname in the standard
Docker format:

   ```
   docker tag IMAGE_NAME registry.replicated.com/APPLICATION_SLUG/TARGET_IMAGE_NAME:TAG
   ```

   Where:
   * `IMAGE_NAME` is the name of the existing private image for your application.
   * `APPLICATION_SLUG` is the slug assigned to your application. You can find your application slug on the **Images** page of the vendor portal.
   * `TARGET_IMAGE_NAME` is a name for the image. Replicated recommends that the `TARGET_IMAGE_NAME` is the same as the `IMAGE_NAME`.
   * `TAG` is a tag for the image.

   For example:

   ```shell
   $ docker tag worker registry.replicated.com/myapp/worker:1.0.1
   ```

1. Push your private image to the Replicated private registry:
  ```shell
  $ docker push registry.replicated.com/APPLICATION_SLUG/TARGET_IMAGE_NAME:TAG
  ```

## Use an external registry

When packaging and delivering an enterprise application, a common problem is the need to include private Docker images.
Most enterprise applications consist of public images (postgres, mysql, redis, elasticsearch) and private images (the application images).

When delivering an application through the [vendor portal](https://vendor.replicated.com), there’s built-in support to include private images -- without managing or distributing actual registry credentials to your customer.
The license file grants revokable image pull access to private images, whether these are stored in the Replicated private registry, or another private registry server that you’ve decided to use.

If your application images are already available in a private, but accessible image registry (such as Docker Hub, quay.io, ECR, GCR, Artifactory, and so on), then your application licenses can be configured to grant proxy, or pull-through access to the assignee without giving actual credentials to the customer.

This is useful and recommended because it prevents you from having to modify the process you use to build and push application images, and it gives you the ability to revoke a customer’s ability to pull (such as on trial expiration).
This External Registry is shared across all applications in a team, allowing images to be used across multiple apps.

To configure access to your private images, log in to the [vendor portal](https://vendor.replicated.com), and click on the images menu item under your application.
Click **Add External Registry**.
In the modal, enter an endpoint (quay.io, index.docker.io, gcr.io, and so on), and provide a username and password to Replicated that has pull access.
For more information, see the documentation about our registry.
Replicated will store your username and password encrypted and securely, and it (and the encryption key) will never leave our servers.

![Add External Registry](/images/add-external-registry.png)

Your Application custom resource manifest file will reference images that it cannot access.
The app manager recognizes this, and will patch the YAML using Kustomize to change the image name.
When the app manager is attempting to install an application, it will attempt to load image manifest using the image reference from the PodSpec.
If it’s loaded successfully, no changes will be made to the application.
If a 401 is received and authentication is required, the app manager assumes that this is a private image that needs to be proxied through the Replicated registry-proxy service.
A patch will be written to the midstream `kustomization.yaml` to change this image name during deployment.

For example, given a private image hosted at `quay.io/my-org/api:v1.0.1`, a deployment and pod spec may reference it like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example
spec:
  template:
    spec:
      containers:
        - name: api
          image: quay.io/my-org/api:v1.0.1
```

When the application is deployed, the app manager will detect that it cannot access the image at quay.io and will create a patch in the `midstream/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
bases:
- ../../base
images:
- name: quay.io/my-org/api:v1.0.1
  newName: proxy.replicated.com/proxy/my-kots-app/quay.io/my-org/api
```

This will change that image name everywhere it appears.

In addition, the app manager will create an imagePullSecret dynamically and automatically at install time.
This secret is based on the customer license, and will be used to pull all images from `proxy.replicated.com`

Images hosted at `registry.replicated.com` will not be rewritten.  
However, the same secret will be added to those PodSpecs as well.

> Application deployments are supported via image tags in all use cases. The app manager has limited support for deploying via image digests. Use of image digests are only supported for fully online installs where all images can be pulled from the Replicated registry, a public repo, or proxied from a private repo via the Replicated registry.

## Additional namespaces

When deploying pods to namespaces other than the app manager application namespace, the namespace must be added to the `additionalNamespaces` attribute of the [Application](../reference/custom-resource-application) manifest.
This helps to ensure that the application image pull secret will get auto-provisioned by the app manager in the namespace to allow the pod to pull the image.
For more information about the `additionalNamespaces` attribute, see [Defining additional namespaces](operator-defining-additional-namespaces).
