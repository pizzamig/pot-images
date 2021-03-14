# OCI images on freeBSD

The goal of this document is to specify and document the format of OCI images on FreeBSD.

Let's start defining layers. A layer is a tarball with folders and files plus a list of folder and files to be removed

Layers uses checksum as identifiers.
Layers are designed to be used in sequence. The first layer is called root layer.

Example:
layer 0: f5a028e4d9dd2bec64d3adff150604f3bb1b7074
layer 1: 3d1d2978d7ce046741409f0ca4df9fe03a0456d4
layer 2: 035eb33f1f1d309cb159a8c6a986a9b6d7b19d3c

Theoretically, layers are independent from each other, the image contains the order in which they have to be applied.

The image will then contain many information: architectural specification (CPU architecture, operating system and version, etc), the ordered list of layers, and many other details (look at the OCI specification [2] for more details)

## Layers with ZFS

Following the docker approach [1], ZFS can support layers via `snapshot` and `clone`

The layer 0 is created by:

* created an empty ZFS dataset
* apply the tarball of the layer 0
* take the snapshot of the layer 0

To create the layer 1, the system can:

* clone the snapshot of the layer 0
* apply the tarball of the layer 1
* take a snapshot of the new layer 1

## Images with ZFS
Images are clones of the top layer. For instance:

* root layer
* layer 1
* layer 2

To use the image, the system can just clone the snapshot of layer 2.

### how to create the layers

For the root layer, the tar ball is just a tarball of the whole file system

For every other layer, only the differences need to be archived. This tarball can be built using `zfs diff`.
New and modified files will be added to the tarball. Deleted files will be managed with `witheouts` as specified in the OCI specification [2] .

## Bibliography

[1] Docker on ZFS: https://docs.docker.com/storage/storagedriver/zfs-driver/#how-the-zfs-storage-driver-works
[2] OCI image specification: https://github.com/opencontainers/image-spec/releases/tag/v1.0.1
