---
title: "How to scan EC2 AMIs using Amazon Inspector"
url: "https://aws.amazon.com/blogs/security/how-to-scan-ec2-amis-using-amazon-inspector/"
date: "Tue, 18 Jul 2023 16:43:50 +0000"
author: "Luke Notley"
feed_url: "https://aws.amazon.com/blogs/security/category/security-identity-compliance/amazon-inspector/feed/"
---
<p><a href="https://aws.amazon.com/inspector/" rel="noopener" target="_blank">Amazon Inspector</a> is an automated vulnerability management service that continually scans <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> workloads for software vulnerabilities and unintended network exposure. Amazon Inspector supports vulnerability reporting and <a href="https://aws.amazon.com/about-aws/whats-new/2023/04/amazon-inspector-deep-inspection-ec2-instances/" rel="noopener" target="_blank">deep inspection</a> of <a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a> instances, container images stored in <a href="https://aws.amazon.com/ecr/" rel="noopener" target="_blank">Amazon Elastic Container Registry (Amazon ECR)</a>, and <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a> functions. <a href="https://docs.aws.amazon.com/inspector/latest/user/supported.html" rel="noopener" target="_blank">Operating system and programming language support</a> is extensive, ranging from <a href="https://aws.amazon.com/bottlerocket/" rel="noopener" target="_blank">Bottlerocket</a> to Windows Server.</p> 
<p>Many customers use <a href="https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html" rel="noopener" target="_blank">Amazon EC2 Auto Scaling groups</a> as part of their resilience and scaling architecture for their workloads. With Auto Scaling groups, you can scale and deploy rapidly by using <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html" rel="noopener" target="_blank">Amazon Machine Images (AMIs)</a>. However, AMIs within your environment can quickly become outdated as new vulnerabilities are discovered. A security best practice is to perform routine vulnerability assessments of your AMIs to identify whether newfound vulnerabilities apply to them. If you identify a vulnerability, you can update the AMI with the appropriate security patches, test the AMI in lower environments, and deploy the updated AMI in your environment. At this time, Amazon Inspector only supports scanning of running EC2 instances.</p> 
<p>In this blog post, we’ll share a solution that you can use with <a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">Amazon EventBridge</a>, AWS Lambda, <a href="https://aws.amazon.com/step-functions/" rel="noopener" target="_blank">AWS Step Functions</a>, <a href="https://aws.amazon.com/sns/" rel="noopener" target="_blank">Amazon Simple Notification Service (Amazon SNS)</a>­­, and <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> to scan AMIs and generate Amazon Inspector finding reports to help ensure that your AMIs are scanned for known vulnerabilities and updated prior to deployment. Then, we will show you how to periodically scan selected EC2 AMIs based on a tagging strategy, and take automated actions.</p> 
<h2>Prerequisites</h2> 
<p>The solution provided in this post has a number of items that you will need to review and address before you deploy the solution:</p> 
<ol> 
 <li>Make sure that the AMI to be scanned by Amazon Inspector is based from one of the <a href="https://docs.aws.amazon.com/inspector/latest/user/supported.html#supported-os-ec2" rel="noopener" target="_blank">operating systems that AWS supports for EC2 scanning</a>.</li> 
 <li>To successfully complete a scan, Amazon Inspector requires the EC2 instance to be a managed instance in <a href="https://aws.amazon.com/systems-manager/" rel="noopener" target="_blank">AWS Systems Manager</a> that has the <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html" rel="noopener" target="_blank">Systems Manager Agent</a> installed and running, and has an attached <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> instance profile that allows Systems Manager to manage the instance. For more information, see <a href="https://docs.aws.amazon.com/inspector/latest/user/scanning-ec2.html" rel="noopener" target="_blank">Scanning Amazon EC2 instances with Amazon Inspector</a>.</li> 
 <li>If you use customer managed keys to <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html" rel="noopener" target="_blank">encrypt Amazon Elastic Block Store (Amazon EBS) volumes</a> and you have a default EC2 configuration set to encrypt EBS volumes, you will need to configure additional key policy permissions. For the customer managed key that encrypts EBS volumes, add the following example policy statement to the key policy. Make sure to replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;111122223333&gt;</span> with your own AWS account ID. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">{
            "Sid": "Allow use of the key by AMI Scanner State Machine",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam:: <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;111122223333&gt;</span>:role/service-role/AMIScanner-Statemachine-role"
            },
            "Action": [
                "kms:Encrypt",
                "kms:Decrypt",
                "kms:ReEncrypt*",
                "kms:GenerateDataKey*",
                "kms:DescribeKey"
            ],
            "Resource": "*"
        },</code></pre> 
  </div> <p>If you don’t add this additional policy, the Step Functions state machine won’t allow the EC2 instances to launch. For more information, see <a href="https://docs.aws.amazon.com/autoscaling/ec2/userguide/key-policy-requirements-EBS-encryption.html#policy-example-cmk-access" rel="noopener" target="_blank">Key policy sections that allow access to the customer managed key</a>.</p> </li> 
 <li>The solution in this blog post requires that you activate Amazon Inspector in your AWS account. If you haven’t activated Amazon Inspector yet, learn more about the <a href="https://aws.amazon.com/inspector/pricing/" rel="noopener" target="_blank">free trial and pricing</a>, and follow the steps in the <a href="https://docs.aws.amazon.com/inspector/latest/user/getting_started_tutorial.html" rel="noopener" target="_blank">Amazon Inspector documentation</a> to set up the service and start monitoring your account. Alternatively, you can activate Amazon Inspector by using the <a href="https://aws.amazon.com/cli/" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI)</a> and this <a href="https://github.com/aws-samples/inspector2-enablement-with-cli" rel="noopener" target="_blank">GitHub example</a>.</li> 
