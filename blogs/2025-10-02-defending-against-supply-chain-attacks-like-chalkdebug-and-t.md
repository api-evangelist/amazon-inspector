---
title: "Defending against supply chain attacks like Chalk/Debug and the Shai-Hulud worm"
url: "https://aws.amazon.com/blogs/security/defending-against-supply-chain-attacks-like-chalk-debug-and-the-shai-hulud-worm/"
date: "Thu, 02 Oct 2025 16:43:02 +0000"
author: "Chi Tran"
feed_url: "https://aws.amazon.com/blogs/security/category/security-identity-compliance/amazon-inspector/feed/"
---
<div class="RichTextArticleBody RichTextBody">
  Building on top of open source packages can help accelerate development. By using common libraries and modules from npm, PyPI, Maven Central, NuGet, and others, teams can focus on writing code that is unique to their situation. These open source package registries host millions of packages that are integrated into thousands of programs daily.
 <p></p> 
 <p>Unfortunately, these key services are prime targets for threat actors looking to distribute their code at scale. If they can compromise a package in one of these services, that one action can automatically affect thousands of other systems.</p> 
 <div class="RichTextHeading"> 
  <h2>September 8: Chalk and Debug compromise</h2> 
 </div> 
 <p>It started with compromised credentials for a trusted maintainer for npm. After social engineering the credentials, 18 popular packages (including Chalk, Debug, ansi-styles, supports-color, and more) were updated with an injected payload.</p> 
 <p>This payload was designed to silently intercept cryptocurrency activity and manipulate transactions to the bad actor’s benefit.</p> 
 <p>Together these packages are downloaded an estimated two billion times each week. That means even with the rapid response from the maintainer and npm, the couple of hours that the compromised versions were available could have led to significant exposures. Any build systems that downloaded the packages during this window or sites that loaded them remotely were potentially vulnerable.</p> 
 <p>This sophisticated malware used intelligent reconnaissance techniques and adapted its behavior to find the most effective attack vector for its current context.</p> 
 <div class="RichTextHeading"> 
  <h2>September 15: Shai-Hulud worm</h2> 
 </div> 
 <p>The very next week, the Shai-Hulud worm started to spread autonomously through the npm trust chain. This malware uses its initial foothold in a developer’s environment to harvest a variety of credentials, such as npm tokens, GitHub personal access tokens, and cloud credentials.</p> 
 <p>When possible, the malware would expose the harvested credentials publicly. When npm tokens are available, it publishes updated packages that now contain the worm as an additional payload. The now compromised packages will execute the worm as a postinstall script to continue propagating the infection.</p> 
 <p>In addition to this self-propagation method, the worm also attempts to manipulate GitHub repositories it gains access to. Shai-Hulud sets up malicious workflows that run on every repository activity, creating a resilient and continuous exfiltration of code.</p> 
 <p>This exploit showed technical sophistication and a deep understanding of the developer workflows and the trust relationships that power the community. By using the standard npm installation processes, the worm makes detection more challenging because it operates within the behavioral patterns expected of developers.</p> 
 <p>Within the first 24 hours of this exploit, over 180 npm packages had been compromised, again potentially affecting millions of systems. Both incidents show the potential scale of supply chain compromises.</p> 
 <div class="RichTextHeading"> 
  <h2>How to respond to these types of events</h2> 
 </div> 
 <p>If a compromised package has made it into production, you should follow your standard incident response process for active incidents to resolve the issue.<br /> To sweep your development environment, we recommend the following steps:</p> 
 <ol class="rte2-style-ol" id="rte-ca9eea70-9e50-11f0-877c-e313b98d4334" start="1"> 
  <li><b>Audit dependencies</b>: Remove or upgrade to clean versions of Chalk and Debug packages and check for Shai-Hulud-infected packages.</li> 
  <li><b>Rotate secrets</b>: Assume npm tokens, GitHub PATs, and API keys might be compromised. Rotate and reissue credentials immediately.</li> 
  <li><b>Audit build pipelines</b>: Check for unauthorized GitHub Actions workflows or unexpected script insertions.</li> 
  <li><b>Use Amazon Inspector</b>: Review <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/inspector" rel="noopener" target="_blank">Amazon Inspector</a></span> findings for exposure to the Chalk/Debug exploit or Shai-Hulud worm and follow recommended remediation.</li> 
  <li><b>Harden supply chains</b>: Enforce SBOMs, pin package versions, adopt scoped tokens, and isolate continuous integration and delivery (CI/CD) environments.</li> 
 </ol> 
 <div class="RichTextHeading"> 
  <h2>How Amazon Inspector strengthens open source security with OpenSSF</h2> 
 </div> 
 <p>We regularly share the findings from the malicious package detection system in Amazon Inspector with the community through our partnership with the Open Source Security Foundation (OpenSSF). Amazon Inspector uses an automated process to share this type of threat intelligence using the Open Source Vulnerability (OSV) format.</p> 
 <p>Amazon Inspector employs a multi-layered detection approach that combines complementary analysis techniques to identify malicious packages. This approach provides robust protection against both known attack patterns and novel threats.</p> 
 <p>Starting with static analysis using an extensive library of YARA rules, Amazon Inspector can identify suspicious code patterns, obfuscation techniques, and known malicious signatures within package contents. Building on that, the system uses dynamic analysis and behavioral monitoring to identify threats, despite their use of evasion techniques. The final set of analysis is conducted using AI and machine learning models to analyze code semantics and determine the intended purpose versus suspicious functionality within packages.</p> 
 <p>This multi-stage approach enables Amazon Inspector to maintain high detection accuracy while minimizing false positives, helping to make sure that legitimate packages are not incorrectly flagged and sophisticated threats are reliably identified and mitigated.</p> 
 <p>When these threats are detected in open source packages, the system starts the automated workflows to share this threat intelligence with the OpenSSF. This workflow sends the validated threat intelligence to the OpenSSF where the contributions are rigorously reviewed by the OpenSSF maintainers before being merged in the community database. That is where they receive an official MAL-ID or malicious package identifier.</p> 
 <p>This process helps verify and share these types of discoveries as quickly as possible with the community, so that other security tools and researchers benefit from the detection capabilities of Amazon Inspector.</p> 
 <div class="RichTextHeading"> 
  <h2>What’s next?</h2> 
 </div> 
 <p>Chalk/Debug and the Shai-Hulud worm are not novel exploits. These are—unfortunately—the most recent incidents using this vector. Open source repositories are a fantastic resource for developers and help many teams to innovate more quickly. The open source community is working hard to reduce the impact of these types of incidents.</p> 
 <p>That is why we have partnered with the OpenSSF and have contributed reports that highlight over 40,000 npm packages that were compromised or created with malicious intent. We believe that Amazon Inspector is an excellent tool to help you build safely and securely, and while we would love everyone to use it, we are proud that our work and contributions to efforts like OpenSSF are helping improve the security of everyone in the community.</p> 
 <hr /> 
 <p>If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, add a note in <a href="https://repost.aws/tags/TAh3ZC0bgYTTu2DwuFqLpicw/amazon-inspector" rel="noopener noreferrer" target="_blank">AWS re:Post tagged with Amazon Inspector</a>, or <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
</div> 
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
 <div class="blog-author-box">
  <img alt="Nirali Desai" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/06/13/Nirali-Desai-author.jpg" style="width: 93.750px; height: 125px; margin: 12px 18px 6px 12px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Nirali Desai </span>
  <br /> Nirali is a product leader in cloud security, currently driving Application Security initiatives at Amazon Web Services (AWS). Before joining AWS, she held key roles at Palo Alto Networks, Zscaler, and Cisco Tetration, where she focused on building Secure Access Secure Edge (SASE), end-user security, workload protection, and zero-trust security solutions. She is passionate about building scalable security products that defend against evolving cyber threats.
 </div> 
</footer>
