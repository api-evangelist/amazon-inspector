---
title: "Improve your security investigations with Detective finding groups visualizations"
url: "https://aws.amazon.com/blogs/security/improve-your-security-investigations-with-detective-finding-groups-visualizations/"
date: "Tue, 29 Aug 2023 15:55:14 +0000"
author: "Rich Vorwaller"
feed_url: "https://aws.amazon.com/blogs/security/category/security-identity-compliance/amazon-inspector/feed/"
---
<p>At AWS, we often hear from customers that they want expanded security coverage for the multiple services that they use on AWS. However, alert fatigue is a common challenge that customers face as we introduce new security protections. The challenge becomes how to operationalize, identify, and prioritize alerts that represent real risk.</p> 
<p>In this post, we highlight recent enhancements to <a href="https://aws.amazon.com/detective/" rel="noopener" target="_blank">Amazon Detective</a> finding groups visualizations. We show you how Detective automatically consolidates multiple security findings into a single security event—called <em>finding groups</em>—and how finding groups visualizations help reduce noise and prioritize findings that present true risk. We incorporate additional services like <a href="https://aws.amazon.com/guardduty/" rel="noopener" target="_blank">Amazon GuardDuty</a>, <a href="https://aws.amazon.com/inspector/" rel="noopener" target="_blank">Amazon Inspector</a>, and <a href="https://aws.amazon.com/security-hub/" rel="noopener" target="_blank">AWS Security Hub</a> to highlight how effective findings groups is at consolidating findings for different AWS security services.</p> 
<h2>Overview of solution</h2> 
<p>This post uses several different services. The purpose is twofold: to show how you can enable these services for broader protection, and to show how Detective can help you investigate findings from multiple services without spending a lot of time sifting through logs or querying multiple data sources to find the root cause of a security event. These are the services and their use cases:</p> 
<ul> 
 <li><strong>GuardDuty</strong> – a threat detection service that continuously monitors your AWS accounts and workloads for malicious activity. If potential malicious activity, such as anomalous behavior, credential exfiltration, or command and control (C2) infrastructure communication is detected, GuardDuty generates detailed security findings that you can use for visibility and remediation. Recently, GuardDuty released the following threat detections for specific services that we’ll show you how to enable for this walkthrough: <a href="https://docs.aws.amazon.com/guardduty/latest/ug/rds-protection.html" rel="noopener" target="_blank">GuardDuty RDS Protection</a>, <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty-eks-runtime-monitoring.html" rel="noopener" target="_blank">EKS Runtime Monitoring</a>, and <a href="https://docs.aws.amazon.com/guardduty/latest/ug/lambda-protection.html" rel="noopener" target="_blank">Lambda Protection</a>.</li> 
 <li><strong>Amazon Inspector</strong> – an automated vulnerability management service that continually scans your AWS workloads for software vulnerabilities and unintended network exposure. Like GuardDuty, Amazon Inspector sends a finding for alerting and remediation when it detects a software vulnerability or a compute instance that’s publicly available.</li> 
 <li><strong>Security Hub</strong> – a cloud security posture management service that performs automated, continuous security best practice checks against your AWS resources to help you identify misconfigurations, and aggregates your security findings from integrated AWS security services.</li> 
 <li><strong>Detective</strong> – a security service that helps you investigate potential security issues. It does this by collecting log data from <a href="https://aws.amazon.com/cloudtrail/" rel="noopener" target="_blank">AWS CloudTrail</a>, <a href="https://aws.amazon.com/vpc/" rel="noopener" target="_blank">Amazon Virtual Private Cloud (Amazon VPC)</a> flow logs, and other services. Detective then uses machine learning, statistical analysis, and graph theory to build a linked set of data called a security behavior graph that you can use to conduct faster and more efficient security investigations.</li> 
</ul> 
<p>The following diagram shows how each service delivers findings along with log sources to Detective.<br /> </p>
<div class="wp-caption aligncenter" id="attachment_30688" style="width: 2370px;">
 <img alt="Figure 1: Amazon Detective log source diagram" class="size-full wp-image-30688" height="1320" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/28/Improve-security-investigations-Detective-1.png" width="2360" />
 <p class="wp-caption-text" id="caption-attachment-30688">Figure 1: Amazon Detective log source diagram</p>
