# Terraform State Locking: S3 Lock Files vs. DynamoDB

## 1. What is Terraform state?

Terraform state is a file that records the resources Terraform manages and the information it needs to compare the real infrastructure with the desired configuration.

When using a remote backend, teams can share state instead of keeping it only on one engineer's computer.

## 2. Why is state locking important?

Imagine two engineers run `terraform apply` at nearly the same time against the same state file. Both operations could attempt to change the infrastructure based on conflicting state information.

State locking helps prevent concurrent Terraform operations from modifying the same state simultaneously.

**Key benefit:** Locking reduces the risk of conflicting changes to shared infrastructure.

## 3. S3 native lock files

Terraform supports native state locking through Amazon S3 when the S3 backend is configured with:

```hcl
terraform {
  backend "s3" {
    bucket       = "YOUR-STATE-BUCKET"
    key          = "environments/dev/terraform.tfstate"
    region       = "us-east-1"
    encrypt      = true
    use_lockfile = true
  }
}
```

Replace `YOUR-STATE-BUCKET` with the actual backend bucket name when using this example in a real configuration.

### How it works

- Terraform stores the state file in the configured S3 bucket.
- With `use_lockfile = true`, Terraform uses a separate `.tflock` object to coordinate state access.
- Terraform normally acquires the lock before a state-changing operation and releases it when the operation finishes.
- S3 permissions must allow the required state and lock-file operations.

### Advantages

- Uses S3 without requiring a separate DynamoDB table for locking.
- Reduces the number of AWS services that must be configured.
- Supports shared state workflows for teams.

## 4. DynamoDB state locking

Historically, Terraform's S3 backend supported state locking through an Amazon DynamoDB table. Teams configured the table and supplied its name in the backend configuration.

### How it worked

- Terraform stored state in S3.
- DynamoDB coordinated access to the shared state.
- Terraform used the table to manage lock information and prevent conflicting operations.

### Limitations

- It requires a separate DynamoDB table and associated permissions.
- It adds another AWS resource to configure and maintain.
- DynamoDB-based locking is deprecated in Terraform's S3 backend documentation.

DynamoDB locking remains useful to understand when maintaining older Terraform projects, but new S3 backend configurations should generally use native S3 lock files.

## 5. Comparison

| Feature | S3 native lock file | DynamoDB locking |
|---|---|---|
| State storage | Amazon S3 | Amazon S3 |
| Lock coordination | S3 `.tflock` object | DynamoDB table |
| Extra DynamoDB table | Not required | Required |
| Current guidance | Preferred for new S3 backends | Deprecated |
| Learning value | Current recommended approach | Important for legacy projects |

## 6. How this project uses locking

This project configures its S3 backend with:

```hcl
use_lockfile = true
```

This means Terraform uses S3 native lock-file support. The project does not need to add DynamoDB solely for state locking.

Do not change the existing backend just to demonstrate the older method. Learn the legacy configuration conceptually unless you have a separate, safe test environment.

## 7. Important operational practices

- Never manually delete a lock file while Terraform may still be running.
- If Terraform reports that state is locked, first check whether another operation is active.
- Use `terraform force-unlock` only when you have confirmed that the lock belongs to an operation that is no longer running, and follow the command's safety prompts.
- Enable S3 bucket versioning to help recover from accidental state changes.
- Restrict access to the state bucket because Terraform state may contain sensitive information.
- Do not commit state files, credentials, or sensitive variable files to Git.

## 8. Key interview takeaway

**Question:** What is the difference between S3 native locking and DynamoDB locking in Terraform?

**Answer:** Both help prevent concurrent operations against shared Terraform state. With the S3 backend, native locking uses an S3 `.tflock` object, while the older DynamoDB approach uses a separate DynamoDB table. S3 native locking is the preferred approach for new configurations, and DynamoDB-based locking is deprecated.

## References

- [Terraform S3 backend documentation](https://developer.hashicorp.com/terraform/language/backend/s3)
- [Terraform state locking documentation](https://developer.hashicorp.com/terraform/language/state/locking)
