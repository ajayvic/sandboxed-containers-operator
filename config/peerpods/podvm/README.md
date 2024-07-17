# Introduction

This is a brief readme explaining the usage of the podvm-builder scripts and
related files.  The scripts and related manifest files are primarily used by
the operator to generate a pod VM image.

## PodVM image generation configuration

The configuration used for the podvm image generation is available in the following configmaps:

- Azure: `azure-podvm-image-cm`
- AWS: `aws-podvm-image-cm`

If you want to change the default configuration, then depending on the cloud
provider (eg. aws or azure) you'll need to pre-create the respective
configmaps.  Please review and modify the settings in the configMap as
required.  For example, if you need to add NVIDIA GPU drivers in the podvm
image then set `ENABLE_NVIDIA_GPU: yes`. Likewise if you want to create image
for confidential containers then set `CONFIDENTIAL_COMPUTE_ENABLED: yes`.

Use the following command to create the configMap for AWS:

```sh
kubectl apply -f aws-podvm-image-cm.yaml
```

Use the following command to create the configMap for Azure:

```sh
kubectl apply -f azure-podvm-image-cm.yaml
```

Now when you create a KataConfig with `enablePeerPods: true` with empty
`AZURE_IMAGE_ID` or `AWS_AMI_ID` in `peer-pods-cm`, then depending on the cloud
provider configured, the operator will create the pod VM image based on the
provided config.

## PodVM image upload configuration

The PodVM image can be embedded to an OCI image and used to unwrap and upload using the uploader job. 
This feature is currently supported only for libvirt provider.

To build an OCI image with PodVM image, `Dockerfile.podvm-uploader` can be used as,
```
docker build -t podvm-libvirt \
    --build-arg ACTIVATION_KEY=<rhel_activation_key_for_subscription> \
    --build-arg ORG_ID=<org_id_for_subscription> \
    --build-arg PODVM_IMAGE_SRC=<podvm_image_source> \
    -f Dockerfile.podvm-uploader .
```
Here the `ACTIVATION_KEY` and `ORG_ID` is used for subscription and `PODVM_IMAGE_SRC` is where the `qcow2` image is present.


Ensure that the following values are set on the `libvirt-podvm-image-cm`:
```
LIBVIRT_IMAGE_TYPE: "pre-built"
LIBVIRT_IMAGE_SOURCE: "quay.io/openshift_sandboxed_containers/libvirt-podvm-image:latest"
LIBVIRT_PODVM_IMAGE_PATH: "/image/podvm-s390x.qcow2"
```
