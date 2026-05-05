---
title: "Amazon Inspector detects over 150,000 malicious packages linked to token farming campaign"
url: "https://aws.amazon.com/blogs/security/amazon-inspector-detects-over-150000-malicious-packages-linked-to-token-farming-campaign/"
date: "Fri, 14 Nov 2025 00:15:12 +0000"
author: "Chi Tran"
feed_url: "https://aws.amazon.com/blogs/security/category/security-identity-compliance/amazon-inspector/feed/"
---
<div class="Page-articleBody"> 
 <div class="RichTextArticleBody RichTextBody"> 
  <p><span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/inspector/" rel="noopener" target="_blank">Amazon Inspector</a></span> security researchers have identified and reported over 150,000 packages linked to a coordinated <span class="LinkEnhancement"><a class="Link" href="http://tea.xyz/" rel="noopener" target="_blank">tea.xyz</a></span> token farming campaign in the <span class="LinkEnhancement"><a class="Link" href="https://www.npmjs.com/" rel="noopener" target="_blank">npm</a></span> registry. This is one of the largest package flooding incidents in open source registry history, and represents a defining moment in supply chain security, far surpassing the initial <span class="LinkEnhancement"><a class="Link" href="https://www.sonatype.com/blog/devs-flood-npm-with-10000-packages-to-reward-themselves-with-tea-tokens" rel="noopener" target="_blank">15,000 packages reported by Sonatype researchers</a></span> in April 2024. Through a combination of advanced rule-based detection and AI, the research team uncovered a self-replicating attack pattern where threat actors automatically generate and publish packages to earn cryptocurrency rewards without user awareness, revealing how the campaign has expanded exponentially since its initial identification.</p> 
  <p>This incident demonstrates both the evolving nature of threats where financial incentives drive registry pollution at unprecedented scale, and the critical importance of industry-community collaboration in defending the software supply chain. The Amazon Inspector team’s capability to detect subtle, non-traditional threats through innovative detection methodologies, combined with rapid collaboration with the <span class="LinkEnhancement"><a class="Link" href="https://openssf.org/" rel="noopener" target="_blank">Open Source Security Foundation (OpenSSF)</a></span> to assign malicious package identifiers (MAL-IDs) and coordinate response, provides a blueprint for how security organizations can respond swiftly and effectively to emerging attack vectors. As the open source community continues to grow, this case serves as both a warning that new threats will emerge wherever financial incentives exist, and a demonstration of how collaborative defense can help address supply chain attacks.</p> 
  <div class="RichTextHeading"> 
   <h2>Detection</h2> 
   <p></p>
  </div> 
  <p>On October 24, 2025, Amazon Inspector security researchers deployed a new detection rule—paired with AI—to identify additional suspicious package patterns in the npm registry. Within days, the system began flagging packages linked to the <span class="LinkEnhancement"><a class="Link" href="http://tea.xyz/" rel="noopener" target="_blank">tea.xyz</a></span> protocol—a blockchain-based system designed to reward open source developers.</p> 
  <p>By November 7, the researchers flagged thousands of packages and began investigating what appeared to be a coordinated campaign. The next day, after validating the evaluation results and analyzing the patterns, they reached out to OpenSSF to share their findings and coordinate a response. With OpenSSF’s review and alignment, Amazon Inspector security researchers began systematically submitting discovered packages to the <span class="LinkEnhancement"><a class="Link" href="https://github.com/ossf/malicious-packages" rel="noopener" target="_blank">OpenSSF malicious packages repository,</a></span> with each package receiving a MAL-ID within 30 minutes. The operation continued through November 12, ultimately uncovering over 150,000 malicious packages.</p> 
  <p>Here’s what the investigation revealed:</p> 
  <ul class="rte2-style-ul" id="rte-296d6961-c0e7-11f0-a18c-39aa29de08dc"> 
   <li>Over 150,000 packages linked to the <span class="LinkEnhancement"><a class="Link" href="http://tea.xyz/" rel="noopener" target="_blank">tea.xyz</a></span> token farming campaign</li> 
   <li>Self-replicating automation that creates packages without legitimate functionality</li> 
   <li>Systematic inclusion of tea.yaml files that link packages to blockchain wallet addresses</li> 
   <li>Coordinated publishing activity across multiple developer accounts</li> 
  </ul> 
  <p>Unlike traditional malware, these packages do not contain overtly malicious code. Instead, they exploit the tea.xyz reward mechanism by artificially inflating package metrics through automated replication and dependency chains, allowing threat actors to extract financial benefits from the open source community.</p> 
  <div class="RichTextHeading"> 
   <h2>Token farming as a new attack vector</h2> 
   <p></p>
  </div> 
  <p>This campaign represents a concerning evolution in supply chain security. Although the packages might not steal credentials or deploy ransomware, they pose significant risks:</p> 
  <ul id="rte-63925c40-c0e7-11f0-a18c-39aa29de08dc"> 
   <li><b>Registry pollution</b> – The npm registry is flooded with low-quality, non-functional packages that obscure legitimate software and degrade trust in the open source community.</li> 
   <li><b>Resource exploitation</b> – Registry infrastructure, bandwidth, and storage are consumed by packages created solely for financial gain rather than genuine contribution.</li> 
   <li><b>Precedent for abuse</b> – The success of this campaign could inspire similar exploitation of other reward-based systems, normalizing automated package generation for financial gain.</li> 
   <li><b>Supply chain risk</b> – Even packages that seem benign can add unnecessary dependencies, potentially introducing unexpected behaviors or creating confusion in dependency resolution.</li> 
  </ul> 
  <div class="RichTextHeading"> 
   <h2>Collaboration with OpenSSF: rapid response</h2> 
   <p></p>
  </div> 
  <p>The collaboration between Amazon Inspector security researchers and OpenSSF led to swift action and benefits such as the following:</p> 
  <ul id="rte-63925c41-c0e7-11f0-a18c-39aa29de08dc"> 
   <li><b>Immediate threat intelligence sharing</b> – The researchers’ findings were shared with OpenSSF’s malicious packages repository, providing the community with comprehensive threat data.</li> 
   <li><b>MAL-ID assignment</b> – OpenSSF rapidly assigned MAL-IDs to the detected packages, enabling community-wide blocking and remediation. Average time of assignment was 30 minutes.</li> 
   <li><b>Coordinated disclosure</b> – Both organizations worked together to inform the broader open source community about the threat.</li> 
   <li><b>Enhanced detection standards</b> – Insights from this campaign are informing improved detection capabilities and policy recommendations across the open source security community.</li> 
  </ul> 
  <p>This collaboration exemplifies how industry leaders and community organizations can work together to help protect software supply chains. The rapid assignment of MAL-IDs demonstrates OpenSSF’s commitment to maintaining the integrity of open source registries, while the researchers’ detection work and threat intelligence provide the advanced insights needed to stay ahead of evolving attack patterns.</p> 
  <div class="RichTextHeading"> 
   <h2>Technical details: how the researchers detected the campaign</h2> 
   <p></p>
  </div> 
  <p>Amazon Inspector security researchers used a combination of rule-based detection paired with AI-powered techniques to uncover this campaign. The researchers developed pattern matching rules to identify suspicious characteristics such as the following:</p> 
  <ul class="rte2-style-ul" id="rte-296d9070-c0e7-11f0-a18c-39aa29de08dc"> 
   <li>Presence of tea.yaml configuration files</li> 
   <li>Minimal or cloned code with no original functionality</li> 
   <li>Predictable naming patterns and automated generation signatures</li> 
   <li>Circular dependency chains between related packages</li> 
  </ul> 
  <p>By monitoring publishing patterns, the researchers revealed coordinated campaigns that used automated tooling to create packages at automated speeds.</p> 
  <div class="RichTextHeading"> 
   <h2>How to respond to these types of events</h2> 
   <p></p>
  </div> 
  <p>You should follow your standard incident response process for active incidents to resolve the issue.</p> 
  <p>To sweep your development environment, we recommend the following steps:</p> 
  <ul class="rte2-style-ul" id="rte-296d9071-c0e7-11f0-a18c-39aa29de08dc"> 
   <li><b>Use Amazon Inspector</b> – Check the <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/inspector/latest/user/findings-understanding.html" rel="noopener" target="_blank">findings</a></span> for packages that are linked to tea.xyz token farming and follow recommended remediation.</li> 
   <li><b>Audit packages</b> – Remove low-quality, non-functional packages.</li> 
   <li><b>Harden supply chains</b> – Enforce software bills of materials (SBOMs), pin package versions, and isolate continuous integration and continuous delivery (CI/CD) environments.</li> 
  </ul>
 </div> 
