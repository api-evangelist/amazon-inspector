---
title: "How to generate security findings to help your security team with incident response simulations"
url: "https://aws.amazon.com/blogs/security/how-to-generate-security-findings-to-help-your-security-team-with-incident-response-simulations/"
date: "Mon, 01 Apr 2024 16:00:03 +0000"
author: "Jonathan Nguyen"
feed_url: "https://aws.amazon.com/blogs/security/category/security-identity-compliance/amazon-inspector/feed/"
---
<blockquote>
 <p><strong>April 8, 2024:</strong> We have updated the post to revise the CloudFormation launch stack link to provision the CloudFormation template.</p>
</blockquote> 
<hr /> 
<p>Continually reviewing your organization’s incident response capabilities can be challenging without a mechanism to create security findings with actual <a href="https://aws.amazon.com/aws" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> resources within your AWS estate. As prescribed within the <a href="https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/run-regular-simulations.html" rel="noopener" target="_blank">AWS Security Incident Response whitepaper</a>, it’s important to periodically review your incident response capabilities to make sure your security team is continually maturing internal processes and assessing capabilities within AWS. Generating sample security findings is useful to understand the finding format so you can enrich the finding with additional metadata or create and prioritize detections within your security information event management (SIEM) solution. However, if you want to conduct an end-to-end incident response simulation, including the creation of real detections, sample findings might not create actionable detections that will start your incident response process because of alerting suppressions you might have configured, or imaginary metadata (such as synthetic <a href="https://aws.amazon.com/ec2" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a> instance IDs), which might confuse your remediation tooling.</p> 
<p>In this post, we walk through how to deploy a solution that provisions resources to generate simulated security findings for actual provisioned resources within your AWS account. Generating simulated security findings in your AWS account gives your security team an opportunity to validate their cyber capabilities, investigation workflow and playbooks, escalation paths across teams, and exercise any response automation currently in place.</p> 
<blockquote>
 <p><strong>Important:</strong> It’s strongly recommended that the solution be deployed in an isolated AWS account with no additional workloads or sensitive data. No resources deployed within the solution should be used for any purpose outside of generating the security findings for incident response simulations. Although the security findings are non-destructive to existing resources, they should still be done in isolation. For any AWS solution deployed within your AWS environment, your security team should review the resources and configurations within the code.</p>
</blockquote> 
<h2>Conducting incident response simulations</h2> 
<p>Before deploying the solution, it’s important that you know what your goal is and what <a href="https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/types-of-simulations.html" rel="noopener" target="_blank">type of simulation</a> to conduct. If you’re primarily curious about the format that active <a href="https://aws.amazon.com/guardduty" rel="noopener" target="_blank">Amazon GuardDuty</a> findings will create, you should generate <a href="https://docs.aws.amazon.com/guardduty/latest/ug/sample_findings.html" rel="noopener" target="_blank">sample findings with GuardDuty</a>. At the time of this writing, <a href="https://aws.amazon.com/inspector/" rel="noopener" target="_blank">Amazon Inspector</a> doesn’t currently generate sample findings.</p> 
<p>If you want to validate your incident response playbooks, make sure you have playbooks for the security findings the solution generates. If those playbooks don’t exist, it might be a good idea to start with a high-level tabletop exercise to identify which playbooks you need to create.</p> 
<p>Because you’re running this sample in an AWS account with no workloads, it’s recommended to run the sample solution as a <a href="https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/types-of-simulations.html" rel="noopener" target="_blank">purple team exercise</a>. Purple team exercises should be periodically run to support training for new analysts, validate existing playbooks, and identify areas of improvement to reduce the <a href="https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/mean-time-to-respond.html" rel="noopener" target="_blank">mean time to respond</a> or identify areas where processes can be optimized with automation.</p> 
<p>Now that you have a good understanding of the different simulation types, you can create security findings in an isolated AWS account.</p> 
<h2>Prerequisites</h2> 
<ol> 
 <li>[Recommended] A separate AWS account containing no customer data or running workloads</li> 
 <li><a href="https://aws.amazon.com/guardduty/" rel="noopener" target="_blank">GuardDuty</a>, along with GuardDuty <a href="https://docs.aws.amazon.com/guardduty/latest/ug/kubernetes-protection.html" rel="noopener" target="_blank">Kubernetes Protection</a></li> 
 <li><a href="https://aws.amazon.com/inspector/" rel="noopener" target="_blank">Amazon Inspector</a> must be enabled</li> 
 <li>[Optional] <a href="https://aws.amazon.com/security-hub/" rel="noopener" target="_blank">AWS Security Hub</a> can be enabled to show a consolidated view of security findings generated by GuardDuty and Inspector</li> 
