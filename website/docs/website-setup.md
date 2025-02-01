# Website Setup

Migrating from GitHub Pages to Cloudflare Pages.

## Cloudflare

Account Home, Compute (Workers) then the Pages tab to connect Cloudflare to
GitHub jeffmccune/jeffmccune.com

## Google Organization

Setup a google organization so we have inexpensive access to a GKE autopilot
cluster, identity and access management, and workload identity token exchange.

Created a new browser profile for the task.  The overall process is:

1. Verify the domain.
2. Configure SMTP forwarding rules in Cloudflare.
3. Setup the domain admin cloud identity account.
4. Grant access to the primary google account I use.
5. Setup some Google security groups.
6. Create the GKE autopilot cluster.
7. Configure holos to use the autopilot cluster.

[Sign up for Cloud Identity Free](https://cloud.google.com/identity/docs/set-up-cloud-identity-admin#sign-up-for-cloud-identity-free)

Went through the wizard and setup `super-admin@jeffmccune.com`.  Two factor,
passkey in iCloud keychain.  Had to enable passkeys at https://admin.google.com
and sign into Chrome to get the passkey to save correctly.

The rest of the setup is done in a chrome profile logged in as super-admin.

Started Google Cloud setup by browsing to https://console.cloud.google.com

Worked through setting up logging and and security but didn't setup the
Hierarchy and access.

Created a project, jeffmccune-com and switched to gcloud for the rest.

## GKE Security Group

Browse to https://admin.google.com/ac/groups and create a group named `gke-security-groups`.  See [Set up your Google Groups](https://cloud.google.com/kubernetes-engine/docs/how-to/google-groups-rbac#setup-group)

## Kubernetes Cluster

```bash
PROJECT_ID="$(gcloud config get-value project)"
PROJECT_NUMBER="$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')"
ORG_DOMAIN="example.com"
```

Init to the super-admin account.  Fine for now.

```
gcloud container clusters create-auto autopilot \
  --release-channel=stable \
  --monitoring=SYSTEM \
  --logging=SYSTEM \
  --enable-master-global-access \
  --enable-master-authorized-networks \
  --master-authorized-networks=0.0.0.0/0 \
  --region=us-central1 \
  --security-group=gke-security-groups@$ORG_DOMAIN
```

Get the kubeconfig

```bash
mkdir -p ~/.holos
KUBECONFIG=${HOME}/.holos/kubeconfig.autopilot.${PROJECT_ID} \
  gcloud container clusters get-credentials autopilot --region=us-central1
```

Move to where `holos` expects it.

```bash
(cd ~/.holos && ln -sf kubeconfig.autopilot.${PROJECT_ID} kubeconfig.provisioner
```

We need to run some kubectl commands.

```bash
export KUBECONFIG="${HOME}/.holos/kubeconfig.provisioner"
```

Create the secrets namespace

```bash
kubectl create namespace secrets
```

Now we can script secrets with `holos`.  Workload identity setup and integration
with External Secrets operator is another topic.