</div>
<p></p> 
<h2>Enable the required services</h2> 
<p>If you’ve already enabled the services needed for this post—GuardDuty, Amazon Inspector, Security Hub, and Detective—skip to the next section. For instructions on how to enable these services, see the following resources:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_settingup.html" rel="noopener" target="_blank">Getting started with GuardDuty</a></li> 
 <li><a href="https://docs.aws.amazon.com/inspector/latest/user/getting_started_tutorial.html" rel="noopener" target="_blank">Getting started with Amazon Inspector</a></li> 
 <li><a href="https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-enable.html" rel="noopener" target="_blank">Enabling Security Hub manually </a></li> 
 <li><a href="https://docs.aws.amazon.com/detective/latest/adminguide/detective-enabling.html" rel="noopener" target="_blank">Enabling Amazon Detective</a></li> 
</ul> 
<p>Each of these services offers a free 30-day trial and provides estimates on charges after your trial expires. You can also use the <a href="https://calculator.aws/#/" rel="noopener" target="_blank">AWS Pricing Calculator</a> to get an estimate.</p> 
<p>To enable the services across multiple accounts, consider using a delegated administrator account in <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a>. With a delegated administrator account, you can automatically enable services for multiple accounts and manage settings for each account in your organization. You can view other accounts in the organization and add them as member accounts, making central management simpler. For instructions on how to enable the services with AWS Organizations, see the following resources:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_organizations.html" rel="noopener" target="_blank">Managing GuardDuty accounts with AWS Organizations</a></li> 
 <li><a href="https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-securityhub.html" rel="noopener" target="_blank">AWS Security Hub and AWS Organizations</a></li> 
 <li><a href="https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-inspector2.html" rel="noopener" target="_blank">Amazon Inspector with AWS Organizations</a></li> 
 <li><a href="https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-detective.html" rel="noopener" target="_blank">Amazon Detective with AWS Organizations</a></li> 
</ul> 
<h2>Enable GuardDuty protections</h2> 
<p>The next step is to enable the latest detections in GuardDuty and learn how Detective can identify multiple threats that are related to a single security event.</p> 
<p>If you’ve already enabled the different GuardDuty protection plans, skip to the next section. If you recently enabled GuardDuty, the protections plans are enabled by default, except for EKS Runtime Monitoring, which is a two-step process.</p> 
<p>For the next steps, we use the delegated administrator account in GuardDuty to make sure that the protection plans are enabled for each AWS account. When you use GuardDuty (or Security Hub, Detective, and Inspector) with <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a>, you can designate an account to be the delegated administrator. This is helpful so that you can configure these security services for multiple accounts at the same time. For instructions on how to enable a delegated administrator account for GuardDuty, see <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_organizations.html" rel="noopener" target="_blank">Managing GuardDuty accounts with AWS Organizations</a>.</p> 
<h3>To enable EKS Protection</h3> 
<ol> 
 <li>Sign in to the <a href="https://console.aws.amazon.com/guardduty/" rel="noopener" target="_blank">GuardDuty console</a> using the delegated administrator account, choose <strong>Protection plans</strong>, and then choose <strong>EKS Protection</strong>.</li> 
 <li>In the <strong>Delegated administrator</strong> section, choose<strong> Edit </strong>and then choose <strong>Enable </strong>for each scope or protection. For this post, select <strong>EKS Audit Log Monitoring</strong>, <strong>EKS Runtime Monitoring</strong>, and <strong>Manage agent automatically</strong>, as shown in Figure 2. For more information on each feature, see the following resources: 
  <ul> 
   <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty-eks-audit-log-monitoring.html" rel="noopener" target="_blank">EKS Audit Log Monitoring</a></li> 
   <li><a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty-eks-runtime-monitoring.html" rel="noopener" target="_blank">EKS Runtime Monitoring</a></li> 
   <p></p>
   <div class="wp-caption aligncenter" id="attachment_30689" style="width: 976px;">
    <img alt="Figure 2: Enable EKS Runtime Monitoring" class="size-full wp-image-30689" height="449" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/28/Improve-security-investigations-Detective-2.png" width="966" />
    <p class="wp-caption-text" id="caption-attachment-30689">Figure 2: Enable EKS Runtime Monitoring</p>
   </div> 
  </ul> </li> 
 <li>To enable these protections for current accounts, in the <strong>Active member accounts </strong>section, choose<strong> Edit </strong>and <strong>Enable </strong>for each scope of protection.</li> 
 <li>To enable these protections for new accounts, in the <strong>New account default configuration </strong>section, choose <strong>Edit </strong>and<strong> Enable </strong>for each scope of protection.</li> 
