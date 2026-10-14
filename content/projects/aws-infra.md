+++
title = "AWS Infra"
date = "2026-09-10"
draft = false
cover = "/images/cover/aws-infra.png"
#tags = [""]
showFullContent = false
hideComments = true

#for SEO
#keywords = [""]
#description = ""
+++
[Github](https://github.com/asdiAdi/aws-infra)

I extracted my static-site AWS setup into a reusable CDK library with a small CLI, so new sites are one stack + one workflow.
<!--more-->

{{< details summary="TLDR" >}}
- Reusable AWS CDK constructs published as `npm install @asdi/aws-infra` with `asdi` CLI (`init`, `add`, `param`).
- `StaticSite`: Route53 + ACM + private S3 (OAC) + CloudFront + scoped S3 sync / invalidate policy. Optional SPA fallback.
- `GithubDeploy`: GitHub OIDC federated role with branch or environment trust, plus optional SSM read prefixes.
- Successor to [Static Site Infrastructure](/projects/static-site-infrastructure): same architecture, generalized.
- Technologies used: [AWS CDK](https://docs.aws.amazon.com/cdk/v2/guide/home.html), [Route53](https://aws.amazon.com/route53/), [Certificate Manager](https://aws.amazon.com/certificate-manager/), [S3](https://aws.amazon.com/s3/), [CloudFront](https://aws.amazon.com/cloudfront/), [GitHub Actions](https://github.com/features/actions)
{{< /details >}}

This is the successor to [Static Site Infrastructure](/projects/static-site-infrastructure) (`cdk-static-sites`).
That project proved the Route53 + ACM + S3 + CloudFront pattern; `aws-infra` generalizes it into `StaticSite` / `StaticSiteStack` plus `GithubDeploy` / `GithubDeployStack`, with templates and workflows so each new site is just config.

## Quickstart

From your website project root:

```bash
npm install @asdi/aws-infra
asdi init
asdi add static-website
asdi add sync
npx cdk synth
npx cdk deploy
```

- `init` creates `cdk.json` (`{ "app": "npx tsx infra/index.ts" }`).
- `add static-website` copies the `infra/static-website.ts` starter composing both stacks.
- `add sync` copies `.github/workflows/sync.yml` (`s3 sync` + CloudFront invalidation via OIDC `vars.AWS_ROLE_TO_ASSUME` / `vars.AWS_REGION`).

Minimal stack:

```typescript
const site = new StaticSiteStack(app, "site", {
  env: { account: process.env.CDK_DEFAULT_ACCOUNT ?? "", region: "us-east-1" },
  secondLevelDomain: "example.com",
  subDomain: "www",
});

new GithubDeployStack(app, "deploy", {
  env: { account: process.env.CDK_DEFAULT_ACCOUNT ?? "", region: "us-east-1" },
  roleName: "MyAppGithubDeploy",
  github: { owner: "my-org", ownerId: "123456", repo: "my-app", repoId: "789012", branch: "main" },
  managedPolicies: [site.staticSite.managedPolicy],
});
```

## Prerequisites

- Existing Route53 hosted zone (uses `HostedZone.fromLookup` at synth time).
- `us-east-1` when auto-creating the ACM cert (CloudFront requirement), or bring your own `hostedZone` + `certificate`.
- Existing GitHub OIDC provider plus `GithubCdkDeploy` managed policy for the deploy role.
- Optional `ssmParameterPrefixes: ["/myapp/prod"]` for SSM reads; `asdi param push/pull/delete` syncs multi-prefix `.env` files with `# path:` sections.

Full codebase and CLI reference in [GitHub](https://github.com/asdiAdi/aws-infra).
