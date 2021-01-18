# FlexVolume

The [Kubernetes Storage Special Interest Group (SIG)](https://github.com/kubernetes/community/blob/master/sig-storage) defines three methods to implement a volume plugin:

1. In-tree volume plugin [deprecated],
2. Out-of-tree FlexVolume driver [deprecated],
3. Out-of-tree CSI driver.

Flexvolume is deprecated, but the Kubernetes Storage-SIG plans to continue to support and maintain the Flex Volume API.

As of Kubernetes 1.9, there are two `out-of-tree` methods to implement volume plugins: `Container Storage Interface (CSI)` and `FlexVolume`.

`Out-of-tree` volume plugins enable storage developers to create custom storage plugins. Before the introduction of the CSI and FlexVolume, all volume plugins were `in-tree` meaning they were built, linked, compiled, and shipped with the core Kubernetes binaries and extend the core Kubernetes API.

FlexVolume has existed since Kubernetes 1.2, and is a GA feature since Kubernetes 1.8. FlexVolume uses an exec-based model to interface with drivers. The FlexVolume driver binaries must be installed in a pre-defined volume plugin path on each node and in some cases the control plane nodes as well. Pods interact with FlexVolume drivers through the flexvolume in-tree volume plugin.

The plugin expects the following call-outs are implemented for the backend drivers. Call-outs are invoked from Kubelet and Controller Manager.

* Init,
* Attach,
* Detach,
* Wait for attach,
* Volume is attached,
* Mount device,
* Unmount device,
* Mount,
* Unmount.

For more information about FlexVolume, see [flex](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-storage/flexvolume.md).