</ol> 
<h3>To enable RDS Protection</h3> 
<p>The next step is to enable RDS Protection. GuardDuty RDS Protection works by analysing RDS login activity for potential threats to your <a href="https://aws.amazon.com/rds/aurora/" rel="noopener" target="_blank">Amazon Aurora</a> databases (MySQL-Compatible Edition and Aurora PostgreSQL-Compatible Editions). Using this feature, you can identify potentially suspicious login behavior and then use Detective to investigate CloudTrail logs, VPC flow logs, and other useful information around those events.</p> 
<ol> 
 <li>Navigate to the <strong>RDS Protection </strong>menu and under<strong> Delegated administrator (this account)</strong>, select <strong>Enable</strong> and <strong>Confirm</strong>.</li> 
 <li>In the <strong>Enabled for</strong> section, select <strong>Enable all</strong> if you want RDS Protection enabled on all of your accounts. If you want to select a specific account, choose <strong>Manage Accounts</strong> and then select the accounts for which you want to enable RDS Protection. With the accounts selected, choose <strong>Edit Protection Plans</strong>, <strong>RDS Login Activity</strong>, and<strong> Enable for X selected account.</strong></li> 
 <li>(Optional) For new accounts, turn on <strong>Auto-enable RDS Login Activity Monitoring for new member accounts as they join your organization</strong>.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_30690" style="width: 1132px;">
 <img alt="Figure 2: Enable EKS Runtime Monitoring" class="size-full wp-image-30690" height="369" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/28/Improve-security-investigations-Detective-3.png" width="1122" />
 <p class="wp-caption-text" id="caption-attachment-30690">Figure 2: Enable EKS Runtime Monitoring</p>
</div> 
<h3>To enable Lambda Protection</h3> 
<p>The final step is to enable Lambda Protection. Lambda Protection helps detect potential security threats during the invocation of <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a> functions. By monitoring network activity logs, GuardDuty can generate findings when Lambda functions are involved with malicious activity, such as communicating with command and control servers.</p> 
<ol> 
 <li>Navigate to the <strong>Lambda Protection </strong>menu and under <strong>Delegated administrator (this account)</strong>, select <strong>Enable</strong> and <strong>Confirm</strong>.</li> 
 <li>In the <strong>Enabled for</strong> section, select <strong>Enable all</strong> if you want Lambda Protection enabled on all of your accounts. If you want to select a specific account, choose <strong>Manage Accounts</strong> and select the accounts for which you want to enable RDS Protection. With the accounts selected, choose <strong>Edit Protection Plans</strong>, <strong>Lambda Network Activity Monitoring</strong>, and<strong> Enable for X selected account.</strong></li> 
 <li>(Optional) For new accounts, turn on <strong>Auto-enable Lambda Network Activity Monitoring for new member accounts as they join your organization</strong>.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_30691" style="width: 1113px;">
 <img alt="Figure 4: Enable Lambda Network Activity Monitoring" class="size-full wp-image-30691" height="337" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/28/Improve-security-investigations-Detective-4.png" width="1103" />
 <p class="wp-caption-text" id="caption-attachment-30691">Figure 4: Enable Lambda Network Activity Monitoring</p>
