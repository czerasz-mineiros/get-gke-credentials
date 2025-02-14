<!--
Copyright 2020 Google LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->
# get-gke-credentials

This action configures authentication to a [GKE cluster][gke] via a `kubeconfig` file that can be used with `kubectl` or other methods of interacting with the cluster.

Authentication is performed by generating a [short-lived token][token] (default behaviour) or via the [GCP auth plugin][gcp-auth-plugin] present in `kubectl` which uses the service account keyfile path in [GOOGLE_APPLICATION_CREDENTIALS][gcp-gcloud-auth].

## Prerequisites

This action requires:

- Google Cloud credentials that are authorized to view a GKE cluster. See the Authorization section below for more information.

- [Create a GKE cluster](https://cloud.google.com/kubernetes-engine/docs/quickstart?_ga=2.267842766.1374248275.1591025444-475066991.1589991158)

## Usage

```yaml
jobs:
  job_id:
    permissions:
      contents: 'read'
      id-token: 'write'

    steps:
    - id: 'auth'
      uses: 'google-github-actions/auth@v0'
      with:
        workload_identity_provider: 'projects/123456789/locations/global/workloadIdentityPools/my-pool/providers/my-provider'
        service_account: 'my-service-account@my-project.iam.gserviceaccount.com'

    - id: 'get-credentials'
      uses: 'google-github-actions/get-gke-credentials@v0'
      with:
        cluster_name: 'my-cluster'
        location: 'us-central1-a'

    # The KUBECONFIG env var is automatically exported and picked up by kubectl.
    - id: 'get-pods'
      run: 'kubectl get pods'
```

## Inputs

-   `cluster_name` - (Required) Name of the cluster for which to get
    credentials. If specified as a full resource name (e.g.
    "projects/p/locations/l/clusters/c"), then then "project_id" and "location"
    inputs are optional. If only specified as the name (e.g. "my-cluster"), then
    the "project_id" and "location" inputs may be required.

-   `location` - (Optional) Location (e.g. region or zone) in which the cluster
    resides. This value is required unless you specify "cluster_name" as a full
    resource name.

-   `project_id` - (Optional) Project ID where the cluster is deployed. If
    provided, this will override the project configured by previous steps or
    environment variables. If not provided, the project will be inferred,
    best-effort.

-   `use_auth_provider` - (Optional) If true, use the Google Cloud auth plugin in
    kubectl instead of a short-lived access token. The default value is false.

-   `use_internal_ip` - (Optional) If true, use the internal IP address for the
    cluster endpoint. This is mostly used with private GKE clusters. The default
    value is false.

-   `credentials` - (**DEPRECATED**) This input is deprecated. See [auth
    section](#via-google-github-actionsauth) for more details. Service account
    key to use for authentication. This should be the JSON formatted private key
    which can be exported from the Cloud Console. The value can be raw or
    base64-encoded.

## Outputs

- Exports env var `KUBECONFIG` which is set to the generated `kubeconfig` file path.

## Authorization

There are a few ways to authenticate this action. A service account will be needed
with **at least** the following roles:

- Kubernetes Engine Cluster Viewer (`roles/container.clusterViewer`):
  - Get and list access to GKE Clusters.
`

### Via google-github-actions/auth

Use [google-github-actions/auth](https://github.com/google-github-actions/auth) to authenticate the action. You can use [Workload Identity Federation][wif] or traditional [Service Account Key JSON][sa] authentication.
by specifying the `credentials` input. This Action supports both the recommended [Workload Identity Federation][wif] based authentication and the traditional [Service Account Key JSON][sa] based auth.

See [usage](https://github.com/google-github-actions/auth#usage) for more details.

#### Authenticating via Workload Identity Federation

```yaml
jobs:
  job_id:
    permissions:
      contents: 'read'
      id-token: 'write'

    steps:
    - id: 'auth'
      uses: 'google-github-actions/auth@v0'
      with:
        workload_identity_provider: 'projects/123456789/locations/global/workloadIdentityPools/my-pool/providers/my-provider'
        service_account: 'my-service-account@my-project.iam.gserviceaccount.com'

    - id: 'get-credentials'
      uses: 'google-github-actions/get-gke-credentials@v0'
      with:
        cluster_name: 'my-cluster'
        location: 'us-central1-a'
```

#### Authenticating via Service Account Key JSON

```yaml
jobs:
  job_id:
    steps:
    - id: 'auth'
      uses: 'google-github-actions/auth@v0'
      with:
        credentials_json: '${{ secrets.gcp_credentials }}'

    - id: 'get-credentials'
      uses: 'google-github-actions/get-gke-credentials@v0'
      with:
        cluster_name: 'my-cluster'
        location: 'us-central1-a'
```

### Via Application Default Credentials

If you are hosting your own runners, **and** those runners are on Google Cloud,
you can leverage the Application Default Credentials of the instance. This will
authenticate requests as the service account attached to the instance. **This
only works using a custom runner hosted on GCP.**

```yaml
jobs:
  job_id:
    steps:
    - id: 'get-credentials'
      uses: 'google-github-actions/get-gke-credentials@v0'
      with:
        cluster_name: 'my-cluster'
        location: 'us-central1-a'
```

The action will automatically detect and use the Application Default
Credentials.

## Versioning

We recommend pinning to the latest available major version:

```yaml
- uses: 'google-github-actions/get-gke-credentials@v0'
```

While this action attempts to follow semantic versioning, but we're ultimately
human and sometimes make mistakes. To prevent accidental breaking changes, you
can also pin to a specific version:

```yaml
- uses: 'google-github-actions/get-gke-credentials@v0.1.0'
```

However, you will not get automatic security updates or new features without
explicitly updating your version number. Note that we only publish `MAJOR` and
`MAJOR.MINOR.PATCH` versions. There is **not** a floating alias for
`MAJOR.MINOR`.

[gke]: https://cloud.google.com/kubernetes-engine
[gcp-auth-plugin]: https://github.com/kubernetes/client-go/tree/master/plugin/pkg/client/auth/gcp
[gcp-gcloud-auth]: https://cloud.google.com/kubernetes-engine/docs/how-to/api-server-authentication#using-gcloud-config
[token]: https://kubernetes.io/docs/reference/access-authn-authz/authentication/#openid-connect-tokens
[sm]: https://cloud.google.com/secret-manager
[sa]: https://cloud.google.com/iam/docs/creating-managing-service-accounts
[wif]: https://cloud.google.com/iam/docs/workload-identity-federation
[gh-runners]: https://help.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners
[gh-secret]: https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets
[setup-gcloud]: ../setup-gcloud