</ol> 
<h2>Solution architecture</h2> 
<p>The architecture of the solution can be found in Figure 1.</p> 
<div class="wp-caption aligncenter" id="attachment_33785" style="width: 790px;">
 <img alt="Figure 1: Sample solution architecture diagram" class="size-full wp-image-33785" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/03/26/img1-8.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-33785">Figure 1: Sample solution architecture diagram</p>
</div> 
<ol> 
 <li>A user specifies the type of security findings to generate by passing an <a href="https://aws.amazon.com/cloudformation" rel="noopener" target="_blank">AWS CloudFormation</a> parameter.</li> 
 <li>An <a href="https://aws.amazon.com/sns" rel="noopener" target="_blank">Amazon Simple Notification Service (Amazon SNS)</a> topic is created to subscribe to findings for notifications. Subscribed users are notified of the finding through the deployed SNS topic.</li> 
 <li>Upon user selection of the CloudFormation parameter, <a href="https://aws.amazon.com/ec2" rel="noopener" target="_blank">EC2</a> instances are provisioned to run commands to generate security findings.<br /> 
  <blockquote>
   <p><strong>Note</strong>: If the parameter <span style="font-family: courier;">inspector</span> is provided during deployment, then only one EC2 instance is deployed. If the parameter <span style="font-family: courier;">guardduty</span> is provided during deployment, then two EC2 instances are deployed.</p>
  </blockquote> </li> 
 <li>For Amazon Inspector findings: 
  <ol> 
   <li>The Amazon EC2 user data creates a .txt file with vulnerable images, pulls down Docker images from <a href="https://github.com/vulhub/vulhub" rel="noopener" target="_blank">open source vulhub</a>, and creates an <a href="https://aws.amazon.com/ecr" rel="noopener" target="_blank">Amazon Elastic Container Registry (Amazon ECR)</a> repository with the vulnerable images.</li> 
   <li>The EC2 user data pushes and tags the images in the ECR repository which results in Amazon Inspector findings being generated.</li> 
   <li>An <a href="https://aws.amazon.com/eventbridge" rel="noopener" target="_blank">Amazon EventBridge</a> cron-style trigger rule, <span style="font-family: courier;">inspector_remediation_ecr</span>, invokes an <a href="https://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> function.</li> 
   <li>The Lambda function, <span style="font-family: courier;">ecr_cleanup_function</span>, cleans up the vulnerable images in the deployed Amazon ECR repository based on applied tags and sends a notification to the Amazon SNS topic.<br /> 
    <blockquote>
     <p><strong>Note</strong>: The <span style="font-family: courier;">ecr_cleanup_function</span> Lambda function is also invoked as a custom resource to clean up vulnerable images during deployment. If there are issues with cleanup, the EventBridge rule continually attempts to clean up vulnerable images.</p>
    </blockquote> </li> 
  </ol> </li> 
 <li>For GuardDuty, the following actions are taken and resources are deployed: 
  <ol> 
   <li>An <a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> user named guardduty-demo-user is created with an IAM access key that is INACTIVE.</li> 
   <li>An <a href="https://aws.amazon.com/systems-manager/" rel="noopener" target="_blank">AWS Systems Manager</a> parameter stores the IAM access key for guardduty-demo-user.</li> 
   <li>An <a href="https://aws.amazon.com/secrets-manager/" rel="noopener" target="_blank">AWS Secrets Manager</a> secret stores the inactive IAM secret access key for guardduty-demo-user.</li> 
   <li>An <a href="https://aws.amazon.com/dynamodb/" rel="noopener" target="_blank">Amazon DynamoDB</a> table is created, and the table name is stored in a Systems Manager parameter to be referenced within the EC2 user data.</li> 
   <li>An <a href="https://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> bucket is created, and the bucket name is stored in a Systems Manager parameter to be referenced within the EC2 user data.</li> 
   <li>A Lambda function adds a threat list to GuardDuty that includes the IP addresses of the EC2 instances deployed as part of the sample.</li> 
   <li>EC2 user data generates GuardDuty findings for the following: 
    <ol> 
     <li><a href="https://aws.amazon.com/eks" rel="noopener" target="_blank">Amazon Elastic Kubernetes Service (Amazon EKS)</a> 
      <ol> 
       <li>Installs eksctl from <a href="https://github.com/aws-samples/generate-aws-security-services-findings/blob/1154e13da86c8e524b4fb21c6053a621f2238832/source/user-data/guardduty-user-data.sh#L11" rel="noopener" target="_blank">GitHub</a>.</li> 
       <li>Creates an EC2 key pair.</li> 
       <li>Creates an EKS cluster (dependent on <a href="https://docs.aws.amazon.com/eks/latest/userguide/troubleshooting.html#ICE" rel="noopener" target="_blank">availability zone capacity</a>).</li> 
       <li>Updates EKS cluster configuration to make a dashboard public.</li> 
      </ol> </li> 
     <li>DynamoDB 
      <ol> 
       <li>Adds an item to the DynamoDB table for <strong>Joshua Tree</strong>.</li> 
      </ol> </li> 
     <li>EC2 
      <ol> 
       <li>Creates an <a href="https://aws.amazon.com/cloudtrail" rel="noopener" target="_blank">AWS CloudTrail</a> trail named <span style="font-family: courier;">guardduty-demo-trail-&lt;GUID&gt;</span> and subsequently deletes the same CloudTrail trail. The <span style="font-family: courier;">&lt;GUID&gt;</span> is randomly generated by using the <a href="https://tldp.org/LDP/abs/html/randomvar.html" rel="noopener" target="_blank">$RANDOM</a> function</li> 
       <li>Runs <span style="font-family: courier;">portscan</span> on 172.31.37.171 (an RFC 1918 private IP address) and private IP of the EKS Deployment EC2 instance provisioned as part of the sample. Port scans are primarily used by bad actors to search for potential vulnerabilities. The target of the port scans are internal IP addresses and do not leave the sample VPC deployed.</li> 
       <li>Curls DNS domains that are labeled for bitcoin, command and control, and other domains associated with known threats.</li> 
      </ol> </li> 
     <li>Amazon S3 
      <ol> 
       <li>Disables Block Public Access and server access logging for the S3 bucket provisioned as part of the solution.</li> 
      </ol> </li> 
     <li>IAM 
      <ol> 
       <li>Deletes the existing account password policy and creates a new password policy with a minimum length of six characters.</li> 
      </ol> </li> 
    </ol> </li> 
  </ol> </li> 
 <li>The following <a href="https://aws.amazon.com/eventbridge" rel="noopener" target="_blank">Amazon EventBridge</a> rules are created: 
  <ol> 
   <li><span style="font-family: courier;">guardduty_remediation_eks_rule</span> – When a GuardDuty finding for EKS is created, a Lambda function attempts to delete the EKS resources. Subscribed users are notified of the finding through the deployed SNS topic.</li> 
   <li><span style="font-family: courier;">guardduty_remediation_credexfil_rule</span> – When a GuardDuty finding for <span style="font-family: courier;">InstanceCredentialExfiltration</span> is created, a Lambda function is used to <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_revoke-sessions.html" rel="noopener" target="_blank">revoke the IAM role’s temporary security credentials</a> and AWS permissions. Subscribed users are notified of the finding through the deployed SNS topic.</li> 
   <li><span style="font-family: courier;">guardduty_respond_IAMUser_rule</span> – When a GuardDuty finding for IAM is created, subscribed users are notified through the deployed SNS topic. There is no remediation activity triggered by this rule.</li> 
   <li><span style="font-family: courier;">Guardduty_notify_S3_rule</span> – When a GuardDuty finding for Amazon S3 is created, subscribed users are notified through the deployed Amazon SNS topic. This rule doesn’t invoke any remediation activity.</li> 
  </ol> </li> 
 <li>The following Lambda functions are created: 
  <ol> 
   <li><span style="font-family: courier;">guardduty_iam_remediation_function</span> – This function revokes active sessions and sends a notification to the SNS topic.</li> 
   <li><span style="font-family: courier;">eks_cleanup_function</span> – This function deletes the EKS resources in the EKS CloudFormation template.<br /> 
    <blockquote>
     <p><strong>Note</strong>: Upon attempts to delete the overall sample CloudFormation stack, this runs to delete the EKS CloudFormation template.</p>
    </blockquote> </li> 
  </ol> </li> 
 <li>An S3 bucket stores EC2 user data scripts run from the EC2 instances</li> 