</div> 
<p>Now that you’ve enabled these new protections, GuardDuty will start monitoring EKS audit logs, EKS runtime activity, RDS login activity, and Lambda network activity. If GuardDuty detects suspicious or malicious activity for these log sources or services, it will generate a finding for the activity, which you can review in the GuardDuty console. In addition, you can automatically forward these findings to Security Hub for consolidation, and to Detective for security investigation.</p> 
<h2>Detective data sources</h2> 
<p>If you have Security Hub and other AWS security services such as GuardDuty or Amazon Inspector enabled, findings from these services are forwarded to Security Hub. With the exception of sensitive data findings from <a href="https://aws.amazon.com/macie/" rel="noopener" target="_blank">Amazon Macie</a>, you’re automatically opted in to other AWS service integrations when you enable Security Hub. For the full list of services that forward findings to Security Hub, see <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-internal-providers.html" rel="noopener" target="_blank">Available AWS service integrations</a>.</p> 
<p>With each service enabled and forwarding findings to Security Hub, the next step is to enable the data source in Detective called <strong>AWS security findings</strong>, which are the findings forwarded to Security Hub. Again, we’re going to use the delegated administrator account for these steps to make sure that AWS security findings are being ingested for your accounts.</p> 
<h3>To enable AWS security findings</h3> 
<ol> 
 <li>Sign in to the <a href="https://console.aws.amazon.com/detective/" rel="noopener" target="_blank">Detective console</a> using the delegated administrator account and navigate to <strong>Settings</strong> and then <strong>General</strong>.</li> 
 <li>Choose <strong>Optional source packages, Edit</strong>, select <strong>AWS security findings</strong>, and then choose <strong>Save</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_30702" style="width: 604px;">
   <img alt="Figure 5: Enable AWS security findings" class="size-full wp-image-30702" height="279" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/28/Improve-security-investigations-Detective-5b.png" width="594" />
   <p class="wp-caption-text" id="caption-attachment-30702">Figure 5: Enable AWS security findings</p>
  </div> </li> 
</ol> 
<p>When you enable Detective, it immediately starts creating a security behavior graph for AWS security findings to build a linked dataset between findings and entities, such as RDS login activity from Aurora databases, EKS runtime activity, and suspicious network activity for Lambda functions. For GuardDuty to detect potential threats that affect your database instances, it first needs to undertake a learning period of up to two weeks to establish a baseline of normal behavior. For more information, see <a href="https://docs.aws.amazon.com/guardduty/latest/ug/rds-protection.html" rel="noopener" target="_blank">How RDS Protection uses RDS login activity monitoring</a>. For the other protections, after suspicious activity is detected, you can start to see findings in both GuardDuty and Security Hub consoles. This is where you can start using Detective to better understand which findings are connected and where to prioritize your investigations.</p> 
<h2>Detective behavior graph</h2> 
<p>As Detective ingests data from GuardDuty, Amazon Inspector, and Security Hub, as well as CloudTrail logs, VPC flow logs, and <a href="https://aws.amazon.com/eks/" rel="noopener" target="_blank">Amazon Elastic Kubernetes Service (Amazon EKS)</a> audit logs, it builds a behavior graph database. <a href="https://aws.amazon.com/nosql/graph/" rel="noopener" target="_blank">Graph databases</a> are purpose-built to store and navigate relationships. Relationships are first-class citizens in graph databases, which means that they’re not computed out-of-band or by interfering with relationships through querying foreign keys. Because Detective stores information on relationships in your graph database, you can effectively answer questions such as “are these security findings related?”. In Detective, you can use the search menu and profile panels to view these connections, but a quicker way to see this information is by using finding groups visualizations.</p> 
<h3>Finding groups visualizations</h3> 
<p>Finding groups extract additional information out of the behavior graph to highlight findings that are highly connected. Detective does this by running several machine learning algorithms across your behavior graph to identify related findings and then statically weighs the relationships between those findings and entities. The result is a finding group that shows GuardDuty and Amazon Inspector findings that are connected, along with entities like <a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a> instances, AWS accounts, and <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> roles and sessions that were impacted by these findings. With finding groups, you can more quickly understand the relationships between multiple findings and their causes because you don’t need to connect the dots on your own. Detective automatically does this and presents a visualization so that you can see the relationships between various entities and findings.</p> 
<h3>Enhanced visualizations</h3> 
<p>Recently, we released several enhancements to finding groups visualizations to aid your understanding of security connections and root causes. These enhancements include:</p> 
<ul> 
 <li><strong>Dynamic legend</strong> – the legend now shows icons for entities that you have in the finding group instead of showing all available entities. This helps reduce noise to only those entities that are relevant to your investigation.</li> 
 <li><strong>Aggregated evidence and finding icons</strong> – these icons provide a count of similar evidence and findings. Instead of seeing the same finding or evidence repeated multiple times, you’ll see one icon with a counter to help reduce noise.</li> 
 <li><strong>More descriptive side panel information</strong> – when you choose a finding or entity, the side panel shows additional information, such as the service that identified the finding and the finding title, in addition to the finding type, to help you understand the action that invoked the finding.</li> 
 <li><strong>Label titles </strong>– you can now turn on or off titles for entities and findings in the visualization so that you don’t have to choose each to get a summary of what the different icons mean.</li> 
