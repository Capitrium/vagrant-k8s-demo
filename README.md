# Vagrant-Env

A demo Vagrant environment with Prometheus, Node-Exporter, and Grafana installed on top of a single-node Kubernetes cluster.

Kubernetes was chosen as the orchestration tool given its popularity and the author's familiarity with both with K8s and the general ecosystem. In particular, the prometheus-operator was chosen to drasticly simplify the installation and configuration of Prometheus. The entire installation process reduces to installing Kubernetes with `kubeadm`, and applying some manifests with `kubectl`; the entire installation, including the Grafana dashboards, are provided declaratively. Combining Vagrant's shared ports with Kubernetes `NodePort` Services provides a simple way to expose services running in the guest cluster to the host.

All applications are available as soon as `vagrant up` has finished.

Prometheus is available at localhost:9090

Grafana is available at localhost:3000 with a test dashboard showing some basic Prometheus metrics.