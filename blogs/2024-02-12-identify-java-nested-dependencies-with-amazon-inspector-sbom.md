---
title: "Identify Java nested dependencies with Amazon Inspector SBOM Generator"
url: "https://aws.amazon.com/blogs/security/identify-java-nested-dependencies-with-amazon-inspector-sbom-generator/"
date: "Mon, 12 Feb 2024 17:55:15 +0000"
author: "Chi Tran"
feed_url: "https://aws.amazon.com/blogs/security/category/security-identity-compliance/amazon-inspector/feed/"
---
<p><a href="https://aws.amazon.com/inspector/" rel="noopener" target="_blank">Amazon Inspector</a> is an automated vulnerability management service that continually scans <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> workloads for software vulnerabilities and unintended network exposure. Amazon Inspector currently supports vulnerability reporting for <a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a> instances, container images stored in <a href="https://aws.amazon.com/ecr/" rel="noopener" target="_blank">Amazon Elastic Container Registry (Amazon ECR)</a>, and <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a>.</p> 
<p>Java archive files (JAR, WAR, and EAR) are widely used for packaging Java applications and libraries. These files can contain various dependencies that are required for the proper functioning of the application. In some cases, a JAR file might include other JAR files within its structure, leading to nested dependencies. To help maintain the security and stability of Java applications, you must identify and manage nested dependencies.</p> 
<p>In this post, I will show you how to navigate the challenges of uncovering nested Java dependencies, guiding you through the process of analyzing JAR files and uncovering these dependencies. We will focus on the vulnerabilities that Amazon Inspector identifies using the <a href="https://docs.aws.amazon.com/inspector/latest/user/sbom-generator.html" rel="noopener" target="_blank">Amazon Inspector SBOM Generator</a>.</p> 
<h2>The challenge of uncovering nested Java dependencies</h2> 
<p>Nested dependencies in Java applications can be outdated or contain known vulnerabilities linked to <a href="https://www.cve.org/" rel="noopener" target="_blank">Common Vulnerabilities and Exposures (CVEs)</a>. A crucial issue that customers face is the tendency to overlook nested dependencies during analysis and triage. This oversight can lead to the misclassification of vulnerabilities as false positives, posing a security risk.</p> 
<p>This challenge arises from several factors:</p> 
<ul> 
 <li><strong>Volume of vulnerabilities</strong> — When customers encounter a high volume of vulnerabilities, the sheer number can be overwhelming, making it challenging to dedicate sufficient time and resources to thoroughly analyze each one.</li> 
 <li><strong>Lack of tools or insufficient tooling</strong> — There is often a gap in the available tools to effectively identify nested dependencies (for example, <a href="https://maven.apache.org/plugins/maven-dependency-plugin/tree-mojo.html" rel="noopener" target="_blank">mvn dependency:tree</a>, <a href="https://owasp.org/www-project-dependency-check/" rel="noopener" target="_blank">OWASP Dependency-Check</a>). Without the right tools, customers can miss critical dependencies hidden deep within their applications.</li> 
 <li><strong>Understanding the complexity</strong> — Understanding the intricate web of nested dependencies requires a specific skill set and knowledge base. Deficits in this area can hinder effective analysis and risk mitigation.</li> 
</ul> 
<h2>Overview of nested dependencies</h2> 
<p>Nested dependencies occur when a library or module that is required by your application relies on additional libraries or modules. This is a common scenario in modern software development because developers often use third-party libraries to build upon existing solutions and to benefit from the collective knowledge of the open-source community.</p> 
<p>In the context of JAR files, nested dependencies can arise when a JAR file includes other JAR files as part of its structure. These nested files can have their own dependencies, which might further depend on other libraries, creating a chain of dependencies. Nested dependencies help to modularize code and promote code reuse, but they can introduce complexity and increase the potential for security vulnerabilities if they aren’t managed properly.</p> 
<h3>Why it’s important to know what dependencies are consumed in a JAR file</h3> 
<p>Consider the following examples, which depict a typical file structure of a Java application to illustrate how nested dependencies are organized:</p> 
<h4 id="example_1" style="font-style: italic;">Example 1 — Log4J dependency</h4> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">MyWebApp/
|-- mywebapp-1.0-SNAPSHOT.jar
|   |-- spring-boot-3.0.2.jar
|   |   |-- spring-boot-autoconfigure-3.0.2.jar
|   |   |   |-- ...
|   |   |   |   |-- log4j-to-slf4j.jar</code></pre> 
</div> 
<p>This structure includes the following files and dependencies:</p> 
<ul> 
 <li><span style="font-family: courier;">mywebapp-1.0-SNAPSHOT.jar</span> is the main application JAR file.</li> 
 <li>Within <span style="font-family: courier;">mywebapp-1.0-SNAPSHOT.jar</span>, there’s <span style="font-family: courier;">spring-boot-3.0.2.jar</span>, which is a dependency of the main application.</li> 
 <li>Nested within <span style="font-family: courier;">spring-boot-3.0.2.jar</span>, there’s <span style="font-family: courier;">spring-boot-autoconfigure-3.0.2.jar</span>, a transitive dependency.</li> 
 <li>Within <span style="font-family: courier;">spring-boot-autoconfigure-3.0.2.jar</span>, there’s <span style="font-family: courier;">log4j-to-slf4j.jar</span>, which is our nested <a href="https://logging.apache.org/log4j/2.x/" rel="noopener" target="_blank">Log4J</a> dependency.</li> 
