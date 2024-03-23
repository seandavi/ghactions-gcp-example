## ghactions to gcp auth example repo

See <https://github.com/google-github-actions/auth>

### setup

```
# TODO: replace ${PROJECT_ID} with your value below.
export PROJECT_ID=ghactions-46891
export GITHUB_ORG=seandavi
export GITHUB_REPO=ghactions-gcp-example

gcloud iam workload-identity-pools create "github" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --display-name="GitHub Actions Pool"

gcloud iam workload-identity-pools describe "github" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --format="value(name)"


# TODO: replace ${PROJECT_ID} and ${GITHUB_ORG} with your values below.

gcloud iam workload-identity-pools providers create-oidc "${GITHUB_REPO}" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="github" \
  --display-name="My GitHub repo Provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository,attribute.repository_owner=assertion.repository_owner" \
  --attribute-condition="assertion.repository_owner == '${GITHUB_ORG}'" \
  --issuer-uri="https://token.actions.githubusercontent.com"

# TODO: replace ${PROJECT_ID} with your value below.

gcloud iam workload-identity-pools providers describe "${GITHUB_REPO}" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="github" \
  --format="value(name)"
```

Use this value as the workload_identity_provider value in the GitHub Actions YAML:

```
- uses: 'google-github-actions/auth@v2'
  with:
    project_id: 'my-project'
    workload_identity_provider: '...' # "projects/123456789/locations/global/workloadIdentityPools/github/providers/my-repo"


```
As needed, allow authentications from the Workload Identity Pool to Google Cloud resources. These can be any Google Cloud resources that support federated ID tokens, and it can be done after the GitHub Action is configured.

The following example shows granting access from a GitHub Action in a specific repository a secret in Google Secret Manager.
```
# TODO: replace ${PROJECT_ID}, ${WORKLOAD_IDENTITY_POOL_ID}, and ${REPO}
# with your values below.
#
# ${REPO} is the full repo name including the parent GitHub organization,
# such as "my-org/my-repo".
#
# ${WORKLOAD_IDENTITY_POOL_ID} is the full pool id, such as
# "projects/123456789/locations/global/workloadIdentityPools/github".

gcloud projects add-iam-policy-binding ghactions-46891 \
  --member="principalSet://iam.googleapis.com/projects/390395656114/locations/global/workloadIdentityPools/github/attribute.repository/ghactions-gcp-example" \
  --role="roles/storage.Admin"

# or with repository owner (see above for the "conditions")
gcloud projects add-iam-policy-binding ghactions-46891 \
  --member="principalSet://iam.googleapis.com/projects/390395656114/locations/global/workloadIdentityPools/github/attribute.repository_owner/seandavi" \
  --role="roles/storage.admin"
```