</ol> 
<h2>Solution overview and architecture</h2> 
<p>In this solution, you will use the follow AWS services and features:</p> 
<ul> 
 <li><strong>Task orchestration</strong> 
  <ul> 
   <li>AWS Step Functions state machine workflows are used in this solution to verify that conditions are successfully validated before moving to the next task. This helps ensure that the Amazon Inspector scanning of the temporary instance launched in the first state machine is completed before the second state machine starts. This can help reduce the overall cost of the solution and can help prevent the first state machine from reaching state transition limitations.</li> 
   <li>Lambda functions handle the logic for retrieving AMIs to be scanned, launching temporary instances, creating Amazon EventBridge rules, tagging AMIs, and exporting Amazon Inspector reports to Amazon S3.</li> 
  </ul> </li> 
 <li><strong>AMI tagging</strong> 
  <ul> 
   <li>To use this solution, you need to tag the AMIs that Amazon Inspector will scan, because a Lambda function will use these tags to start the solution orchestration. For this post, we use the tag <span style="font-family: courier;">InspectorScan</span> with a value of <span style="font-family: courier;">true</span>. With <a href="https://aws.amazon.com/about-aws/whats-new/2020/12/amazon-machine-images-support-tag-on-create-tag-based-access-control/" rel="noopener" target="_blank">AMI tagging</a>, you can configure automated processes as part of your deployment pipelines to implement the tagging.</li> 
  </ul> </li> 
 <li><strong>Storage of exported Amazon Inspector findings</strong> 
  <ul> 
   <li>Amazon S3 helps you store the exported Amazon Inspector findings report and use them in a standardized format for multiple use cases across AWS services, or use <a href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a> to query the reports, which we will cover later in the post. Each scanned report is stored in the S3 bucket and is named in the form <span style="font-family: courier;">AMI-NAME/guid.JSON</span> or <span style="font-family: courier;">AMI-NAME/guid.CSV</span>, depending on the export format that you specify.</li> 
   <li>You can also use <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/enable-event-notifications.html" rel="noopener" target="_blank">S3 event notifications</a> to alert different operational teams that there are Amazon Inspector scan results that require review.</li> 
  </ul> </li> 
 <li><strong>Encryption of Amazon Inspector findings reports</strong> 
  <ul> 
   <li><a href="https://aws.amazon.com/kms/" rel="noopener" target="_blank">AWS Key Management Service (AWS KMS)</a> is used to encrypt the findings report. The AWS KMS key used must be a customer managed, symmetric KMS encryption key, and importantly, the key must be in the same AWS Region as the S3 bucket that you configured to store the report. The solution in this post creates a new KMS key, as well as a key policy that is configured to grant permissions for Amazon Inspector to use the key.</li> 
  </ul> </li> 
 <li><strong>Event tracking and scheduling</strong> 
  <ul> 
   <li>This solution uses an Amazon EventBridge rule to listen for completed Amazon Inspector scan events for each temporary EC2 instance launch. When the EventBridge rule finds a matched event, the rule passes the required parameters and invokes the second Step Functions state machine. The event pattern used in this solution uses the following format: 
    <div class="hide-language"> 
     <pre><code class="lang-text">	{
			"source": ["aws.inspector2"],
			"detail-type": ["Inspector2 Scan"],
			"resources": ["i-abcdef01234567890"]
		}</code></pre> 
    </div> </li> 
   <li>You can schedule the AMI scanning by using an EventBridge rule that invokes a Lambda function that runs on a schedule. The Lambda function uses a cron expression to occur weekly. You can configure this parameter according to your requirements. Initially, this rule will be disabled to allow you to configure and enable the rule at a later stage.</li> 
   <li>Amazon SNS sends notifications during the AMI scanning solution process. From the SNS topic, you can <a href="https://docs.aws.amazon.com/sns/latest/dg/sns-create-subscribe-endpoint-to-topic.html" rel="noopener" target="_blank">configure different subscriptions</a>, depending on your preferred use case and environment. An example of a subscription could be a shared mailbox email address for the security team or incident ticketing system.</li> 
  </ul> </li> 
