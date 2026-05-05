---
title: "Amazon Inspector suppression rules best practices for AWS Organizations"
url: "https://aws.amazon.com/blogs/security/amazon-inspector-suppression-rules-best-practices-for-aws-organizations/"
date: "Tue, 05 Nov 2024 19:57:57 +0000"
author: "Mojgan Toth"
feed_url: "https://aws.amazon.com/blogs/security/category/security-identity-compliance/amazon-inspector/feed/"
---
<p>Vulnerability management is a vital part of network, application, and infrastructure security, and its goal is to protect an organization from inadvertent access and exposure of sensitive data and infrastructure. As part of vulnerability management, organizations typically perform a risk assessment to determine which vulnerabilities pose the greatest risk, evaluate their impact on business goals and overall strategy, and assess the relevant regulatory requirements.</p> 
<p>In this post, we explain how to use mechanisms to appropriately prioritize vulnerabilities across your accounts in <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a>. We discuss how to apply tags to resources so that you can use risk-based prioritization of <a href="https://aws.amazon.com/inspector/" rel="noopener" target="_blank">Amazon Inspector</a> findings in your environment, and we talk about some best practices for using suppression rules to suppress less-critical findings in Amazon Inspector, at scale. We also emphasize practices to create a culture of continuous vulnerability management.</p> 
<h2 id="vulnerability-management-with-amazon-inspector">Vulnerability management with Amazon Inspector</h2> 
<p><a href="https://aws.amazon.com/inspector/" rel="noopener" target="_blank">Amazon Inspector</a> is a vulnerability management service that continuously scans your <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> workloads for software vulnerabilities and unintended network exposure. Amazon Inspector automatically discovers and scans running <a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a> instances, container images in <a href="https://aws.amazon.com/ecr/" rel="noopener" target="_blank">Amazon Elastic Container Registry (Amazon ECR)</a>, and <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a> functions.</p> 
<p>Amazon Inspector creates a <em>finding</em> when it discovers a software vulnerability or an unintended network exposure. A finding describes the vulnerability, identifies the affected resource, rates the severity of the vulnerability, and provides remediation guidance. You can create <em>suppression rules</em> in Amazon Inspector to suppress findings that are less critical, so that you can focus on higher-priority findings.</p> 
<h2 id="best-practices-for-vulnerability-management-in-aws-organizations">Best practices for vulnerability management in AWS Organizations</h2> 
<p>We recommend that you use the best practices discussed in this section to ease the task of resolving thousands of vulnerability findings in your organization in <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a>.</p> 
<h3 id="best-practice-1-set-up-a-delegated-admin">Best practice #1: Set up a delegated admin</h3> 
<p>You can use Amazon Inspector to manage vulnerability scanning for multiple AWS accounts in an organization. To do this, the AWS Organizations management account needs to designate an account as the delegated administrator account for Amazon Inspector. The delegated administrator account has centralized control over the Amazon Inspector deployment, which allows for more efficient and effective management of security monitoring tasks across the multiple accounts within AWS Organizations. These tasks include activating or deactivating scans for member accounts, aggregating findings by AWS Region, viewing aggregated finding data from the entire organization, and creating and managing suppression rules.</p> 
<p>Amazon Inspector is a regional service, meaning you must designate a delegated administrator, add member accounts, and activate scan types in each AWS Region you want to use Amazon Inspector in. When you’re setting up your delegated administrator account, be aware of the following factors:</p> 
<ul> 
 <li>Delegated admins can create and manage Center for Internet Security (CIS) scan configurations for the accounts in the organization, except for any scan configurations that are created by member accounts.</li> 
 <li>In a multi-account setup, only delegated admins are able to set up scan mode configuration for the complete organization.</li> 
 <li>You can use Amazon Inspector to perform on-demand and targeted assessments against OS-level CIS configuration benchmarks for Amazon EC2 instances across your organization.</li> 