</ol> 
<h2>Solution deployment</h2> 
<p>You can deploy the <strong>SecurityFindingGeneratorStack</strong> solution by using either the <a href="https://aws.amazon.com/console/" rel="noopener" target="_blank">AWS Management Console</a> or the <a href="https://aws.amazon.com/cdk/" rel="noopener" target="_blank">AWS Cloud Development Kit (AWS CDK)</a>.</p> 
<h3>Option 1: Deploy the solution with AWS CloudFormation using the console</h3> 
<p>Use the console to sign in to your chosen AWS account and then choose the <strong>Launch Stack</strong> button to open the <a href="https://aws.amazon.com/cloudformation" rel="noopener" target="_blank">AWS CloudFormation</a> console pre-loaded with the template for this solution. It takes approximately 10 minutes for the CloudFormation stack to complete.</p> 
<p><a href="https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=securityhubcorrelation&amp;templateURL=https://aws-security-blog-content.s3.amazonaws.com/public/sample/1426-generate-security-services-findings/security_finding_generator.yaml" rel="noopener noreferrer" target="_blank"><img alt="Launch Stack" class="aligncenter size-full wp-image-10149" height="36" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2019/06/05/launch-stack-button.png" width="190" /></a></p> 
<h3>Option 2: Deploy the solution by using the AWS CDK</h3> 
<p>You can find the latest code for the SecurityFindingGeneratorStack solution in the <a href="https://github.com/aws-samples/generate-aws-security-services-findings" rel="noopener" target="_blank">SecurityFindingGeneratorStack GitHub repository</a>, where you can also contribute to the sample code. For instructions and more information on using the <a href="https://aws.amazon.com/cdk" rel="noopener" target="_blank">AWS Cloud Development Kit (AWS CDK)</a>, see <a href="https://aws.amazon.com/getting-started/guides/setup-cdk/" rel="noopener" target="_blank">Get Started with AWS CDK</a>.</p> 
<h4>To deploy the solution by using the AWS CDK</h4> 
<ol> 
 <li>To build the app when navigating to the project’s root folder, use the following commands: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">npm install -g aws-cdk-lib
