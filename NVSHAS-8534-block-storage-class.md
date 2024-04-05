## NVSHAS-8534 - To block the usage of specific storage classes

## Table of Contents

- [Section 1: Overview](#section-1-overview)
- [Section 2: Design](#section-2-design)

## Section 1: Background

Hello Team,

Can we block the usage of specific storage classes with NeuVector? Maybe with Admission Controls?

There are some predefined storage classes that users should not use; they need to be blocked.

Please let me know if any information is needed.

Thanks.

TODO: mention Kubewarden implementation

## Section 2: Kubernetes resources

TODO: explain a bit about how a workload uses storageclass.

- illustrate the workload and PVC
- (1) use PVC
- (2) use PVC template (Statefulset)
  (show real yaml)

TODO: mentioned the resource creation, use the diagram in the powerpoint..
TODO: show the use case... create deployment first, check the status in pending and then create PVC.. PV. show the screen shot

<p align="left">
<img src="./materials/pvc-3.png" width="80%">
</p>

## Section 3: Admission Controller behavior

TODO: mention the validation call from k8s

## Section 4: Proposal plan

Add a new criteria, "certain storage classname usage"
Describe the plan... remember the validation request which has the storageclass criteria

show the case

- (1) when workload request comes in, the PVC information is ready
- (2) when workload request comes in, the PVC information is NOT ready

including the UI change

## Section 5: References

<p align="left">
<img src="./materials/pvc-1.png" width="80%">
</p>

<p align="left">
<img src="./materials/pvc-2.png" width="80%">
</p>
