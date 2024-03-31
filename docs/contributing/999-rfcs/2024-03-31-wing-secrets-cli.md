---
title: "#6105 Wing Secrets CLI"
description: Creating Wing secrets from the CLI
---

# Wing Secrets CLI
- **Author(s)**: @hasanaburayyan
- **Submission Date**: 2024-03-31
- **Stage**: Draft

Creating secrets through the Wing CLI.

## Background

TODO

## Sketch

Running the blow command will result in an interactive prompt where you can input the Slack credentials:

```bash
wing secrets main.w --create

2 secrets found in main.w

Enter the value for SLACK_SIGNING_SECRET: ********
Enter the value for SLACK_BOT_TOKEN: ********

Secrets saved to .env file
```

This results in a `.env` file being created with the secrets stored in it.

### AWS secret configuration

Just like the local configuration, the AWS configuration is done in the same way, but with the `-t tf-aws` flag:

```bash
wing secrets main.w --create -t tf-aws

2 secrets found in main.w

Enter the value for SLACK_SIGNING_SECRET: ********
Enter the value for SLACK_BOT_TOKEN: ********

Secrets saved to AWS Secrets Manager
```

This will save the secrets to AWS Secrets Manager.

## Implementation Ideas

Should be able to do a compile and gather all the secret resources, then in my initial thoughts these are passed to the platform in a new hook `configureSecrets(appSecrets: cloud.Secret[]): string` which returns a success/error message