</ul> 
<h3>To use the finding groups visualization </h3> 
<ol> 
 <li>Open the <a href="https://console.aws.amazon.com/detective/" rel="noopener" target="_blank">Detective console</a>, choose <strong>Summary</strong>, and then choose <strong>View all finding groups</strong>.</li> 
 <li>Choose the title of an available finding group and scroll down to <strong>Visualization</strong>.</li> 
 <li>Under the <strong>Select layout</strong> menu, choose one of the layouts available, or choose and drag each icon to rearrange the layout according to how you’d like to see connections.</li> 
 <li>For a complete list of involved entities and involved findings, scroll down below the visualization.</li> 
</ol> 
<p>Figure 6 shows an example of how you can use finding groups visualization to help identify the root cause of findings quickly. In this example, an IAM role was connected to newly observed geolocations, multiple GuardDuty findings detected malicious API calls, and there were newly observed user agents from the IAM session. The visualization can give you high confidence that the IAM role is compromised. It also provides other entities that you can search against, such as the IP address, S3 bucket, or new user agents.</p> 
<div class="wp-caption aligncenter" id="attachment_30693" style="width: 1386px;">
 <img alt="Figure 6: Finding groups visualization" class="size-full wp-image-30693" height="565" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/28/Improve-security-investigations-Detective-6.png" width="1376" />
 <p class="wp-caption-text" id="caption-attachment-30693">Figure 6: Finding groups visualization</p>
</div> 
<p>Now that you have the new GuardDuty protections enabled along with the data source of AWS security findings, you can use finding groups to more quickly visualize which IAM sessions have had multiple findings associated with unauthorized access, or which EC2 instances are publicly exposed with a software vulnerability and active GuardDuty finding—these patterns can help you determine if there is an actual risk.</p> 
<h2>Conclusion</h2> 
<p>In this blog post, you learned how to enable new GuardDuty protections and use Detective, finding groups, and visualizations to better identify, operationalize, and prioritize AWS security findings that represent real risk. We also highlighted the new enhancements to visualizations that can help reduce noise and provide summaries of detailed information to help reduce the time it takes to triage findings. If you’d like to see an investigation scenario using Detective, watch the video <a href="https://youtu.be/Rz8MvzPfTZA" rel="noopener" target="_blank">Amazon Detective Security Scenario Investigation</a>.</p> 
<p>If you have feedback about this post, submit comments in the <strong>Comments</strong> section below. You can also start a new thread on <a href="https://repost.aws/tags/TAUrK2r73PTHyirNVS4hKn6w/amazon-detective" rel="noopener" target="_blank">Amazon Detective re:Post</a> or <a href="https://console.aws.amazon.com/support/home" rel="noopener" target="_blank">contact AWS Support</a>.</p> 
<p><strong>Want more AWS Security how-to content, news, and feature announcements? Follow us on <a href="https://twitter.com/AWSsecurityinfo" rel="noopener noreferrer" target="_blank" title="Twitter">Twitter</a>.</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Rich Vorwaller" class="aligncenter size-full wp-image-29083" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/04/04/richvor.png" width="120" />
  </div> 
  <h3 class="lb-h4">Rich Vorwaller</h3> 
  <p>Rich is a Principal Product Manager of Amazon Detective. He came to AWS with a passion for walking backwards from customer security problems. AWS is a great place for innovation, and Rich is excited to dive deep on how customers are using AWS to strengthen their security posture in the cloud. In his spare time, Rich loves to read, travel, and perform a little bit of amateur stand-up comedy.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Nicholas Doropoulos" class="aligncenter size-full wp-image-27278" height="2400" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2022/09/16/doronik.jpg" width="1800" />
  </div> 
  <h3 class="lb-h4">Nicholas Doropoulos</h3> 
  <p>Nicholas is an AWS Cloud Security Engineer, a bestselling Udemy instructor, and a subject matter expert in AWS Shield, Amazon GuardDuty, AWS IAM, and AWS Certificate Manager. Outside work, he enjoys spending time with his wife and their beautiful baby son. </p>
 </div> 
</footer>