npm install</code></pre> 
  </div> </li> 
 <li>Run the following command in your terminal while authenticated in your separate deployment AWS account to bootstrap your environment. Be sure to replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;INSERT_AWS_ACCOUNT&gt;</span> with your account number and replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;INSERT_REGION&gt;</span> with the AWS Region that you want the solution deployed to. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">cdk bootstrap aws://<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;INSERT_AWS_ACCOUNT&gt;</span>/<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;INSERT_REGION&gt;</span></code></pre> 
  </div> </li> 
 <li>Deploy the stack to generate findings based on a specific parameter that is passed. The following parameters are available: 
  <ol style="font-family: courier;"> 
   <li>inspector</li> 
   <li>guardduty</li> 
  </ol> 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">cdk deploy SecurityFindingGeneratorStack –-parameters securityserviceuserdata=inspector</code></pre> 
  </div> </li> 
</ol> 
<h2>Reviewing security findings</h2> 
<p>After the solution successfully deploys, security findings should start appearing in your AWS account’s GuardDuty console within a couple of minutes.</p> 
<h3>Amazon GuardDuty findings</h3> 
<p>In order to create a diverse set of GuardDuty findings, the solution uses Amazon EC2 <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html" rel="noopener" target="_blank">user data</a> to run scripts. Those scripts can be found in the <a href="https://github.com/aws-samples/generate-aws-security-services-findings/tree/main/source/user-data" rel="noopener" target="_blank">sample repository</a>. You can also review and change scripts as needed to fit your use case or to remove specific actions if you don’t want specific resources to be altered or security findings to be generated.</p> 
<p>A comprehensive list of active GuardDuty finding types and details for each finding can be found in the <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html" rel="noopener" target="_blank">Amazon GuardDuty user guide</a>. In this solution, activities which cause the following GuardDuty findings to be generated, are performed:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#backdoor-ec2-ccactivitybdns" rel="noopener" target="_blank">Backdoor:EC2/C&amp;CActivity.B!DNS</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#cryptocurrency-ec2-bitcointoolbdns" rel="noopener" target="_blank">CryptoCurrency:EC2/BitcoinTool.B!DNS</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-iam.html#stealth-iam-passwordpolicychange" rel="noopener" target="_blank">Stealth:IAMUser/PasswordPolicyChange</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-kubernetes.html#discovery-kubernetes-successfulanonymousaccess" rel="noopener" target="_blank">Discovery:Kubernetes/SuccessfulAnonymousAccess</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-kubernetes.html#discovery-kubernetes-maliciousipcaller" rel="noopener" target="_blank">Discovery:Kubernetes/MaliciousIPCaller</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-kubernetes.html#execution-kubernetes-execinkubesystempod" rel="noopener" target="_blank">Execution:Kubernetes/ExecInKubeSystemPod</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-kubernetes.html#policy-kubernetes-adminaccesstodefaultserviceaccount" rel="noopener" target="_blank">Policy:Kubernetes/AdminAccessToDefaultServiceAccount</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-kubernetes.html#policy-kubernetes-anonymousaccessgranted" rel="noopener" target="_blank">Policy:Kubernetes/AnonymousAccessGranted</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-kubernetes.html#persistence-kubernetes-containerwithsensitivemount" rel="noopener" target="_blank">Persistence:Kubernetes/ContainerWithSensitiveMount</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-kubernetes.html#privilegeescalation-kubernetes-privilegedcontainer" rel="noopener" target="_blank">PrivilegeEscalation:Kubernetes/PrivilegedContainer</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#recon-ec2-portscan" rel="noopener" target="_blank">Recon:EC2/Portscan</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-s3.html#policy-s3-bucketblockpublicaccessdisabled" rel="noopener" target="_blank">Policy:S3/BucketBlockPublicAccessDisabled</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-s3.html#stealth-s3-serveraccessloggingdisabled" rel="noopener" target="_blank">Stealth:S3/ServerAccessLoggingDisabled</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#trojan-ec2-blackholetrafficdns" rel="noopener" target="_blank">Trojan:EC2/BlackholeTraffic!DNS</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#trojan-ec2-dgadomainrequestb" rel="noopener" target="_blank">Trojan:EC2/DGADomainRequest.B</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#trojan-ec2-dnsdataexfiltration" rel="noopener" target="_blank">Trojan:EC2/DNSDataExfiltration</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#trojan-ec2-drivebysourcetrafficdns" rel="noopener" target="_blank">Trojan:EC2/DriveBySourceTraffic!DNS</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#trojan-ec2-phishingdomainrequestdns" rel="noopener" target="_blank">Trojan:EC2/PhishingDomainRequest!DNS</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#unauthorizedaccess-ec2-torclient" rel="noopener" target="_blank">UnauthorizedAccess:EC2/TorClient</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#unauthorizedaccess-ec2-metadatadnsrebind" rel="noopener" target="_blank">UnauthorizedAccess:EC2/MetadataDNSRebind</a></li> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#unauthorizedaccess-ec2-maliciousipcallercustom" rel="noopener" target="_blank">UnauthorizedAccess:EC2/MaliciousIPCaller.Custom</a></li> 
</ul> 
<p>To generate the EKS security findings, the EKS Deployment EC2 instance is running eksctl commands that deploy CloudFormation templates. If the EKS cluster doesn’t deploy, it might be because of <a href="https://docs.aws.amazon.com/eks/latest/userguide/troubleshooting.html#ICE" rel="noopener" target="_blank">capacity restraints in a specific Availability Zone</a>. If this occurs, manually delete the failed EKS CloudFormation templates.</p> 
<p>If you want to create the EKS cluster and security findings manually, you can do the following:</p> 
<ol> 
 <li>Sign in to the <a href="https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Home:" rel="noopener" target="_blank">Amazon EC2</a> console.</li> 
 <li>Connect to the EKS Deployment EC2 instance using an IAM role that has access to start a session through <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/session-manager-to-linux.html" rel="noopener" target="_blank">Systems Manager</a>. After connecting to the <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent-technical-details.html#ssm-user-account" rel="noopener" target="_blank">ssm-user</a>, issue the following commands in the <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html#what-is-a-session" rel="noopener" target="_blank">Session Manager session</a>: 
  <ol style="font-family: courier;"> 
   <li>sudo chmod 744 /home/ec2-user/guardduty-script.sh</li> 
   <li>chown ec2-user /home/ec2-user/guardduty-script.sh</li> 
   <li>sudo /home/ec2-user/guardduty-script.sh</li> 
  </ol> </li> 