</ul> 
<h3 id="best-practice-2-manage-findings-at-scale-with-suppression-rules">Best practice #2: Manage findings at scale with suppression rules</h3> 
<p>There can be thousands of specific common vulnerabilities and exposures (CVEs) or Amazon Resource Names (ARNs) in the findings across your accounts, and therefore managing these findings at scale with proper suppression rules will lead you towards achieving successful vulnerability management.</p> 
<p>A <em>suppression rule</em> is a set of criteria consisting of a filter attribute paired with a value, which is used to filter findings by automatically archiving new findings that match the specified criteria. You can <a href="https://docs.aws.amazon.com/inspector/latest/user/findings-managing-supression-rules.html" rel="noopener" target="_blank">create suppression rules</a> to exclude vulnerabilities you don’t intend to act on, so that you can prioritize your most important findings. Suppression rules don’t impact the finding itself and don’t prevent Amazon Inspector from generating a finding. Suppression rules are only used to filter your list of findings and make it easier for you to navigate and prioritize them.</p> 
<p>Some helpful filters that you can use in suppression rules are <strong>Resource tag</strong>, <strong>Resource type</strong>, <strong>Severity</strong>, <strong>Vulnerability ID</strong>, and <strong>Amazon Inspector score</strong>. For example, you can categorize the findings based on severity levels (Critical, High, Medium, Low, Informational, and Untriaged). To learn more about how Amazon Inspector determines a severity rating for each finding, see <a href="https://docs.aws.amazon.com/inspector/latest/user/findings-understanding-severity.html" rel="noopener" target="_blank">Understanding severity levels for your Amazon Inspector findings</a>.</p> 
<p>You can navigate through the findings in Amazon Inspector based on different categories such as vulnerability, account, or instance. On the <strong>All findings</strong> page in Amazon Inspector, if you select a CVE ID, you can view details for the affected resources and the individual AWS account IDs, as shown in Figure 1. This can help you choose filter criteria to use in suppression rules.</p> 
<div class="wp-caption aligncenter" id="attachment_36301" style="width: 1432px;">
 <img alt="Figure 1: Amazon Inspector findings and severity levels" class="size-full wp-image-36301" height="583" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/10/28/Amazon-Inspector-suppression-rules.png" style="border: 1px solid #bebebe;" width="1422" />
 <p class="wp-caption-text" id="caption-attachment-36301">Figure 1: Amazon Inspector findings and severity levels</p>
