# Architecture notes

Read bottom-up: an upper layer fails when a lower one fails, never the other way round.

1. **Host.** One Ubuntu machine, one HDD. Internet through an office captive portal. Runs HAProxy (HTTPS entry),
   a watchdog, the daily backup job and the GitOps checkout.
2. **Virtualization.** KVM/libvirt on a private /24 network. One control-plane VM and two workers are active;
   two extra control-plane VMs are kept off for lack of IOPS.
3. **Node OS.** Talos Linux. Configured only through its API with `talosctl`. No shell on nodes.
4. **Kubernetes.** API reached through a Talos-managed VIP. Storage is `local-path`, so stateful pods are pinned
   to the node holding their data. A descheduler rebalances stateless pods.
5. **GitOps.** Argo CD app-of-apps with automated sync and self-heal. Manual `kubectl` edits are reverted.
   Secrets are SealedSecrets.
6. **Platform apps.** Traefik + cert-manager (ingress, internal TLS), Keycloak (SSO, database on managed
   PostgreSQL over TLS verify-full), kube-prometheus-stack, Argo Rollouts, Headlamp.
7. **Outside the cluster.** CI on GitHub Actions and on two Compute Engine VMs (Jenkins, GitLab CE plus runner);
   UIs reachable only through SSH tunnels. Encrypted offsite backups on Backblaze B2 with 30-day lifecycle.
   Heartbeats to healthchecks.io from the host and both VMs.

## Failure drills (questions I use to check my own understanding)

- The captive portal expires mid-day: what stops, what keeps working?
- Worker 1 dies: which pods cannot move, and why?
- Someone edits the Keycloak database host with `kubectl edit`: what happens five minutes later?
- The laptop holding the backup decryption key is lost: what is the impact?
