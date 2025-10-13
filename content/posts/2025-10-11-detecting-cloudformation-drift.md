---
title: "Detecting Manual AWS Changes in CloudFormation Stacks"
date: 2025-10-11
dateUpdated: Last Modified
permalink: /posts/detecting-manual-aws-changes-in-cloudformation-stacks/
tags:
  - DevOps
layout: layouts/post.njk
---

Manual infrastructure changes happen when someone "quickly fixes" something in production through the AWS Console and forgets to update and re-run the CloudFormation code. Someone scales up an RDS instance during an incident. Someone tweaks a security group rule to debug connectivity. Someone changes an environment variable to test a fix.

The changes work, the immediate problem is solved, and the team forgets. The next day, a routine deployment reverts those undocumented changes. Production breaks, but nobody knows why.

CloudFormation's drift detection catches these manual changes by comparing your stack template definition against actual resources running in AWS.

## How Drift Detection Works

```bash
# Trigger drift detection
aws cloudformation detect-stack-drift --stack-name prod-infrastructure

# Get the results
aws cloudformation describe-stack-resource-drifts --stack-name prod-infrastructure
```

The first command starts the detection process. The second retrieves the results. For large stacks, detection takes a few seconds to a few minutes.

When CloudFormation finds drift, the output shows exactly what changed:

```json
{
  "StackResourceDrift": {
    "StackId": "arn:aws:cloudformation:...",
    "LogicalResourceId": "DatabaseInstance",
    "PhysicalResourceId": "prod-postgres-main",
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

Someone changed the instance class from `db.t3.medium` to `db.t3.large`. The code says medium, but AWS is running large.

## What I Do With Detected Drift

Two options when drift is detected:

**Update the code to match reality:** The change was intentional and should be permanent. I update my CloudFormation template to reflect the actual state, commit it, and redeploy.

**Revert the infrastructure:** The change was a temporary fix or a mistake. I redeploy the CloudFormation stack, which reverts the resource to what the template specifies.

I decide based on why the change happened:

- Performance fix during incident -> update code
- Debug tweak meant to be temporary -> revert infrastructure
- Configuration experiment that worked -> update code

## Limitations

Drift detection only catches changes to resources CloudFormation manages. If someone creates a new resource outside CloudFormation, drift detection won't find it.

---

**References:**

- [Detect unmanaged configuration changes to stacks and resources with drift detection - AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html)
- [DetectStackDrift - AWS CloudFormation API Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_DetectStackDrift.html)
- [Detect drift on an entire CloudFormation stack - AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/detect-drift-stack.html)
