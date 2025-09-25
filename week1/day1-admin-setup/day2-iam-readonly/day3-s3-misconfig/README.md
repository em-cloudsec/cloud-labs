# Day 3 – S3 Misconfiguration & Remediation

## Objective
- Understand how S3 misconfigurations can lead to public data exposure.
- Practice fixing bucket permissions and verifying restricted access.

## Steps Completed
1. Created an S3 bucket (`<earls-labs-bucket-01>`).
2. Uploaded test file `test.txt`.
3. Modified bucket policy to allow public reads (`"Principal": "*"`) → file accessible in browser.
4. Enabled **Block Public Access** to remediate.
5. Retested → Access denied as expected.

## Results
- ✅ File was accessible when bucket policy allowed public reads.
- ✅ Access was denied after re-enabling block public access.

### Misconfiguration Policy (JSON)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<earls-labs-bucket-01>/*"
    }
  ]
}

<Error>
  <Code>AccessDenied</Code>
  <Message>Access Denied</Message>
  <RequestId>RQKGHYG74808WH7B</RequestId>
  <HostId>r6bt4LmOX8rJ17wFwjC6UIB1SPkq3NG71RAmuLUU5GFoPu//O/MIla3X31/agENjSnaxmZMl6LE=</HostId>
</Error>