</ul> 
<p>This structure illustrates how a Java application might include nested dependencies, with Log4J nested within other libraries. The actual nesting and dependencies will vary based on the specific libraries and versions that you use in your project.</p> 
<h4 id="example_2" style="font-style: italic;">Example 2 — Jackson dependency</h4> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">MyFinanceApp/
|-- myfinanceapp-2.5.jar
|   |-- jackson-databind-2.9.10.jar
|   |   |-- jackson-core-2.9.10.jar
|   |   |   |-- ...
|   |   |-- jackson-annotations-2.9.10.jar
|   |   |   |-- ...</code></pre> 
</div> 
<p>This structure includes the following files and dependencies:</p> 
<ul> 
 <li><span style="font-family: courier;">myfinanceapp-2.5.jar</span> is the primary application JAR file.</li> 
 <li>Within <span style="font-family: courier;">myfinanceapp-2.5.jar</span>, there is <span style="font-family: courier;">jackson-databind-2.9.10.1.jar</span>, which is a library that the main application relies on for JSON processing.</li> 
 <li>Nested within <span style="font-family: courier;">jackson-databind-2.9.10.1.jar</span>, there are other Jackson components such as <span style="font-family: courier;">jackson-core-2.9.10.jar</span> and <span style="font-family: courier;">jackson-annotations-2.9.10.jar</span>. These are dependencies that jackson-databind itself requires to function.</li> 
</ul> 
<p>This structure is an example for Java applications that use <a href="https://github.com/FasterXML/jackson-databind" rel="noopener" target="_blank">Jackson</a> for JSON operations. Because Jackson libraries are frequently updated to address various issues, including performance optimizations and security fixes, developers need to be aware of these nested dependencies to keep their applications up-to-date and secure. If you have detailed knowledge of where these components are nested within your application, it will be easier to maintain and upgrade them.</p> 
<h4 id="example_3" style="font-style: italic;">Example 3 — Hibernate dependency</h4> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">MyERPSystem/
|-- myerpsystem-3.1.jar
|   |-- hibernate-core-5.4.18.Final.jar
|   |   |-- hibernate-validator-6.1.5.Final.jar
|   |   |   |-- ...
|   |   |-- hibernate-entitymanager-5.4.18.Final.jar
|   |   |   |-- ...</code></pre> 
</div> 
<p>This structure includes the following files and dependencies:</p> 
<ul> 
 <li><span style="font-family: courier;">myerpsystem-3.1.jar</span> as the primary JAR file of the application.</li> 
 <li>Within <span style="font-family: courier;">myerpsystem-3.1.jar</span>, <span style="font-family: courier;">hibernate-core-5.4.18.Final.jar</span> serves as a dependency for object-relational mapping (ORM) capabilities.</li> 
 <li>Nested dependencies such as <span style="font-family: courier;">hibernate-validator-6.1.5.Final.jar</span> and <span style="font-family: courier;">hibernate-entitymanager-5.4.18.Final.jar</span> are crucial for the validation and entity management functionalities that Hibernate provides.</li> 
