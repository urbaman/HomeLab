# Kubescape for observability and security checks

## Install locally

```bash
curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | /bin/bash
```

It will install the kubescape command and run the first scan

```bash
Installing Kubescape...
######################################################################## 100.0%
Finished Installation.

Remember to add the Kubescape CLI to your path with:
$ export PATH=$PATH:/home/ubuntu/.kubescape/bin

Executing Kubescape.
✅  Initialized scanner
✅  Loaded policies
✅  Loaded exceptions
✅  Loaded account configurations
✅  Accessed Kubernetes objects
✅  Collected RBAC resources
Control: C-0015 100% |█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| (62/62, 7 it/s)
✅  Done scanning. Cluster: kubernetes-admin-kubernetes
✅  Done aggregating results

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Controls: 62 (Failed: 39, Passed: 19, Action Required: 4)
Failed Resources by Severity: Critical — 0, High — 280, Medium — 465, Low — 181

┌──────────┬─────────────────────────────────────────────────┬──────────────────┬───────────────┬────────────────────┐
│ SEVERITY │                  CONTROL NAME                   │ FAILED RESOURCES │ ALL RESOURCES │ % COMPLIANCE-SCORE │
├──────────┼─────────────────────────────────────────────────┼──────────────────┼───────────────┼────────────────────┤
│ Critical │ Disable anonymous access to Kubelet service     │        0         │       0       │ Action Required *  │
│ Critical │ Enforce Kubelet client TLS authentication       │        0         │       0       │ Action Required *  │
│ High     │ Forbidden Container Registries                  │        0         │      78       │ Action Required ** │
│ High     │ Resources memory limit and request              │        54        │      78       │        31%         │
│ High     │ Resource limits                                 │        55        │      78       │        29%         │
│ High     │ Applications credentials in configuration files │        6         │      193      │        97%         │
│ High     │ List Kubernetes secrets                         │        22        │      135      │        84%         │
│ High     │ Host PID/IPC privileges                         │        5         │      78       │        94%         │
│ High     │ HostNetwork access                              │        7         │      78       │        91%         │
│ High     │ Writable hostPath mount                         │        24        │      78       │        69%         │
│ High     │ Insecure capabilities                           │        4         │      78       │        95%         │
│ High     │ HostPath mount                                  │        27        │      78       │        65%         │
│ High     │ Resources CPU limit and request                 │        55        │      78       │        29%         │
│ High     │ Privileged container                            │        21        │      78       │        73%         │
│ Medium   │ Exec into container                             │        3         │      135      │        98%         │
│ Medium   │ Data Destruction                                │        13        │      135      │        90%         │
│ Medium   │ Non-root containers                             │        49        │      78       │        37%         │
│ Medium   │ Allow privilege escalation                      │        47        │      78       │        40%         │
│ Medium   │ Ingress and Egress blocked                      │        66        │      79       │        16%         │
│ Medium   │ Delete Kubernetes events                        │        5         │      135      │        96%         │
│ Medium   │ Automatic mapping of service account            │        69        │      178      │        61%         │
│ Medium   │ Cluster-admin binding                           │        3         │      135      │        98%         │
│ Medium   │ CoreDNS poisoning                               │        6         │      135      │        96%         │
│ Medium   │ Validate admission controller (mutating)        │        4         │       4       │         0%         │
│ Medium   │ Access container service account                │        62        │      107      │        42%         │
│ Medium   │ Cluster internal networking                     │        16        │      21       │        24%         │
│ Medium   │ Linux hardening                                 │        45        │      78       │        42%         │
│ Medium   │ Configured liveness probe                       │        34        │      78       │        56%         │
│ Medium   │ Portforwarding privileges                       │        3         │      135      │        98%         │
│ Medium   │ No impersonation                                │        4         │      135      │        97%         │
│ Medium   │ Secret/ETCD encryption enabled                  │        1         │       1       │         0%         │
│ Medium   │ Audit logs enabled                              │        1         │       1       │         0%         │
│ Medium   │ Images from allowed registry                    │        0         │      78       │ Action Required ** │
│ Medium   │ CVE-2022-0492-cgroups-container-escape          │        29        │      78       │        63%         │
│ Medium   │ Anonymous access enabled                        │        5         │      130      │        96%         │
│ Low      │ Immutable container filesystem                  │        53        │      78       │        32%         │
│ Low      │ Configured readiness probe                      │        36        │      78       │        54%         │
│ Low      │ Validate admission controller (validating)      │        4         │       4       │         0%         │
│ Low      │ Network mapping                                 │        16        │      21       │        24%         │
│ Low      │ PSP enabled                                     │        1         │       1       │         0%         │
│ Low      │ Image pull policy on latest tag                 │        1         │      78       │        99%         │
│ Low      │ Label usage for resources                       │        41        │      78       │        47%         │
│ Low      │ K8s common labels usage                         │        29        │      78       │        63%         │
├──────────┼─────────────────────────────────────────────────┼──────────────────┼───────────────┼────────────────────┤
│          │                RESOURCE SUMMARY                 │       220        │      613      │       66.56%       │
└──────────┴─────────────────────────────────────────────────┴──────────────────┴───────────────┴────────────────────┘
FRAMEWORKS: AllControls (compliance: 66.09), NSA (compliance: 55.30), MITRE (compliance: 62.77)

* This control requires the host-scanner capability. To activate the host scanner capability, proceed with the installation of the kubescape operator chart found here: https://github.com/kubescape/helm-charts/tree/main/charts/kubescape-cloud-operator
** Control configurations are empty

View results
────────────
Now, take your scan results to the next level with actionable insights on ARMO Platform.

Sign up for free here - https://cloud.armosec.io/account/sign-up?customerGUID=5e4077e5-6f1a-4d13-9c7c-a8feb09861ec&invitationToken=Bg5APNBTXz9RILfCKawoCAVEe0fOwMN71k1kKJtEtYRyM1FmeokABoPKSxFzw8eJO0i1nvbtt2Y63aftVg1rgMhRIlU0xgYhqI8ygXND3s3dLyg9SZUN8ldymJAaTDmY&utm_medium=createaccount&utm_source=ARMOgithub

ℹ️   Run with '--verbose'/'-v' flag for detailed resources view
```

Add the kubescape PATH at the end your .bashrc file.

```bash
vi .bashrc
```

```bash
export PATH=$PATH:/home/ubuntu/.kubescape/bin
```

## Install in the cluster

Go to [https://cloud.armosec.io/dashboard](https://cloud.armosec.io/dashboard) and register, the add a cluster and run the proposed command on a control plane, adding the `--set kubescape.serviceMonitor.enabled=true` values, then verify on the web console.

Go to the webgui console to view the checks