</ul> 
<p>Figure 1 shows the solution architecture.</p> 
<div class="wp-caption aligncenter" id="attachment_30180" style="width: 790px;">
 <img alt="Figure 1: Amazon Inspector scanning of an AMI" class="size-full wp-image-30180" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/07/14/img1-3.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-30180">Figure 1: Amazon Inspector scanning of an AMI</p>
</div> 
<p>The high-level workflow of the solution is as follows:</p> 
<ol> 
 <li>You can use <a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">EventBridge</a> to create a scheduled rule to invoke a Lambda function. You can set the rule for daily, weekly, or monthly, depending on your use case.</li> 
 <li>The Lambda function searches for AMIs with the appropriate tags and passes these as parameters to the Step Functions workflow.</li> 
 <li>The first Step Functions state machine is invoked for each AMI to be scanned.</li> 
 <li>The first Step Functions workflow deploys a temporary EC2 instance from the AMI that is defined.</li> 
 <li>A Lambda function is invoked to create an EventBridge rule.</li> 
 <li>An EventBridge rule is created to listen for the successful Amazon Inspector scanned event of the temporary EC2 instance.</li> 
 <li>A Lambda function is invoked to tag the EC2 instance.</li> 
 <li>The temporary EC2 instance is tagged, showing Amazon Inspector that scanning is in progress.</li> 
 <li>The first Step Functions workflow sends a notification to an SNS topic.</li> 
 <li>The EventBridge rule parses the required parameters and invokes the second Step Functions state machine.</li> 
 <li>A Lambda function is invoked to generate an Amazon Inspector report and export the findings to an S3 bucket.</li> 
 <li>The scanned Amazon Inspector AMI results are saved to an S3 bucket.</li> 
 <li>The Step Functions workflow terminates the temporary EC2 instance that can reduce cost and clean up the process.</li> 
 <li>A Lambda function is invoked to delete the temporary EventBridge rule.</li> 
 <li>The temporary EventBridge rule and targets are deleted.</li> 
 <li>A Lambda function is invoked to tag the AMI.</li> 
 <li>The scanned AMI is updated with tagging metadata.</li> 
 <li>The second Step Functions workflow sends a final notification to an SNS topic.</li> 