</ul> 
<p>In instances where <a href="https://www.sap.com/products/erp/what-is-erp.html" rel="noopener" target="_blank">MyERPSystem</a> encounters operational issues due to a mismatch between the <a href="https://mvnrepository.com/artifact/org.hibernate/hibernate-core" rel="noopener" target="_blank">Hibernate</a> versions and another library (that is, a newer version of <a href="https://mvnrepository.com/artifact/org.springframework" rel="noopener" target="_blank">Spring</a> expecting a different version of Hibernate), developers can use the detailed insights that Amazon Inspector SBOM Generator provides. This tool helps quickly pinpoint the exact versions of Hibernate and its nested dependencies, facilitating a faster resolution to compatibility problems.</p> 
<p>Here are some reasons why it’s important to understand the dependencies that are consumed within a JAR file:</p> 
<ul> 
 <li><strong>Security</strong> — Nested dependencies can introduce vulnerabilities if they are outdated or have known security issues. A prime example is the <a href="https://www.cisa.gov/news-events/news/apache-log4j-vulnerability-guidance" rel="noopener" target="_blank">Log4J vulnerability discovered in late 2021 (CVE-2021-44228)</a>. This vulnerability was critical because Log4J is a widely used logging framework, and threat actors could have exploited the flaw remotely, leading to serious consequences. What exacerbated the issue was the fact that Log4J often existed as a nested dependency in various Java applications (see <a href="#example_1">Example 1</a>), making it difficult for organizations to identify and patch each instance.</li> 
 <li><strong>Compliance</strong> — Many organizations must adhere to strict policies about third-party libraries for licensing, regulatory, or security reasons. Not knowing the dependencies, especially nested ones such as in the Log4J case, can lead to non-compliance with these policies.</li> 
 <li><strong>Maintainability</strong> — It’s crucial that you stay informed about the dependencies within your project for timely updates or replacements. Consider the Jackson library (<a href="#example_2">Example 2</a>), which is often updated to introduce new features or to patch security vulnerabilities. Managing these updates can be complex, especially when the library is a nested dependency.</li> 
 <li><strong>Troubleshooting</strong> — Identifying dependencies plays a critical role in resolving operational issues swiftly. An example of this is addressing compatibility issues between Hibernate and other Java libraries or frameworks within your application due to version mismatches (<a href="#example_3">Example 3</a>). Such problems often manifest as unexpected exceptions or degraded performance, so you need to have a precise understanding of the libraries involved.</li> 
</ul> 
<p>These examples underscore that you need to have deep visibility into JAR file contents to help protect against immediate threats and help ensure long-term application health and compliance.</p> 
<h2>Existing tooling limitations</h2> 
<p>When analyzing Java applications for nested dependencies, one of the main challenges is that existing tools can’t efficiently narrow down the exact location of these dependencies. This issue is particularly evident with tools such as <span style="font-family: courier;">mvn dependency:tree</span>, OWASP Dependency-Check, and similar dependency analysis solutions.</p> 
<p>Although tools are available to analyze Java applications for nested dependencies, they often fall short in several key areas. The following points highlight common limitations of these tools:</p> 
<ul> 
 <li><strong>Inadequate depth in dependency trees</strong> — Although other tools provide a hierarchical view of project dependencies, they often fail to delve deep enough to reveal nested dependencies, particularly those that are embedded within other JAR files as nested dependencies. Nested dependencies are repackaged within a library and aren’t immediately visible in the standard dependency tree.</li> 
 <li><strong>Lack of specific location details</strong> — These tools typically don’t offer the granularity needed to pinpoint the exact location of a nested dependency within a JAR file. For large and complex Java applications, it may be challenging to identify and address specific dependencies, especially when they are deeply embedded.</li> 
 <li><strong>Complexity in large projects</strong> — In projects with a vast and intricate network of dependencies, these tools can struggle to provide clear and actionable insights. The output can be complicated and difficult to navigate, leaving customers without a clear path to identifying critical dependencies.</li> 
