# GPU Manager

This version of GPU Manager is forked from tkestack GPU manager, it builds the GPU manager image based on multi-architecture. 
To be specific, the modification lies in the build/Dockerfile and hack/build.sh

The original project and details can be found here: [gpu-manager](https://github.com/tkestack/gpu-manager)

## Usage

The usage can be found at [Multi-Arch GPU manager docker image setup](https://inside-docupedia.bosch.com/confluence/display/RIXICV/Multi-arch+GPU-manager+docker+image+setup+in+WSL2)

## Prebuilt image

Prebuilt image can be found at `rayrayrayzhang/gpu-manager`

## Deploy

GPU Manager is running as daemonset, and because of the RABC restriction and hydrid cluster,
you need to do the following steps to make this daemonset run correctly.

- service account and clusterrole

```
kubectl create sa gpu-manager -n kube-system
kubectl create clusterrolebinding gpu-manager-role --clusterrole=cluster-admin --serviceaccount=kube-system:gpu-manager
```

- label node with `nvidia-device-enable=enable`

```
kubectl label node <node> nvidia-device-enable=enable
```

- submit daemonset yaml

```
kubectl create -f gpu-manager.yaml
```

## Pod template example

There is nothing special to submit a Pod except the description of GPU resource is no longer 1
. The GPU
resources are described as that 100 `tencent.com/vcuda-core` for 1 GPU and N `tencent.com/vcuda-memory` for GPU memory (1 tencent.com/vcuda-memory means 256Mi
GPU memory). And because of the limitation of extend resource validation of Kubernetes, to support
GPU utilization limitation, you should add `tencent.com/vcuda-core-limit: XX` in the annotation
 field of a Pod.
 
 **Notice: the value of `tencent.com/vcuda-core` is either the multiple of 100 or any value
smaller than 100.For example, 100, 200 or 20 is valid value but 150 or 250 is invalid**

- Submit a Pod with 0.3 GPU utilization and 7680MiB GPU memory with 0.5 GPU utilization limit

```
apiVersion: v1
kind: Pod
metadata:
  name: vcuda
  annotations:
    tencent.com/vcuda-core-limit: 50
spec:
  restartPolicy: Never
  containers:
  - image: <test-image>
    name: nvidia
    command:
    - /usr/local/nvidia/bin/nvidia-smi
    - pmon
    - -d
    - 10
    resources:
      requests:
        tencent.com/vcuda-core: 50
        tencent.com/vcuda-memory: 30
      limits:
        tencent.com/vcuda-core: 50
        tencent.com/vcuda-memory: 30
```

- Submit a Pod with 2 GPU card

```
apiVersion: v1
kind: Pod
metadata:
  name: vcuda
spec:
  restartPolicy: Never
  containers:
  - image: <test-image>
    name: nvidia
    command:
    - /usr/local/nvidia/bin/nvidia-smi
    - pmon
    - -d
    - 10
    resources:
      requests:
        tencent.com/vcuda-core: 200
        tencent.com/vcuda-memory: 60
      limits:
        tencent.com/vcuda-core: 200
        tencent.com/vcuda-memory: 60
```

## Contributors

[Rui Zhang](mailto:fixed-term.rui.zhang7@cn.bosch.com)

[Lilly Wu](mailto:lilly.wu@cn.bosch.com)


## 3rd Party Licenses <a name="3rd-party-licenses"></a>

|Name       | License       | Type        |
| ----------| ------------- | ----------- |
| [tkestack/GPU-manager](https://github.com/tkestack/gpu-manager) | [Apache License 2.0](https://github.com/tkestack/gpu-manager/blob/master/LICENSE) | Source Code |


## License

[![License: BIOSL v4](http://bios.intranet.bosch.com/bioslv4-badge.svg)](#license)

## FAQ

If you have some questions about this project, you can first refer to [FAQ](./docs/faq.md) to find a solution.