</ol> 
<h2>Deploy the solution</h2> 
<p>The solution will be deployed with the scheduled rule in Amazon EventBridge disabled to allow you to create your tagging strategy and to familiarize yourself with the solution. Later in this post, we’ll cover how to enable the Amazon EventBridge scheduled rule.</p> 
<h3>Step 1: Deploy the CloudFormation template</h3> 
<p>For this next step, make sure that you deploy the <a href="https://github.com/aws-samples/inspector-ami-scanning-solution/blob/main/Scheduled-Multi-AMI-Scanner.yaml" rel="noopener" target="_blank">CloudFormation template provided for multi-AMI scanning</a> in the AWS account and Region where you want to test this solution.</p> 
<h4>To deploy the CloudFormation template</h4> 
<ol> 
 <li>Choose the following <strong>Launch Stack</strong> button to launch a CloudFormation stack in your account. Note that the stack will launch in the N. Virginia (<span style="font-family: courier;">us-east-1</span>) Region. To deploy this solution into other AWS Regions<a href="https://github.com/aws-samples/inspector-ami-scanning-solution/blob/main/Scheduled-Multi-AMI-Scanner.yaml" rel="noopener" target="_blank">, download the solution’s CloudFormation template</a>, modify it, and deploy it to the selected Region. <p><a href="https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=AMIScanner&amp;templateURL=https://aws-security-blog-content.s3.amazonaws.com/public/sample/1634-how-to-scan-ec2-amis/Scheduled-Multi-AMI-Scanner.yaml" rel="noopener" target="_blank"><img alt="Select this image to open a link that starts building the CloudFormation stack" class="aligncenter size-full wp-image-10149" height="36" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2019/06/05/launch-stack-button.png" width="190" /></a></p> <p>Make sure that you configure the following parameters in the CloudFormation template so that it deploys successfully:</p> 
  <ul> 
   <li><span style="font-family: courier;">AMITagName</span> — The AMI tag name to check if the AMI should be scanned by Amazon Inspector.</li> 
   <li><span style="font-family: courier;">AMITagValue</span> — The AMI tag value to check if the AMI should be scanned by Amazon Inspector.</li> 
   <li><span style="font-family: courier;">InspectorReportFormat</span> — The report format, which can be either CSV or JSON.</li> 
   <li><span style="font-family: courier;">InstanceSubnetID</span> — The subnet ID to launch the temporary EC2 instance into.</li> 
   <li><span style="font-family: courier;">InstanceType</span> — The instance type to deploy the AMI to for temporary scanning purposes.</li> 
   <li><span style="font-family: courier;">KmsKeyAdministratorRole</span> — The existing IAM role that needs to have administrator access to the KMS key created for the solution. This key provides access to encrypt and decrypt the Amazon Inspector report.</li> 
   <li><span style="font-family: courier;">S3ReportBucketName</span> — The name of the S3 bucket to be created.</li> 
   <li><span style="font-family: courier;">SnsTopic</span> — The name of the new SNS topic to be created. This name defines the SNS topic that notifications are published to.</li> 
  </ul> </li> 
 <li>Review the stack name and the parameters for the template.</li> 
 <li>On the <strong>Quick create stack</strong> screen, scroll to the bottom and select <strong>I acknowledge that AWS CloudFormation might create IAM resources</strong>.</li> 
 <li>Choose <strong>Create stack</strong>. The deployment of the CloudFormation stack will take 3–4 minutes.</li> 
</ol> 
<p>After the CloudFormation stack has deployed successfully, you can use the deployed solution.</p> 
<h3>Step 2: Manually run the first Step Functions workflow</h3> 
<p>The first Step Functions state machine requires parameters to be passed in; the <span style="font-family: courier;">SingleAMI</span> Lambda function accomplishes this. You can start the Lambda function by <a href="https://docs.aws.amazon.com/lambda/latest/dg/testing-functions.html" rel="noopener" target="_blank">creating a test event</a> and passing the correct JSON text and parameters. The following parameters are available in the output section of the CloudFormation stack that the solution deployed:</p> 
<ul> 
 <li><span style="font-family: courier;">AmiId</span> — The ID of the AMI to be used for deploying the EC2 instance. This is the EC2 AMI to be scanned.</li> 
 <li><span style="font-family: courier;">EC2InstanceProfile</span> — The Amazon Resource Name (ARN) of the EC2 instance profile that the CloudFormation stack created.</li> 
 <li><span style="font-family: courier;">InstanceType</span> — The type of EC2 instance to use for deployment.</li> 
 <li><span style="font-family: courier;">KmsKeyName</span> — The ARN of the KMS key to be used for encrypting and decrypting the Amazon Inspector report that the CloudFormation stack created.</li> 
 <li><span style="font-family: courier;">S3Bucket</span> — The name of the S3 bucket to which the Amazon Inspector reports will be exported. The S3 bucket was created previously by the CloudFormation stack.</li> 
 <li><span style="font-family: courier;">S3ReportFormat</span> — The report format that Amazon Inspector will use to export the findings report; either the JSON or the CSV format is valid.</li> 
 <li><span style="font-family: courier;">SnsTopc</span> — The ARN of the SNS topic to which notifications will be sent. This SNS topic was created previously by the CloudFormation stack.</li> 
 <li><span style="font-family: courier;">StateMachineArn</span> — The ARN of the first Step Functions state machine, which the Lambda function will run first.</li> 
 <li><span style="font-family: courier;">SubnetId</span> — The ID of the VPC subnet to which the EC2 instance will be attached and launched into. This is a required parameter and could be a subnet that is created specifically for this scanning purpose.</li> 
