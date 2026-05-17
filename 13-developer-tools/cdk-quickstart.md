# CDK Quickstart

This is a quick-start link. For the full CDK guide see **[../07-devops-iac/cdk.md](../07-devops-iac/cdk.md)**.

## 60-second cheat sheet

```bash
# Install
npm install -g aws-cdk

# Initialize project
mkdir my-stack && cd my-stack
cdk init app --language typescript

# Bootstrap (first time per region/account)
cdk bootstrap aws://123456789012/ap-south-1

# Synth + diff + deploy
cdk synth
cdk diff
cdk deploy
```

### Minimal stack

`lib/my-stack.ts`:
```ts
import { Stack, StackProps } from "aws-cdk-lib";
import { Construct } from "constructs";
import * as s3 from "aws-cdk-lib/aws-s3";

export class MyStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);
    new s3.Bucket(this, "Bucket", { versioned: true });
  }
}
```

`bin/my-stack.ts`:
```ts
#!/usr/bin/env node
import { App } from "aws-cdk-lib";
import { MyStack } from "../lib/my-stack";

const app = new App();
new MyStack(app, "MyStack", { env: { region: "ap-south-1" } });
```

```bash
cdk deploy
```

For the deeper guide — constructs, patterns, pipelines, testing — read **[CDK](../07-devops-iac/cdk.md)**.