</ul> 
<h2>Address tooling limitations with Amazon Inspector SBOM Generator</h2> 
<p>The <a href="https://docs.aws.amazon.com/inspector/latest/user/sbom-generator.html" rel="noopener" target="_blank">Amazon Inspector SBOM Generator (Sbomgen)</a> introduces a significant advancement in the identification of nested dependencies in Java applications. Although the concept of monitoring dependencies is well-established in software development, AWS has tailored this tool to enhance visibility into the complexities of software compositions. By generating a software bill of materials (SBOM) for a container image, Sbomgen provides a detailed inventory of the software installed on a system, including hidden nested dependencies that traditional tools can overlook. This capability enriches the existing toolkit, offering a more granular and actionable understanding of the dependency structure of your applications.</p> 
<p>Sbomgen works by scanning for files that contain information about installed packages. Upon finding such files, it extracts essential data such as package names, versions, and other metadata. Then it transforms this metadata into a <a href="https://cyclonedx.org/" rel="noopener" target="_blank">CycloneDX</a> SBOM, providing a structured and detailed view of the dependencies.</p> 
<p>For information about how to install Sbomgen, see <a href="https://docs.aws.amazon.com/inspector/latest/user/sbom-generator.html#install-sbomgen" rel="noopener" target="_blank">Installing Amazon Inspector SBOM Generator (Sbomgen)</a></p> 
<p>A key feature of Sbomgen is its ability to provide explicit paths to each dependency.</p> 
<p>For example, given a compiled jar application <span style="font-family: courier;">MyWebApp-0.0.1-SNAPSHOT.jar</span>, users can run the following CLI command with Sbomgen:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">./inspector-sbomgen localhost --path /path/to/MyWebApp-0.0.1-SNAPSHOT.jar --scanners java-jar</code></pre> 
</div> 
<p>The output should look similar to the following:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
  "bom-ref": "comp-11",
  "type": "library",
  "name": "org.apache.logging.log4j/log4j-to-slf4j",
  "version": "2.19.0",
  "hashes": [
    {
      "alg": "SHA-1",
      "content": "30f4812e43172ecca5041da2cb6b965cc4777c19"
    }
  ],
  "purl": "pkg:maven/org.apache.logging.log4j/log4j-to-slf4j@2.19.0",
  "properties": [
...
    {
      "name": "amazon:inspector:sbom_generator:source_path",
      "value": "/tmp/MyWebApp-0.0.1-SNAPSHOT.jar/BOOT-INF/lib/spring-boot-3.0.2.jar/BOOT-INF/lib/spring-boot-autoconfigure-3.0.2.jar/BOOT-INF/lib/logback-classic-1.4.5.jar/BOOT-INF/lib/logback-core-1.4.5.jar/BOOT-INF/lib/log4j-to-slf4j-2.19.0.jar/META-INF/maven/org.apache.logging.log4j/log4j-to-slf4j/pom.properties"
    }
  ]
}</code></pre> 
</div> 
<p>In this output, the <span style="font-family: courier;">amazon:inspector:sbom_collector:path</span> property is particularly significant. It provides a clear and complete path to the location of the specific dependency (in this case, <span style="font-family: courier;">log4j-to-slf4j</span>) within the application’s structure. This level of detail is crucial for several reasons:</p> 
<ul> 
 <li><strong>Precise location identification</strong> — It helps you quickly and accurately identify the exact location of each dependency, which is especially useful for nested dependencies that are otherwise hard to locate.</li> 
 <li><strong>Effective risk management</strong> — When you know the exact path of dependencies, you can more efficiently assess and address security risks associated with these dependencies.</li> 
 <li><strong>Time and resource efficiency</strong> — It reduces the time and resources needed to manually trace and analyze dependencies, streamlining the vulnerability management process.</li> 
 <li><strong>Enhanced visibility and transparency</strong> — It provides a clearer understanding of the application’s dependency structure, contributing to better overall management and maintenance.</li> 
 <li><strong>Comprehensive package information</strong> — The detailed package information, including name, version, hashes, and <a href="https://github.com/package-url/purl-spec?tab=readme-ov-file#purl" rel="noopener" target="_blank">package URL</a>, of Sbomgen equips you with a thorough understanding of each dependency’s specifics, aiding in precise vulnerability tracking and software integrity verification.</li> 
</ul> 
<h2>Mitigate vulnerable dependencies</h2> 
<p>After you identify the nested dependencies in your Java JAR files, you should verify whether these dependencies are outdated or vulnerable. Amazon Inspector can help you achieve this by doing the following:</p> 
<ul> 
 <li>Comparing the discovered dependencies against a database of known vulnerabilities.</li> 
 <li>Providing a list of potentially vulnerable dependencies, along with detailed information about the associated CVEs.</li> 
 <li>Offering recommendations on how to mitigate the risks, such as updating the dependencies to a newer, more secure version.</li> 
</ul> 
<p>By integrating Amazon Inspector into your software development lifecycle, you can continuously monitor your Java applications for vulnerable nested dependencies and take the necessary steps to help ensure that your application remains secure and compliant.</p> 
<h2>Conclusion</h2> 
<p>To help secure your Java applications, you must manage nested dependencies. Amazon Inspector provides an automated and efficient way to discover and mitigate potentially vulnerable dependencies in JAR files. By using the capabilities of Amazon Inspector, you can help improve the security posture of your Java applications and help ensure that they adhere to best practices.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Chi Tran " class="aligncenter size-full wp-image-33385" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/02/08/actran.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Chi Tran</h3> 
  <p>Chi is a Security Researcher who helps ensure that AWS services, applications, and websites are designed and implemented to the highest security standards. He’s a SME for Amazon Inspector and enthusiastically assists customers with advanced issues and use cases. Chi is passionate about information security — API security, penetration testing (he’s OSCP, OSCE, OSWE, GPEN certified), application security, and cloud security.</p> 
  <p></p>
 </div> 
</footer>