</ul> 
<p>The following is an example parameter configuration and JSON that you can use to run the Lambda function. Make sure to replace each <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;user input placeholder&gt;</span> with your own information.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
"AmiId" : "<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AMI-ABCDEF01234567890&gt;</span>",
"Ec2InstanceProfile" : "arn:aws:iam::<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;111122223333&gt;</span>:instance-profile/Ec2InstanceLaunchRole",
"InstanceType" : "t3.medium",
"KMSKeyName" : "arn:aws:kms:region-name:<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;111122223333&gt;</span>:key/<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;a1b2c3d4-5678-90ab-cdef-EXAMPLE11111&gt;</span>",
"S3Bucket" : "<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;DOC-EXAMPLE-BUCKET-111122223333&gt;</span>",
"S3ReportFormat" : "CSV",
"SnsTopic" : "arn:aws:sns:region-name-2:<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;111122223333&gt;</span>:InspectorScanner",
"StateMachine": "arn:aws:states:region-name:<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;111122223333&gt;</span>:stateMachine:AMIScanner-Part1-LaunchEC2",
"SubnetId" : "<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;SUBNET-ABCDEF01234567890&gt;</span>"
}</code></pre> 
</div> 
<p>After the first state machine is finished, the EventBridge rule listens for the successful Amazon Inspector scan event. An SNS notification is sent, similar to the following.</p> 
<div class="hide-language">
 <code style="color: #000000;"><br /> {"AWS Inspector AMI Scan status":"EC2 instance","For AMI":"ami-abcdef01234567890","Temporarily launched AMI using instance":"i-abcdef01234567890"}<p></p> </code>
 <p><code style="color: #000000;"> </code></p>
</div> 
<p>After Amazon Inspector has finished scanning the EC2 instance, and the second state machine completes successfully, the Amazon Inspector finding report appears in the S3 bucket and notifications appear on the SNS topic that was created. The following is an example of an SNS notification.</p> 
<div class="hide-language">
 <code style="color: #000000;"><br /> {"AWS Inspector AMI Scan completed":"Successfully","For AMI":"ami-abcdef01234567890","AWS Inspector report located at S3 Bucket":"DOC-EXAMPLE-BUCKET-111122223333","Temporarily launched AMI using instance":"i-abcdef01234567890"}<p></p> </code>
 <p><code style="color: #000000;"> </code></p>
</div> 
<h3>Enable scheduled scanning</h3> 
<p>You can enable the EventBridge scheduled rule to handle multiple AMIs and automatic scheduling. The scheduled rule invokes a Lambda function on a scheduled basis that identifies AMIs with the appropriate tags and passes parameters to the Step Functions workflow.</p> 
<h4>To enable the rule</h4> 
<ul> 
 <li>In the EventBridge rules console, navigate to <strong>AMIScanner-ScheduledSolutionTask</strong>, and choose <strong>Enable</strong>. 
  <div class="wp-caption alignnone" id="attachment_30184" style="width: 610px;">
   <img alt="Figure 2: Enable Amazon EventBridge scheduled rule" class="size-full wp-image-30184" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/07/14/img2-4.png" style="border: 1px solid #bebebe;" width="600" />
   <p class="wp-caption-text" id="caption-attachment-30184">Figure 2: Enable Amazon EventBridge scheduled rule</p>
  </div> </li> 
