# monitoring

Deploys `kube-prometheus-stack` (Prometheus + Grafana) into the `monitoring` namespace.

This is an [app-of-apps](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/#app-of-apps-pattern) child: `application.yaml` is itself an ArgoCD `Application` resource (namespace `argocd`), discovered and applied by `root-app`'s recursive directory scan of `apps/`. Once applied, ArgoCD's controller picks it up as an independent Application — no changes to `k8s-cluster`'s Ansible were needed.

- **Chart:** `prometheus-community/kube-prometheus-stack` `88.3.0`, sourced directly from the Helm repo (no vendored manifests).
- **Sizing:** trimmed for a 1 master + 2× t3.medium worker lab cluster with no dynamic storage provisioner — Alertmanager and the kubeadm-only component monitors (`kubeControllerManager`/`kubeScheduler`/`kubeEtcd`/`kubeProxy`, which bind to `127.0.0.1` on kubeadm and would just show as permanently down) are disabled; Prometheus uses a 2Gi `emptyDir` with 24h retention instead of a PersistentVolume.
- **Access:** Grafana is exposed via the existing NGINX Ingress Controller using **path-based** routing (`grafana-ingress.yaml`, a plain manifest alongside `application.yaml` — the chart's own built-in Ingress is disabled since it requires a `host`, which would force a DNS/hosts-file workaround). No new NodePort. Reach it through a bastion SSH tunnel to NodePort 30080, then browse `http://localhost:30080/grafana/` — no hostname or hosts-file edit needed. `serve_from_sub_path`/`root_url` are set in the chart values so Grafana's assets resolve correctly under `/grafana/`. Prometheus itself has no external exposure; Grafana reaches it over the in-cluster Service DNS via the chart's auto-provisioned datasource.
- **Grafana admin password:**
  ```bash
  kubectl get secret -n monitoring monitoring-grafana -o jsonpath='{.data.admin-password}' | base64 -d; echo
  ```