</div> 
<p>You manage suppression rules at the organization level, and the rules apply to all the member accounts. If Amazon Inspector generates a new finding that matches a suppression rule, the service automatically sets the status of the finding to <strong>Suppressed</strong>. The findings that match suppression rule criteria don’t appear in the findings list, by default. Therefore, the suppressed findings don’t impact your service quotas. Member accounts inherit the suppression rules from the delegated administrator. The delegated administrator account is limited to 500 rules (per Region), and this is a hard limit.</p> 
<p>Keep in mind that member accounts in an organization cannot create or manage suppression rules. Only standalone accounts and Amazon Inspector delegated administrators can create and manage suppression rules. So, if there is a member account within an organization that needs independent management of its own suppression rules, then the account owner needs to activate Amazon Inspector separately in their account.</p> 
<h3 id="best-practice-3-suppress-findings-based-on-amazon-inspector-score">Best practice #3: Suppress findings based on Amazon Inspector score</h3> 
<p>Because your time is limited and the volume of security vulnerability findings can be large, especially in bigger organizations, you need to be able to quickly identify and respond to the vulnerabilities that pose the greatest risk to your organization.</p> 
<p>One quick approach to suppressing findings is to use the <a href="https://docs.aws.amazon.com/inspector/latest/user/findings-understanding-score.html" rel="noopener" target="_blank">Amazon Inspector score</a>. Amazon Inspector examines the security metrics that compose the <a href="https://nvd.nist.gov/vuln" rel="noopener" target="_blank">National Vulnerability Database (NVD)</a> Common Vulnerability Scoring System (CVSS) base score for a vulnerability, adjusts them according to your compute environment, and then produces a numerical score from 1 to 10 that reflects the vulnerability’s severity.</p> 
<p>The NVD/CVSS score is a composition of security metrics, such as threat complexity, exploit code maturity, and privileges required, but it is not a measure of risk.</p> 
<p>Be cautious not to over-suppress your findings. Over-suppressing findings can inadvertently expose applications and systems to unmitigated security risks. It’s important to maintain a careful, measured approach when applying suppression rules. Maintaining visibility into the true risk profile for each finding is essential for proactive, comprehensive vulnerability management.</p> 
<h3 id="best-practice-4-use-tags-to-enable-risk-based-prioritization">Best practice #4: Use tags to enable risk-based prioritization</h3> 
<p>For a scalable vulnerability management solution, it’s important to have a strategy for tagging resources appropriately across your accounts.</p> 
<p>To prioritize vulnerabilities, first you need to understand and assess each resource’s risk level so that you can tag it properly. Proper tagging enables you to use <em>risk-based prioritization</em><strong>.</strong> This means that when you evaluate findings, you look at factors such as the risk level of the resource, the severity of the vulnerability, and the impact of the vulnerability on your organization’s environment so that you can focus on the critical vulnerabilities first. This seems like an obvious recommendation, but its importance cannot be overstated. In the cloud, you have to understand and protect everything you build.&nbsp;Asset mapping must include relationship mapping to understand the implications and risk paths of potential security events.</p> 
<p>The priority for remediating cloud resource issues depends on the level of exposure of the resource. Resources in public subnets should generally be prioritized over those in private subnets. Resources running in production environments should be prioritized over those in development and test environments.</p> 
<p>The prioritization also depends on factors like firewall rules, <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> policies, service control policies, and security groups. Resources with more open internet access through various ports and protocols have increased scope for issues like denial of service (DoS), distributed denial of service (DDoS), spoofing, malware, and ransomware, compared to resources with tight access restrictions.</p> 
<h3 id="best-practice-5-base-suppression-rules-on-proper-resource-tags">Best practice #5: Base suppression rules on proper resource tags</h3> 
<p>In complex multi-account environments, it can be challenging to centrally manage suppression rules by using resource IDs, subnet IDs, or VPC IDs, because these values are specific to individual accounts and change over time with new deployments or modifications. This makes it difficult to keep the suppression rules up to date. Here, we review how you can take advantage of risk-based prioritization based on tags (best practice #4) along with the Amazon Inspector score to effectively manage, prioritize, and track your findings.</p> 
<p>The following example provides a suggested tagging strategy that you can use across your AWS Cloud resources in your organization for the purpose of vulnerability management:</p> 
<p>EnvironmentName, RiskExposureScore</p> 
<p>With this tagging strategy, you create prioritization through the suppression of rules across the environment and dismiss the findings that you need to postpone or ignore so that you can focus on the high-value findings. You can also create different rules for different environments with different risk factors. For example, you might want to suppress findings for resources that have low risk exposure levels, are in your non-production environment, and are within these severity levels: Informational, Untriaged, Low, or Medium. You can also take advantage of the <strong>Resource Tag</strong> field when you create or export a report, to filter out the expected findings.</p> 
<p>In the following table, we provide an example for an AWS Cloud environment that has three main divisions of accounts: Prod, Dev, and Sandbox. We’ve suppressed rules for different severity levels based on the possible risks, exposure level, and how critical the workloads are. In our example, we used a RiskExposureScore of 1, 2, and 3 to be equivalent to low, medium, and high. In other words, RiskExposureScore 1 is for the workloads that are less sensitive or have little to no internet exposure, while RiskExposureScore 3 is for sensitive or critical workload that have internet exposure, are less protected, or have higher possible security risks due to their configuration or poor cyber hygiene.</p> 
<table border="1" style="margin-left: 45px;" width="0"> 
 <colgroup> 
  <col style="width: 27%;" /> 
  <col style="width: 29%;" /> 
  <col style="width: 24%;" /> 
  <col style="width: 18%;" /> 
 </colgroup> 
 <thead> 
  <tr> 
   <th><strong>EnvironmentName</strong></th> 
   <th><strong>RiskExposureScore</strong></th> 
   <th><strong>Severity</strong></th> 
   <th><strong>Suppressed</strong></th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td>Prod</td> 
   <td>1</td> 
   <td>Medium, Low, Informational, Untriaged</td> 
   <td>Yes</td> 
  </tr> 
  <tr> 
   <td>Prod</td> 
   <td>2</td> 
   <td>Low, Informational, Untriaged</td> 
   <td>Yes</td> 
  </tr> 
  <tr> 
   <td>Prod</td> 
   <td>3</td> 
   <td>Informational, Untriaged</td> 
   <td>Yes</td> 
  </tr> 
  <tr> 
   <td>Dev</td> 
   <td>1,2</td> 
   <td>Medium, Low, Informational, Untriaged</td> 
   <td>Yes</td> 
  </tr> 
  <tr> 
   <td>Dev</td> 
   <td>3</td> 
   <td>Low, Informational, Untriaged</td> 
   <td>Yes</td> 
  </tr> 
  <tr> 
   <td>Sandbox</td> 
   <td>1,2</td> 
   <td>Critical, High, Medium, Low, Informational, Untriaged</td> 
   <td>Yes</td> 
  </tr> 
  <tr> 
   <td>Sandbox</td> 
   <td>3</td> 
   <td>High, Medium, Low, Informational, Untriaged</td> 
   <td>Yes</td> 
  </tr> 
 </tbody> 
</table> 
<p>For this specific example, we would like to keep the vulnerability findings that have a severity of High or Critical for the resources in the Prod and Dev accounts, but define different suppression rules across other resources depending on their risk exposure level. We would also like to suppress the majority of the vulnerability findings in the Sandbox accounts, because we don’t have any critical workloads on those accounts. You can use this example as a model for configuring the suppression rules across your environment to prioritize vulnerability findings according to your needs. Also note that you can change, modify, or re-evaluate your suppression rules as you work on remediation, and it’s a best practice to do so as a continuous process.</p> 
<h3 id="best-practice-6-integrate-amazon-inspector-with-aws-security-hub">Best practice #6: Integrate Amazon Inspector with AWS Security Hub</h3> 
<p>You can <a href="https://docs.aws.amazon.com/inspector/latest/user/securityhub-integration.html" rel="noopener" target="_blank">integrate</a> Amazon Inspector with <a href="https://aws.amazon.com/security-hub/" rel="noopener" target="_blank">AWS Security Hub</a> to send findings from Amazon Inspector to Security Hub, and Security Hub can include these findings in its analysis of your security posture. Amazon Inspector findings that match suppression rules are automatically suppressed and won’t appear in the Security Hub console.</p> 
<h3 id="best-practice-7-re-evaluate-your-suppression-rules-on-a-regular-basis">Best practice #7: Re-evaluate your suppression rules on a regular basis</h3> 
<p>The key to an up-to-date security posture and healthy cloud environment is maintaining and adapting your vulnerability management approach as the threat landscape evolves. Here, we’ve highlighted some practices to focus on:</p> 
<ul> 
 <li>Regularly revisit and re-evaluate your suppressed vulnerability detection rules. Vulnerabilities and threats are constantly evolving, so what you suppressed previously might need to be re-enabled.</li> 
 <li>View vulnerability management as a continuous, iterative process, not a static procedure. Regularly assess, update, and adapt security controls to address emerging risks in real time.</li> 
 <li>Emphasize the importance of continuous monitoring and response, not just initial remediation. Vulnerabilities need to be addressed holistically through the entire lifecycle.</li> 
 <li>Foster a culture of security awareness and responsiveness throughout your organization. Everyone should be engaged in identifying and mitigating vulnerabilities on an ongoing basis.</li> 
 <li>Make sure that your vulnerability management program aligns with relevant compliance or regulatory requirements (for example, <a href="https://www.pcisecuritystandards.org/" rel="noopener" target="_blank">PCI-DSS</a>, <a href="https://www.hhs.gov/hipaa/index.html" rel="noopener" target="_blank">HIPAA</a>, or <a href="https://www.nist.gov/cyberframework" rel="noopener" target="_blank">NIST CSF</a>).</li> 
</ul> 
<h2 id="conclusion">Conclusion</h2> 
<p>In this post, we covered how you can effectively prioritize Amazon Inspector findings at scale across your organization’s AWS infrastructure by using suppression rules and applying risk-based prioritization. We also discussed how to use resource tagging as an effective strategy for prioritizing the remediation of Amazon Inspector findings. For additional blog posts related to Amazon Inspector, see the <a href="https://aws.amazon.com/blogs/security/tag/amazon-inspector/page/2/" rel="noopener" target="_blank">AWS Security Blog</a>.</p> 
<p>If you have feedback about this post, submit comments in the <strong>Comments</strong> section below. </p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Mojgan Toth" class="aligncenter size-full wp-image-36299" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/10/28/Mojgan-Toth.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Mojgan Toth</h3> 
  <p>Mojgan is a Senior Technical Account Manager at AWS. She proactively helps public sector customers with strategic technical guidance, solutions, and AWS Cloud best practices. She loves putting together solutions around the Well-Architected Framework. Outside of work, she loves cooking, painting, and spending time with her family, especially her three little boys. They love outdoor activities such as bike rides and hikes.</p> 
  <p></p>
 </div> 
</footer>