</div> 
<p>If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, add a note in <a href="https://repost.aws/tags/TAh3ZC0bgYTTu2DwuFqLpicw/amazon-inspector" rel="noopener noreferrer" target="_blank">AWS re:Post tagged with Amazon Inspector</a>, or <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<hr /> 
<footer> 
 <div class="blog-author-box">
  <img alt="Chi Tran" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/09/30/chi-tran-author.jpg" style="width: 93.750px; height: 125px; margin: 12px 18px 6px 12px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Chi Tran</span>
  <br /> Chi is a Senior Security Researcher at Amazon Web Services, specializing in open-source software supply chain security. He leads the R&amp;D of the engine behind Amazon Inspector that detects malicious packages in open-source software. As an Amazon Inspector SME, Chi provides technical guidance to customers on complex security implementations and advanced use cases. His expertise spans cloud security, vulnerability research, and application security. Chi holds industry certifications including OSCP, OSCE, OSWE, and GPEN, has discovered multiple CVEs, and holds pending patents in open-source security innovation.
 </div> 
 <div class="blog-author-box">
  <img alt="Charlie Bacon" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/10/02/Charlie-bacon-author.jpg" style="width: 93.750px; height: 125px; margin: 12px 18px 6px 12px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Charlie Bacon</span>
  <br /> Charlie is Head of Security Engineering and Research for Amazon Inspector at AWS. He leads the teams behind the vulnerability scanning and inventory collection services which power Amazon Inspector and other Amazon Security vulnerability management tools. Before joining AWS, he spent two decades in the financial and security industries where he held senior roles in both research and product development.
 </div> 
</footer>