</ol> 
<p>It’s important that your security analysts have an incident response playbook. If playbooks don’t exist, you can refer to the <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_remediate.html" rel="noopener" target="_blank">GuardDuty remediation recommendations</a> or <a href="https://github.com/aws-samples/aws-incident-response-playbooks/tree/master/playbooks/Amazon%20GuardDuty%20Playbooks" rel="noopener" target="_blank">AWS sample incident response playbooks</a> to get started building playbooks.</p> 
<h3>Amazon Inspector findings</h3> 
<p>The findings for Amazon Inspector are generated by using the open source <a href="https://github.com/vulhub/vulhub" rel="noopener" target="_blank">Vulhub</a> collection. The open source collection has pre-built vulnerable Docker environments that pull images into Amazon ECR.</p> 
<p>The Amazon Inspector findings that are created vary depending on what exists within the open source library at deployment time. The following are examples of findings you will see in the console:</p> 
<ul> 
 <li><a href="https://access.redhat.com/security/cve/cve-2022-28347" rel="noopener" target="_blank">CVE-2022-28347 – django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2022-34265" rel="noopener" target="_blank">CVE-2022-34265 – django, django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2023-31047" rel="noopener" target="_blank">CVE-2023-31047 – django, django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2022-28346" rel="noopener" target="_blank">CVE-2022-28346 – django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2023-24816" rel="noopener" target="_blank">CVE-2023-24816 – ipython</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2021-45115" rel="noopener" target="_blank">CVE-2021-45115 – django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2022-23833" rel="noopener" target="_blank">CVE-2022-23833 – django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2021-31542" rel="noopener" target="_blank">CVE-2021-31542 – django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2023-4622" rel="noopener" target="_blank">CVE-2023-4622 – kernel-devel, kernel</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2023-36053" rel="noopener" target="_blank">CVE-2023-36053 – django, django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2021-45116" rel="noopener" target="_blank">CVE-2021-45116 – django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2023-4207" rel="noopener" target="_blank">CVE-2023-4207 – kernel-devel, kernel</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2023-24580" rel="noopener" target="_blank">CVE-2023-24580 – django, django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2022-36359" rel="noopener" target="_blank">CVE-2022-36359 – django, django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2023-4623" rel="noopener" target="_blank">CVE-2023-4623 – kernel-devel, kernel</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2021-44420" rel="noopener" target="_blank">CVE-2021-44420 – django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2023-4921" rel="noopener" target="_blank">CVE-2023-4921 – kernel-devel, kernel</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2022-22818" rel="noopener" target="_blank">CVE-2022-22818 – django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2021-45452" rel="noopener" target="_blank">CVE-2021-45452 – django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2021-33203" rel="noopener" target="_blank">CVE-2021-33203 – django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2021-32052" rel="noopener" target="_blank">CVE-2021-32052 – django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2021-3281" rel="noopener" target="_blank">CVE-2021-3281 – django</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2023-3772" rel="noopener" target="_blank">CVE-2023-3772 – kernel-devel, kernel</a></li> 
 <li><a href="https://access.redhat.com/security/cve/cve-2023-43804" rel="noopener" target="_blank">CVE-2023-43804 – urllib3</a></li> 
 <li><a href="https://security.snyk.io/vuln/SNYK-PYTHON-DJANGO-5880505" rel="noopener" target="_blank">IN1-PYTHON-DJANGO-5880505 – django, django</a></li> 
 <li><a href="https://security.snyk.io/vuln/SNYK-PYTHON-PYLINT-568073" rel="noopener" target="_blank">IN1-PYTHON-PYLINT-568073 – pylint, pylint</a></li> 
 <li><a href="https://security.snyk.io/vuln/SNYK-PYTHON-DJANGO-5932095" rel="noopener" target="_blank">IN1-PYTHON-DJANGO-5932095 – django, django</a></li> 
 <li><a href="https://security.snyk.io/vuln/SNYK-PYTHON-PYLINT-1089548" rel="noopener" target="_blank">IN1-PYTHON-PYLINT-1089548 – pylint, pylint</a></li> 
 <li><a href="https://security.snyk.io/vuln/SNYK-PYTHON-PYLINT-609883" rel="noopener" target="_blank">IN1-PYTHON-PYLINT-609883 – pylint, pylint</a></li> 
