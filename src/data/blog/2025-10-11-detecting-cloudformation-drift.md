---
title: "Detecting Manual AWS Changes in CloudFormation Stacks"
pubDatetime: 2025-10-11
description: "How to detect and manage configuration drift in AWS CloudFormation stacks."
slug: detecting-manual-aws-changes-in-cloudformation-stacks
tags:
  - devops
draft: false
---

Manual infrastructure changes happen when someone "quickly fixes" something in production through the AWS Console. Someone scales up an RDS instance during an incident. Someone tweaks a security group rule to debug connectivity. Someone changes an environment variable to test a fix.

The change works, the immediate problem is solved, and everyone moves on. The next day, a routine deployment reverts those changes. Production breaks and nobody knows why.

CloudFormation's drift detection catches this by comparing your stack template against the actual resources running in AWS.

## How Drift Detection Works

```bash
aws cloudformation detect-stack-drift --stack-name prod-infrastructure
aws cloudformation describe-stack-resource-drifts --stack-name prod-infrastructure
```

The first command starts detection. The second retrieves results. For large stacks, detection takes a few seconds to a few minutes.

When drift is found, the output shows exactly what changed:

```json
{
  "StackResourceDrift": {
    "LogicalResourceId": "DatabaseInstance",
    "ResourceType": "AWS::RDS::DBInstance",
    "ExpectedProperties": "{\"DBInstanceClass\":\"db.t3.medium\"}",
    "ActualProperties": "{\"DBInstanceClass\":\"db.t3.large\"}",
    "PropertyDifferences": [
      {
        "PropertyPath": "/DBInstanceClass",
        "ExpectedValue": "db.t3.medium",
        "ActualValue": "db.t3.large",
        "DifferenceType": "NOT_EQUAL"
      }
    ],
    "StackResourceDriftStatus": "MODIFIED"
  }
}
```

The instance class changed from `db.t3.medium` to `db.t3.large`. The template says medium, AWS is running large.

## What to Do With Detected Drift

**Update the template to match reality** — the change was intentional and should be permanent. Update the CloudFormation template, commit it, redeploy.

**Revert the infrastructure** — the change was temporary or a mistake. Redeploy the stack and let CloudFormation restore the resource to what the template specifies.

The decision comes down to intent: a performance fix during an incident belongs in the template. A debug tweak that was never meant to stick belongs reverted.

## Limitations

Drift detection only catches changes to resources CloudFormation manages. Resources created outside CloudFormation entirely won't appear.

---

**TLDR:** Manual AWS changes silently diverge from your IaC. Drift detection surfaces the diff. Either update the template or redeploy to reconcile.

---

**References:**

- [Detect unmanaged configuration changes to stacks and resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html)
- [DetectStackDrift API Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_DetectStackDrift.html)
- [Detect drift on an entire CloudFormation stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/detect-drift-stack.html)