</ul> 
<h3>Extend the solution</h3> 
<p>With <a href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a>, you can run SQL queries on raw data that is stored in S3 buckets. The Amazon Inspector reports are exported to S3, and you can query the data and create tables by using <a href="https://aws.amazon.com/glue/" rel="noopener" target="_blank">AWS Glue</a> crawlers. To make sure that AWS Glue can crawl the S3 data, you need to add the role that you create for AWS Glue to the AWS KMS key permissions, so that AWS Glue can decrypt the S3 data. The following is an example policy JSON that you can update. Make sure to replace the AWS account ID <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;111122223333&gt;</span> and S3 bucket name <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;DOC-EXAMPLE-BUCKET-111122223333&gt;</span> with your own information.</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">{
"Sid": "Allow the AWS Glue crawler usage of the KMS key",
"Effect": "Allow",
"Principal": {
"AWS": "arn:aws:iam::<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;111122223333&gt;</span>:role/service-role/AWSGlueServiceRole-S3InspectorReports"
},
"Action": [
"kms:Decrypt",
"kms:GenerateDataKey*"
],
"Resource": "arn:aws:s3:::<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;DOC-EXAMPLE-BUCKET-111122223333&gt;</span>"
},</code></pre> 
</div> 
<p>After an AWS Glue Data Catalog has been built, you can run the crawler on a scheduled basis to help keep the catalog up to date with the latest Amazon Inspector findings as they are exported into the S3 bucket.</p> 
<p>Using Amazon Athena, you can run queries against the Amazon Inspector reports to generate output data that is relevant to your environment. For example, to list the AMIs that are affected by high-severity findings, you can run the following SQL query. Make sure to replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;DOC-EXAMPLE-BUCKET-111122223333&gt;</span> with your own information.</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">SELECT DISTINCT partition_0 from "<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;DOC-EXAMPLE-BUCKET-111122223333&gt;</span>" where severity='HIGH'</code></pre> 
</div> 
<p>With the results, you can <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-tutorial-update-ami.html" rel="noopener" target="_blank">use AWS Systems Manager to update the relevant AMIs</a> to include the latest patches and update the launch template used in <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-tutorial-update-patch-windows-ami-autoscaling.html" rel="noopener" target="_blank">your Auto Scaling groups.</a></p> 
<p>To further extend this solution, you can also use <a href="https://aws.amazon.com/quicksight/" rel="noopener" target="_blank">Amazon QuickSight</a> to visualize the data by connecting to the AWS Glue table and producing dashboards for consumption.</p> 
<h2>Conclusion</h2> 
<p>By performing security assessments of your AMIs on a regular basis, you can gain greater visibility and control over the security of your EC2 instances that are created from those AMIs. In this blog post, you learned how to set up AMI vulnerability assessments, and how the results of these continuous vulnerability assessments can help you keep your environment up to date with security patches. For additional hands-on walkthroughs for Amazon Inspector, see <a href="https://catalog.workshops.aws/inspector/" rel="noopener" target="_blank">Amazon Inspector workshops</a>. You can find the code for this blog post in the <a href="https://github.com/aws-samples/inspector-ami-scanning-solution/" rel="noopener" target="_blank">inspector-ami-scanning-solution GitHub repository</a>.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<p><strong>Want more AWS Security news? Follow us on <a href="https://twitter.com/AWSsecurityinfo" rel="noopener noreferrer" target="_blank" title="Twitter">Twitter</a>.</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Luke Notley" class="aligncenter size-full wp-image-30186" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/07/14/Luke-Notley.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Luke Notley</h3> 
  <p>Luke is a Senior Solutions Architect with Amazon Web Services and is based in Western Australia. Luke has a passion for helping customers connect business outcomes with technology and assisting customers throughout their cloud journey, helping them design scalable, flexible, and resilient architectures. In his spare time, he enjoys traveling, coaching basketball teams, and DJing.</p> 
 </div> 
</footer>