</ul> 
<p>For Amazon Inspector findings, you can refer to parts 1 and 2 of <a href="https://aws.amazon.com/blogs/mt/automate-vulnerability-management-and-remediation-in-aws-using-amazon-inspector-and-aws-systems-manager-part-1/" rel="noopener" target="_blank">Automate vulnerability management and remediation in AWS using Amazon Inspector and AWS Systems Manager</a>.</p> 
<h2>Clean up</h2> 
<p>If you deployed the security finding generator solution by using the <strong>Launch Stack</strong> button in the console or the CloudFormation template <span style="font-family: courier;">security_finding_generator_cfn</span>, do the following to clean up:</p> 
<ol> 
 <li>In the <a href="https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1" rel="noopener" target="_blank">CloudFormation console</a> for the account and Region where you deployed the solution, choose the <strong>SecurityFindingGeneratorStack</strong> stack.</li> 
 <li>Choose the option to <strong>Delete</strong> the stack.</li> 
</ol> 
<p>If you deployed the solution by using the AWS CDK, run the command <span style="font-family: courier;">cdk destroy</span>.</p> 
<blockquote>
 <p><strong>Important</strong>: The solution uses eksctl to provision EKS resources, which deploys additional CloudFormation templates. There are custom resources within the solution that will attempt to delete the provisioned CloudFormation templates for EKS. If there are any issues, you should verify and manually delete the following CloudFormation templates:</p>
