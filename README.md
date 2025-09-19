Terraform module for a Postgres CloudSQL Instance in GCP

# Upgrade guide from v2.3.8 to v3.0.0

This upgrade requires google provider [v4.74.0](https://github.com/hashicorp/terraform-provider-google/releases/tag/v4.74.0) or above which introduces the `edition` feature to `google_sql_database_instance` resources. Plan & Apply the google provider version upgrade first **before** upgrading this module version to `v3.0.0`.

### Terraform >= 1.3.0 is required as "optional" variables are used.

### Switched to using random_password to generate default passwords.

With the 3.0.0 release, the `random_id` resource used to generate default passwords has been replaced with `random_password` resource. This improves the default behavior by generating stronger passwords as defaults. To continue using the previously generated password and prevent updates to the `google_sql_user` resources, specify the old password via `user_password` variable or `additional_users.password`. It is recommended to store this value in [Secret Manager](https://cloud.google.com/secret-manager/docs/creating-and-accessing-secrets#secretmanager-create-secret-gcloud) as opposed to passing it in via plain text.

```diff
+ data "google_secret_manager_secret_version" "user_password" {
+  secret_id = "pg-user-pass"
+ }

module "pg" {
  source  = "airasia/postgres_db/google"
- version = "2.3.8"
+ version = "3.0.0"

+ user_password = data.google_secret_manager_secret_version.user_password.secret_data
}
```

# Upgrade guide from v2.1.0 to v2.1.1

This upgrade requires google provider [v3.44.0](https://github.com/hashicorp/terraform-provider-google/releases/tag/v3.44.0) or above which introduces the `deletion_protection` feature to `google_sql_database_instance` resources. Plan & Apply the google provider version upgrade first **before** upgrading this module version to `v2.1.1`.

# Upgrade guide from v2.0.4 to v2.1.0

First make sure you've planned & applied `v2.0.4`. Then, upon upgrading from `v2.0.4` to `v2.1.0`, you may (or may not) see a plan that destroys & creates an equal number of `google_project_iam_member` resources. It is OK to apply these changes as it will only change the data-structure of these resources [from an array to a hashmap](https://github.com/airasia/terraform-google-external_access/wiki/The-problem-of-%22shifting-all-items%22-in-an-array). Note that, after you plan & apply these changes, you may (or may not) get a **"Provider produced inconsistent result after apply"** error. Just re-plan and re-apply and that would resolve the error.
