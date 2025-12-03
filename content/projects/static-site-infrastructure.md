+++
title = "Static Site Infrastructure"
date = "2025-03-13"
draft = false
cover = "/images/cover/static-site-infrastructure.png"
#tags = [""]
showFullContent = false
hideComments = true

#for SEO
#keywords = [""]
#description = ""
+++
[Github](https://github.com/asdiAdi/cdk-static-sites)

I got tired managing static sites on multiple PaaS, so I made one myself.
<!--more-->

{{< details summary="TLDR" >}}
- Unified platform for all my static websites.
- Technologies used: [Route53](https://aws.amazon.com/route53/), [Certificate Manager](https://aws.amazon.com/certificate-manager/), [S3 Bucket](https://aws.amazon.com/s3/), [Cloudfront](https://aws.amazon.com/cloudfront/), [GitHub Actions](https://github.com/features/actions)
{{< /details >}}

Netlify, Vercel, Render, these services are convenient for deploying websites quickly especially static websites.
They are great but at the end of the day they are still using AWS.
If you ever go beyond their limits, the costs will always be higher than just deploying it on AWS directly.
So I thought, why not cut the middleman? 
There are a lot of benefits for me:
- Complete control of the environment.
- A single unified platform for all my static websites.
- Route53 DNS Arecords.
- I can cache the websites using Cloudfront.
- If it gets attacked(to which I highly doubt), costs should be significantly less.

Ok enough of my reasons, I'll explain how it works.
The complete codebase is in [GitHub](https://github.com/asdiAdi/cdk-static-sites) if you want to try it out.
The entire structure is abstracted inside the file `static-site-construct.ts`
When making a new website all I have to do is provide the domain and subdomain name like this.
```typescript
export class PortfolioStack extends cdk.Stack {
    constructor(scope: Construct, id: string, props?: cdk.StackProps) {
        super(scope, id, props);
        
        // just call the construct
        new StaticSiteConstruct(this, "Portfolio", {
                secondLevelDomain: "carladi.com", //domain
                subDomain: "portfolio", //subdomain
        });
    }
}
```
Add the stack inside `cdk-static-sites.ts`
```typescript
new PortfolioStack(app, "PortfolioStack", { env });
```
Then just hit `cdk synth`(for good measure) and `cdk deploy` in the terminal and you're good to go.

## But how does it really work?
This section will explain `static-site-construct.ts`.

The construct consists of 4 AWS services:
- Route53 for SSL validation and arecord
- Certificate Manager for SSL
- Cloudfront for cache
- S3 bucket for storage of the entire static website

In order, this is how it should work:
1. **Hosted Zone Lookup**. Route 53 retrieves the hosted zone associated with the domain.
    ``` typescript
    const hostedZone = route53.HostedZone.fromLookup(
      this,
      `${constructId}-HostedZone`,
      { domainName: props.secondLevelDomain },
    );
    ```
2. **SSL Certificate Creation**. A certificate is generated and validated via DNS, allowing the website to use HTTPS.
    ```typescript
    const certificate = new acm.Certificate(
      this,
      `${constructId}-Certificate`,
      {
        domainName: domainName,
        validation: acm.CertificateValidation.fromDns(hostedZone),
      },
    );
    ```
3. **S3 Bucket Setup**. The S3 bucket stores and serves static files, with auto-deletion and versioning enabled.
    ```typescript
    const bucket = new s3.Bucket(this, `${constructId}-Bucket`, {
      bucketName: domainName,
      websiteIndexDocument: "index.html",
      removalPolicy: cdk.RemovalPolicy.DESTROY,
      autoDeleteObjects: true,
    });
    ```
4. **CloudFront Distribution**. CloudFront connects to the bucket and serves content through a secure HTTPS connection.
    ```typescript
    const distribution = new cloudfront.Distribution(
        this,
        `${constructId}-Distribution`,
        {
            defaultBehavior: {
                origin: cloudfront_origins.S3BucketOrigin.withOriginAccessControl(
                    bucket,
                    {},
                ),
                compress: true,
                allowedMethods: cloudfront.AllowedMethods.ALLOW_GET_HEAD_OPTIONS,
                viewerProtocolPolicy:
                cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
            },
            defaultRootObject: "index.html",
            domainNames: [domainName],
            certificate: certificate,
        },
    );
    ```
5. **Route 53 ARecord**. An ARecord associates the domain with the CloudFront distribution.
    ```typescript
    const alias = new route53_targets.CloudFrontTarget(distribution);
    const arecord = new route53.ARecord(this, `${constructId}-ARecord`, {
        zone: hostedZone,
        target: route53.RecordTarget.fromAlias(alias),
        recordName: props.subDomain,
        ttl: cdk.Duration.days(1),
    });
    ```

## What about CICD?
This is where GitHub actions shines.
I did not bother using the AWS Services for this namely, CodeBuild, CodePipeline, CodeCommit etc.
As I am uploading my code in GitHub, it is much easier to just use their services.
Plus it's mostly free :)

In a `.yml` file inside `/.github/workflows/` add these key parts:
1. **Permissions**. You need to setup [GitHub OIDC](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services) for AWS.
    ```yaml
    permissions:
      id-token: write
      contents: read
    ```
2. **Configure AWS Credentials**.
    The role is assumed using sts.amazonaws.com with the specified ARN.
    ```yaml
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          audience: "sts.amazonaws.com"
          aws-region: us-east-1
          role-to-assume: arn:aws:iam::${{ secrets.AWS_ACCOUNT_ID }}:role/github-actions
    ```
3. **Sync Files to S3**. The aws s3 sync command uploads the compiled files to the S3 bucket and deletes any outdated files.
    ```yaml
      - name: Deploy to S3
        run: aws s3 sync out s3://${{ secrets.AWS_BUCKET_NAME }} --delete
    ```
   
4. **Invalidate CloudFront Cache**. CloudFront cache is invalidated to ensure the latest version of the site is served immediately.
    ```yaml
      - name: Invalidate Cloudfront Cache
        run: aws cloudfront create-invalidation  --distribution-id ${{ secrets.AWS_DISTRIBUTION_ID }} --paths "/*"
    ```
Here's a sample [workflow](https://github.com/asdiAdi/test-static/blob/main/.github/workflows/deploy.yml).
And btw, CICD is not a requirement. These are just static sites, I can easily upload them to S3 manually for smaller projects.

## Conclusion
By doing all these, I get complete control of my websites, lower costs and a unified environment.
GitHub Actions simplifies CI/CD on my chosen projects.
It also means I am completely responsible for any fuckups like a huge aws bill :D