</blockquote> 
<ul> 
 <li>eksctl-GuardDuty-Finding-Demo-cluster</li> 
 <li>eksctl-GuardDuty-Finding-Demo-addon-iamserviceaccount-kube-system-aws-node</li> 
 <li>eksctl-GuardDuty-Finding-Demo-nodegroup-ng-&lt;GUID&gt;</li> 
</ul> 
<h2>Conclusion</h2> 
<p>In this blog post, I showed you how to deploy a solution to provision resources in an AWS account to generate security findings. This solution provides a technical framework to conduct periodic simulations within your AWS environment. By having real, rather than simulated, security findings, you can enable your security teams to interact with actual resources and validate existing incident response processes. Having a repeatable mechanism to create security findings also provides your security team the opportunity to develop and test automated incident response capabilities in your AWS environment.</p> 
<p>AWS has multiple services to assist with increasing your organization’s security posture. <a href="https://aws.amazon.com/security-hub/" rel="noopener" target="_blank">Security Hub</a> provides native integration with AWS security services as well as partner services. From Security Hub, you can also implement automation to respond to findings using custom actions as seen in <a href="https://aws.amazon.com/blogs/security/use-security-hub-custom-actions-to-remediate-s3-resources-based-on-macie-discovery-results/" rel="noopener" target="_blank">Use Security Hub custom actions to remediate S3 resources based on Amazon Macie discovery results</a>. In part two of a two-part series, you can learn <a href="https://aws.amazon.com/blogs/security/how-to-investigate-and-take-action-on-security-issues-in-amazon-eks-clusters-with-amazon-detective-part-2/" rel="noopener" target="_blank">how to use Amazon Detective to investigate security findings in EKS clusters</a>. <a href="https://aws.amazon.com/security-lake/" rel="noopener" target="_blank">Amazon Security Lake</a> automatically normalizes and centralizes your data from AWS services such as Security Hub, AWS CloudTrail, <a href="https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html" rel="noopener" target="_blank">VPC Flow Logs</a>, and <a href="https://aws.amazon.com/route53" rel="noopener" target="_blank">Amazon Route 53</a>, as well as <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/custom-sources.html" rel="noopener" target="_blank">custom sources</a> to provide a mechanism for comprehensive analysis and visualizations. </p> 
<p>If you have feedback about this post, submit comments in the Comments section below. If you have questions about this post, start a new thread on the <a href="https://repost.aws/tags/TAbM0xYgFuTsOtxRDmkvrhMA/incident-response" rel="noopener" target="_blank">Incident Response re:Post</a> or <a href="https://console.aws.amazon.com/support/home" rel="noopener" target="_blank">contact AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Author" class="aligncenter size-full wp-image-22628" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2021/10/20/Jonathan-Nguyen-Author.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Jonathan Nguyen</h3> 
  <p>Jonathan is a Principal Security Architect at AWS. His background is in AWS security with a focus on threat detection and incident response. He helps enterprise customers develop a comprehensive AWS security strategy and deploy security solutions at scale, and trains customers on AWS security best practices.</p> 
  <p></p>
 </div> 
</footer>
