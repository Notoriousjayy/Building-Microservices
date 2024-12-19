# Testing

The field of automated testing has come a long way since I began coding, and it seems like each month brings a new tool or strategy to improve it. However, we still face difficulties when it comes to verifying that our code functions correctly across distributed systems. In this chapter, we’ll examine the hurdles associated with testing more granular, service-oriented architectures and propose approaches that will help you confidently deploy new features.

The concept of testing encompasses a wide range of areas. Even if we focus solely on automated tests, there are many different kinds to consider. With the shift to microservices, we introduce another layer of complexity. It’s vital to understand the various types of tests available so we can find the right balance between quickly delivering software to production and ensuring it meets quality standards. Rather than attempting an all-encompassing review of testing, this chapter concentrates on what distinguishes testing strategies in a microservice-based ecosystem from those in less distributed environments, like single-process monoliths.

The location and timing of testing have also evolved since the first edition of this book. Previously, testing took place primarily before deployment. Increasingly, however, we find ourselves conducting tests directly in production, further blurring the line between development and operational responsibilities.

## **Categories of Testing**  
As a consultant, I’ve often used quadrants to classify concepts, and fortunately, testing fits neatly into this approach. Brian Marick devised an excellent framework for categorizing tests, which Lisa Crispin and Janet Gregory later adapted in their book *Agile Testing*. Depicts their variation of Marick’s quadrant, which serves as a helpful way to organize different testing types.

![Brian Marick's Testing Quadrant](TestingQuadrant.svg)

At the lower half of the quadrant, we find technology-facing tests—those designed to assist developers in building the system. This includes unit tests and performance tests, which focus on specific aspects of the system and are typically automated. The upper half of the quadrant features business-facing tests, aimed at helping non-technical stakeholders understand the system’s behavior. These include comprehensive end-to-end acceptance tests and exploratory tests conducted manually, such as user testing in a UAT (User Acceptance Testing) environment.

It’s important to note that most of these tests are centered on preproduction validation, ensuring the software meets quality standards before deployment. These tests often serve as gatekeepers, determining whether the software is ready for production. However, there’s growing recognition of the value in testing software after deployment in the production environment. We’ll delve deeper into balancing preproduction and production testing later, but for now, it’s worth mentioning that Marick’s quadrant primarily focuses on preproduction.

Each type of test in this framework has a distinct role, and the emphasis on different tests depends on the system’s needs. Recent trends favor automating repetitive tests, reducing reliance on large-scale manual testing—a shift that aligns well with the fast-paced nature of microservices. If manual testing is still a major part of your workflow, consider streamlining and automating these repetitive tasks before fully embracing microservices, as rapid and efficient validation is critical for leveraging their advantages.



**Manual Exploratory Testing**  
Transitioning from a monolithic to a microservices architecture generally has limited impact on exploratory testing, aside from potential organizational changes. As systems and teams become more modular, ownership of manual testing might also shift to align with these new structures. 

Traditionally, tasks like verifying visual aspects of an application were limited to manual exploratory testing. However, advancements in tools for visual assertions have enabled automation of some previously manual tasks. While this automation reduces repetitive work, it doesn’t eliminate the need for manual testing. Instead, it allows testers to focus on creative and exploratory testing that uncovers hidden issues or addresses scenarios where automation isn’t cost-effective.

Manual exploratory testing excels at discovery, providing valuable insights by simulating end-user experiences. It is especially useful in cases where automating a test would be impractical or excessively expensive. Automation should be seen as a way to handle repetitive tasks, freeing testers to engage in higher-value, ad hoc activities that demand creativity and intuition.

In this chapter, the focus is primarily on automated testing, particularly in the context of microservices, rather than manual exploratory testing. This isn’t to downplay the importance of manual testing but to highlight how microservice testing differs from traditional monolithic approaches. To determine the right balance of automated tests and understand their trade-offs, we’ll introduce another model to guide these decisions.

## Test Scope

In his book *Succeeding with Agile*, Mike Cohn introduces a concept called the test pyramid to illustrate the types of automated tests required. This model not only emphasizes the range of test scopes but also suggests the ideal balance of test types. Cohn’s pyramid categorizes automated tests into three levels: unit tests, service tests, and UI tests.

![Mike Cohn's Test Pyramid](TestPyramid.svg)

The pyramid demonstrates that as you move upward, the scope of the tests broadens, giving you greater assurance that the functionality works as intended. However, this comes at the cost of longer feedback cycles, as these tests take more time to execute. Moreover, when a higher-level test fails, identifying the specific functionality that caused the failure becomes more challenging. Conversely, at the base of the pyramid, the tests are narrower in scope, run much faster, and provide quicker feedback. Failures at this level are usually easier to diagnose, often pinpointing the exact line of code that needs fixing. These tests improve the speed of continuous integration processes and help prevent moving on to new tasks before addressing issues. That said, while these lower-level tests are effective at catching specific errors, they don’t provide much confidence that the system as a whole is functioning properly.

One limitation of this model is the ambiguity in terminology. For example, the term "service" can be interpreted in numerous ways, and definitions of what constitutes a "unit test" can vary widely. Is testing a single line of code considered a unit test? Most would agree it is. But is testing multiple functions or classes still a unit test? Opinions diverge on this point. While I often use terms like "unit" and "service" despite their potential for misinterpretation, I prefer to refer to UI tests as "end-to-end tests," a naming convention I will adopt from here onward.

In practice, every team I’ve worked with has its own terminology for tests, differing from those used in the test pyramid. Regardless of what you call them, the essential takeaway is that you need functional automated tests with varying scopes to address different objectives effectively.

To clarify this, let’s examine the layers in more detail. A segment of our MusicCorp system involving a helpdesk application and a main website, both of which interact with a Customer microservice. This microservice retrieves, reviews, and updates customer details while also communicating with a Loyalty microservice that manages customer rewards. While this is just a small slice of the overall system, it serves as a practical example for exploring various testing scenarios.

![Part of Our Music Shop Under Test](MusicShopSystem.svg)

### **Unit Tests**  
Unit tests focus on verifying the functionality of individual functions or methods in isolation. Tests produced as a byproduct of practices like test-driven development (TDD) or techniques such as property-based testing fall under this category. These tests avoid interacting with external systems, such as files or network connections, and instead concentrate solely on the specific code being tested. Ideally, you want a substantial number of unit tests. When implemented efficiently, they execute extremely quickly—on modern hardware, thousands of these tests can be completed in under a minute. Many developers even configure their unit tests to run automatically whenever local files are modified, particularly in interpreted languages, as this enables incredibly fast feedback loops.

In Brian Marick's framework, unit tests are technology-facing since they primarily assist developers rather than addressing business concerns. Their primary purpose is to catch the majority of bugs early in the development process. For instance, in the case of the Customer microservice, unit tests would focus on testing small, isolated sections of the codebase.

![Scope of Unit Tests on Our Example System](UnitTestScope.svg)

The key advantage of unit tests is their ability to provide rapid feedback on the correctness of your functionality. They are also essential when refactoring code, allowing developers to confidently reorganize their codebase knowing that these tightly scoped tests will detect any errors introduced during the process.

### **Service Tests**  
Service tests focus on interacting directly with microservices, bypassing the user interface. In a monolithic system, this might involve testing a set of classes that provide functionality to the UI. However, in a microservices architecture, service tests target the capabilities of an individual microservice.

Testing a single microservice in isolation gives us greater confidence that the service behaves as intended while keeping the scope of the test limited. If a test fails, the issue is likely confined to the microservice being tested. To maintain this isolation, external dependencies and collaborators must be replaced with stubs, ensuring that only the microservice itself is within the test's scope.

![Scope of Service Tests on Our Example System](ServiceTestScope.svg)

While some service tests can be as fast as unit tests, involving elements like databases or network communication with stubbed downstream services can increase execution time. These tests cover a broader scope than unit tests, so diagnosing failures can be more complex. However, service tests are less fragile compared to larger, end-to-end tests, as they involve fewer components and dependencies.

By carefully managing the scope, service tests strike a balance between confidence and complexity, making them a vital part of testing in distributed systems.

### **End-to-End Tests**  
End-to-end tests evaluate the functionality of your entire system as a whole. These tests often simulate user interactions through a graphical user interface (GUI) in a browser but can also replicate other types of user activities, such as file uploads.

Because these tests encompass a significant portion of production code, as illustrated in Figure 9-6, their successful execution provides a strong sense of confidence that the system will perform as expected in a live environment. However, this extensive scope introduces challenges, especially when implementing end-to-end tests in a microservices architecture, where they can be particularly difficult to execute effectively.

**What About Integration Tests?**  
You may have noticed the absence of a specific discussion on integration tests. This omission is intentional. The term "integration test" is often ambiguous and used differently depending on the context. For some, it refers to testing the interaction between two components, such as a service and its database. For others, it is synonymous with full-scale end-to-end testing. In this chapter, I’ve opted for more precise terminology to avoid confusion, leaving it up to you to map your definition of "integration tests" to the concepts described here.

### **Trade-Offs**  
The goal with the test types represented in the pyramid is to strike a thoughtful balance. We want quick feedback loops while also ensuring confidence that our system is functioning as intended.

Unit tests have a narrow focus, allowing us to identify and resolve issues quickly when they fail. They are simple to write and execute rapidly. However, as the scope of tests increases—moving toward service and end-to-end tests—our confidence in the system grows, but feedback becomes slower. These broader tests take more time to write, maintain, and run, which can impact development cycles.

Finding the right mix of test types is an ongoing challenge. If your test suite becomes too slow, consider adding smaller, faster unit tests to catch issues earlier and reduce reliance on slower, broader tests like service or end-to-end tests. Conversely, if issues are slipping through to production, it might indicate gaps in your test coverage that require additional higher-level tests.

A general rule is that the number of tests should increase by an order of magnitude as you move down the pyramid. For example, one system I worked on had 4,000 unit tests, 1,000 service tests, and 60 end-to-end tests. Over time, we realized that the higher number of service and end-to-end tests was negatively impacting feedback speed, so we focused on replacing some of their coverage with smaller, faster tests.

One common pitfall to avoid is the "test snow cone" or inverted pyramid, where most test coverage relies on large-scope tests, with little or no focus on small-scoped ones. This pattern leads to excessively slow test execution, long feedback cycles, and inefficient continuous integration pipelines. In such cases, builds are often delayed, and when something breaks, resolving it can take an extended period due to the length of the test runs.

## Implementing Service Tests

Creating unit tests is relatively straightforward, with a wealth of resources available to guide the process. However, service and end-to-end tests become more intriguing, particularly in the realm of microservices. In this section, we’ll delve into service tests and how to implement them effectively.

Service tests aim to validate a specific slice of functionality within an individual microservice, isolating it from its dependencies. For instance, to write a service test for the Customer microservice depicted in Figure 9-3, we would deploy an instance of the Customer microservice. To ensure the test's focus remains on this microservice, we would stub out its downstream collaborator, the Loyalty microservice. This approach helps guarantee that any test failures are linked to the Customer microservice itself.

As discussed in Chapter 7, once code is committed, the automated build process typically creates a binary artifact for the microservice, such as a container image for the current version of the software. Deploying this artifact is a straightforward step. The challenge lies in simulating the behavior of downstream dependencies.

To implement service tests, the test suite must stub out all external collaborators and configure the microservice under test to communicate with these stubs. The stubs are then programmed to return predefined responses that replicate the behavior of the actual services, allowing the test to mimic real-world interactions without involving the live systems. This setup ensures that tests remain focused, reliable, and effective in isolating issues within the microservice under test.

### **Mocking or Stubbing**  
When I refer to stubbing downstream collaborators, I’m talking about creating a simplified version of a microservice that responds with predefined outputs to specific requests from the microservice under test. For instance, if testing the Loyalty microservice, the stub might be set up to always return a balance of 15,000 for customer ID 123. Importantly, a stub doesn’t track how many times it’s called—it could be zero, once, or a hundred times, and the test would remain unaffected. An alternative to stubbing is using mocks.

Mocks add an extra layer of validation by verifying that specific calls were made during the test. For example, if the microservice under test is supposed to query a collaborator, the mock would ensure this interaction happens. If the call is not made, the test fails. This approach requires more intelligent fake services, but overusing mocks can lead to fragile tests that are easily broken by minor changes. In contrast, stubs are more forgiving since they do not depend on whether or how often they are called.

There are situations, however, where mocks can be helpful. For instance, you might want to confirm that creating a new customer triggers the creation of an associated points balance in the Loyalty service. Balancing the use of mocks and stubs is key and applies to both service tests and unit tests. In general, I rely much more on stubs than mocks for service tests, as stubs tend to be simpler and more robust. For a deeper exploration of this topic, I recommend *Growing Object-Oriented Software, Guided by Tests* by Steve Freeman and Nat Pryce.

While I use mocks sparingly, having a tool that supports both stubs and mocks can be extremely valuable. Although the distinction between the two is usually clear, it can become confusing when additional terms like fakes, spies, and dummies are introduced. Gerard Meszaros simplifies this by grouping all of these under the umbrella term “Test Doubles,” which includes stubs, mocks, and other similar constructs.

### **A Smarter Stub Service**  
In the past, I’ve often built stub services manually for testing purposes. These ranged from using Apache or nginx to embedded Jetty containers or even lightweight Python web servers launched from the command line. Each time, I essentially reinvented the wheel, repeating similar work to set up these stubs. Fortunately, a former Thoughtworks colleague, Brandon Byars, has simplified this process significantly with his tool, **mountebank**, a powerful stub/mock server.

Mountebank acts like a small software appliance that can be programmed via HTTP. The fact that it’s built with Node.js is irrelevant to the services interacting with it. Once mountebank is running, you can send it commands to create “imposters.” These imposters listen on a specified port, support a variety of protocols (such as TCP, HTTP, HTTPS, and SMTP), and return predefined responses to incoming requests. Additionally, mountebank can be configured to set expectations for calls, making it useful as a mock server when needed. Because a single instance can manage multiple imposters, mountebank is ideal for stubbing multiple downstream microservices simultaneously.

Mountebank isn’t limited to functional testing. For example, Capital One used it to replace their existing mocking infrastructure for large-scale performance testing. However, mountebank does have limitations—for instance, it doesn’t currently support stubbing messaging protocols. If you need to verify that an event was sent or received correctly via a message broker, you’ll need a different solution. Tools like Pact might be better suited for these scenarios, which we’ll discuss later.

To test the Customer microservice using mountebank, you could deploy the Customer microservice alongside a mountebank instance configured to act as the stubbed Loyalty microservice. If these service tests pass, you might feel confident enough to deploy the Customer microservice. But what about the services that interact with the Customer microservice, such as the helpdesk and the web shop? A change in the Customer microservice could still break those systems. This highlights the importance of including the broader tests at the top of the pyramid—end-to-end tests—to catch such issues.

## **Implementing (The Challenge of) End-to-End Tests**  
In a microservices architecture, the functionality exposed through user interfaces is supported by a network of underlying microservices. The purpose of end-to-end tests, as depicted in Mike Cohn’s pyramid, is to validate this functionality by simulating user interactions and assessing the entire system’s quality.

To perform an end-to-end test, we must deploy multiple microservices together and execute tests against them as a cohesive unit. These tests cover a broad scope, providing high confidence that the system functions as intended. However, the trade-off is slower execution times and increased difficulty in pinpointing the cause of failures. Let’s explore these tests further using our earlier example.

Suppose you’re deploying a new version of the Customer microservice. While eager to roll out the changes quickly, you’re concerned that they might disrupt the helpdesk or web shop. To address this, you could deploy all the relevant microservices and run tests against the helpdesk and web shop to detect potential issues. A straightforward but naive approach would be to append these tests to the end of the Customer service’s pipeline.

![Adding Our End-to-End Tests Stage: The Right Approach?](CustomerPipeline.svg)

While this seems reasonable, it raises an important question: which versions of the other microservices should the tests use? Should they target the production versions of the helpdesk and web shop? That might make sense initially, but what happens if new versions of those services are queued for deployment? This complicates matters further.

Another challenge arises if multiple services each have their own end-to-end tests that overlap in functionality. For example, if both the Customer and Loyalty microservices require similar test setups, you might find yourself redundantly deploying the same microservices and running overlapping tests, wasting resources.

A more efficient solution is to use a "fan-in" approach, where multiple pipelines converge into a shared end-to-end testing stage. In this model, a successful build for any of the relevant microservices triggers the shared end-to-end tests.This approach eliminates duplication and ensures consistency. Many modern continuous integration (CI) tools support this type of pipeline coordination out of the box.

![A Standard Way to Handle End-to-End Tests Across Services](EndToEndTests.svg)

With this setup, changes to any microservice initiate local tests specific to that service. Once these tests pass, they trigger the shared integration tests. While this approach streamlines end-to-end testing, it’s not without its downsides, as end-to-end tests can still present significant challenges in terms of complexity, speed, and maintenance.

### **Who Writes These End-to-End Tests?**  
For tests tied directly to a specific microservice's pipeline, it makes sense for the team that owns the microservice to write and maintain those tests (we’ll delve deeper into service ownership in Chapter 15). However, when it comes to end-to-end tests, which span multiple microservices and potentially involve multiple teams, the question of ownership becomes more complex: who is responsible for writing and maintaining these tests?

This lack of clarity can lead to a variety of problems. In some cases, end-to-end tests become a free-for-all, with teams adding tests without considering the overall health of the test suite. This often results in an unmanageable explosion of test cases, creating a “test snow cone” effect, as discussed earlier. In other cases, the absence of clear ownership causes test failures to be ignored. When tests break, teams assume it’s someone else’s responsibility, leading to a situation where no one takes accountability for fixing them.

One approach to address this issue, as introduced by Emily Bache, is to assign ownership of specific end-to-end tests to individual teams, even if those tests span multiple microservices. For example, using a "fan-in" pipeline, end-to-end tests can be grouped by functionality and allocated to different teams. In this model:
- Changes to the Web Shop trigger its associated end-to-end tests, owned by the Web Shop team.
- Changes to the Helpdesk trigger its own associated tests.
- Changes to shared services, like Customer or Loyalty, may trigger tests owned by multiple teams.
- 
![A Standard Way to Handle End-to-End Tests Across Services](EndToEndTests2.svg)

While this model helps clarify ownership, it isn’t without its challenges. For instance, a change in the Loyalty microservice could break end-to-end tests owned by both the Web Shop and Helpdesk teams. This creates a scenario where those teams must collaborate with the Loyalty team to resolve the issue, potentially causing delays and friction. 

Some organizations attempt to resolve this by assigning a dedicated team to write all end-to-end tests. However, this can lead to even greater issues. A separate testing team can increase cycle times, as service developers must wait for the testing team to write tests for newly implemented functionality. Furthermore, distancing the developers from their tests often results in less familiarity with how the tests work, making it harder for service teams to debug and fix failures. This siloed approach can erode ownership and accountability, ultimately harming the overall development process.

Balancing end-to-end test ownership is inherently difficult. Duplicating effort across teams should be avoided, but centralizing ownership too much can lead to disconnects between teams and the functionality they build. Ideally, end-to-end tests should be assigned to specific teams wherever possible. If this isn’t feasible, the test suite should be treated as a shared codebase with joint ownership. In this case, all teams contributing to the test suite share responsibility for its health and maintenance.

While this shared ownership model can work, it’s challenging to implement effectively, particularly in larger organizations. At scale, cross-team end-to-end tests often become too complex and problematic to manage. For this reason, many organizations eventually move away from heavy reliance on end-to-end tests in favor of more targeted approaches.

### **How Long Should End-to-End Tests Run?**  
End-to-end tests can be notoriously time-consuming. In some cases, I’ve seen them take an entire day or more to complete. On one project, the full regression suite took an astonishing six weeks to run! Unfortunately, many teams don’t invest the effort needed to streamline their end-to-end test suites, reduce redundant test coverage, or improve execution speed.

This sluggishness, compounded by the potential flakiness of such tests, can become a significant bottleneck. A test suite that takes hours—or even a full day—to run, with frequent failures unrelated to actual functionality issues, can be a nightmare. Even when functionality is genuinely broken, the delay in identifying the issue can leave developers working on something else by the time the failure is reported, making it difficult to regain focus and resolve the problem efficiently.

One way to mitigate this is by running tests in parallel using tools like Selenium Grid. However, while parallelization can speed things up, it’s no substitute for critically evaluating which tests are necessary and removing those that no longer provide value.

The idea of removing tests can be controversial. It often sparks reactions similar to debates about scaling back airport security measures: no matter how inefficient the measures are, proposals to remove them are met with resistance, driven by fears of compromising safety. Similarly, in testing, removing a test—no matter how redundant—can provoke concerns about missed bugs. Teams may hesitate because while you might occasionally be praised for removing an unnecessary test, you’ll almost certainly be blamed if a missing test allows a bug to slip through. 

That said, reducing the number of redundant or overlapping tests in a large test suite is essential for maintaining efficiency. If a feature is covered by 20 different tests, removing half of them could save significant time, especially if those tests collectively take 10 minutes to run. Achieving this requires a nuanced understanding of risk—an area where humans often struggle. This kind of intelligent test suite curation is rare, not because it isn’t valuable, but because it’s challenging to implement effectively. Hoping for better test management won’t make it happen—it requires deliberate action and prioritization.

### **The Great Pile-Up**  
The extended feedback cycles associated with end-to-end tests pose challenges beyond just developer productivity. When a test suite takes a long time to complete, fixing any failures becomes a drawn-out process, reducing the amount of time the tests are passing and blocking software from being ready for deployment. If we adhere to the principle of deploying only software that has successfully passed all tests (as we should), this delay means fewer services reach the production-ready stage.

This often results in a backlog. While developers work to fix a failing integration test stage, additional changes from other teams can accumulate. Not only does this make resolving the issue more complex, but it also increases the scope of changes waiting to be deployed. Ideally, no one should be able to check in new changes when the end-to-end tests are failing. However, with lengthy test suites, enforcing this becomes impractical. Imagine telling 30 developers, "No check-ins until this seven-hour-long build passes!" Allowing check-ins during a broken test stage, though, only exacerbates the issue. A prolonged broken build undermines the purpose of testing as a quick feedback mechanism for code quality. The real solution is to make the test suite faster.

The larger the deployment scope and the higher the associated risks, the greater the chance of introducing issues. To minimize this risk, frequent deployments of small, well-tested changes are essential. When slow end-to-end tests hinder our ability to release these incremental changes, they can end up being counterproductive, creating more challenges than they solve.

### **The Metaversion**  
When working with end-to-end tests, it’s tempting to think, *I know these specific versions of the services work well together, so why not deploy them all at once?* This line of reasoning often leads to the suggestion, *Why not assign a version number to the entire system?* As Brandon Byars aptly puts it, "Now you have 2.1.0 problems."

By tying the versions of multiple services together, you essentially embrace the practice of changing and deploying several services simultaneously. This approach normalizes the idea of batch deployments, undermining one of the core benefits of a microservices architecture: the ability to independently deploy services without depending on others.

Unfortunately, this approach often leads to tighter coupling between services. Over time, services that were once distinct and independent become increasingly intertwined. Because these services are always deployed together, the underlying dependencies are never exposed, and their independence is never tested. Eventually, you find yourself in a situation where deploying one service is impossible without coordinating the deployment of others. At this point, the system resembles a tightly coupled monolith, defeating the purpose of adopting microservices in the first place.

This erosion of service independence undermines the flexibility and scalability that microservices are supposed to provide. Instead of enabling agility, it creates a tangled web of dependencies that makes deployments more complex and risk-prone. Simply put, it’s a step backward.

### **Lack of Independent Testability**  
Independent deployability is a recurring theme when discussing ways to enable teams to work autonomously and deliver software more efficiently. If teams are expected to operate independently, it naturally follows that they should also be able to test independently. However, as we've seen, end-to-end tests can undermine this autonomy, often requiring increased coordination between teams and introducing the challenges that come with it.

Achieving independent testability also involves rethinking how testing infrastructure is used. Shared testing environments, where multiple teams run their tests, are a common approach. However, these environments are often resource-constrained, and any issue in one team's tests can disrupt others, creating bottlenecks. To truly enable independent development and testing, each team should ideally have access to its own dedicated testing environment.

Research highlighted in *Accelerate* shows that high-performing teams tend to conduct most of their testing on demand, without relying on centralized or integrated test environments. This practice supports greater team autonomy and faster delivery cycles.

## Should You Avoid End-to-End Tests?

While end-to-end tests come with notable drawbacks, they can still be effective and manageable in systems with only a small number of microservices. In such cases, they may provide valuable insights and ensure system stability. However, as the number of services grows—3, 4, 10, or even 20—end-to-end test suites tend to become unwieldy. This can lead to an overwhelming increase in test scenarios, sometimes resembling a Cartesian explosion in complexity.

Even with a small number of microservices, managing end-to-end tests becomes challenging when multiple teams share a single suite. Shared test suites erode the principle of independent deployability, as deploying a service now hinges on the success of a test suite maintained across teams. This dependency undermines team autonomy and complicates deployment processes.

The primary goal of end-to-end tests is to ensure that deploying a new service doesn’t disrupt its consumers. As discussed in *“Structural Versus Semantic Contract Breakages,”* using explicit schemas for microservice interfaces can effectively address structural issues, reducing the reliance on extensive end-to-end testing. 

However, schemas cannot detect semantic breakages—behavioral changes that introduce backward incompatibility. End-to-end tests can identify these issues, but at a significant cost in terms of time, complexity, and maintenance. Ideally, we need tests that can catch semantic breaking changes while maintaining a narrower scope, improving test isolation and feedback speed. This is where contract tests and consumer-driven contracts become invaluable, offering a more efficient and targeted alternative to large-scale end-to-end tests.

### **Contract Tests and Consumer-Driven Contracts (CDCs)**  

Contract tests allow a team that relies on an external service to define how it expects that service to behave. These tests focus less on the functionality of the team’s own microservice and more on verifying the expected behavior of the external service. A significant advantage of contract tests is their ability to run against stubs or mocks representing the external service. Whether tested against a stub, a mock, or the actual service, the contract tests should always pass if the expectations are met.

Contract tests become even more powerful when implemented as part of **consumer-driven contracts (CDCs)**. In this approach, contract tests act as an explicit, programmatic description of how the consumer microservice (upstream) expects the producer microservice (downstream) to behave. These tests are shared with the producer team, enabling them to ensure their microservice meets these expectations. Typically, the producer team integrates these consumer contracts into their build pipeline, running them against their service during each build. Because these tests are executed against the producer in isolation, they are faster and more reliable compared to end-to-end tests.

For example, consider the Customer microservice with two consumers: the helpdesk and the web shop. Each consumer defines its expectations of how the Customer microservice should behave, resulting in two sets of contract tests. These tests focus solely on the Customer microservice and do not require other services to be deployed, making them similar in scope and performance to service tests. By isolating the test scope, CDCs enhance speed and reliability.

Collaboration between consumer and producer teams is crucial when creating these tests. For instance, team members from the helpdesk or web shop might pair with the Customer microservice team to ensure the contract tests accurately capture their needs. Beyond testing, CDCs promote clear communication and alignment between teams, reinforcing the idea that strong collaboration is key to microservice architecture—a notion aligned with Conway’s law.

CDCs are positioned at the same level as service tests in the testing pyramid, but with a different focus. They validate how a service meets its consumers’ needs. If a CDC fails during the Customer service build, it clearly indicates which consumer will be affected, prompting either a fix or a discussion about introducing a breaking change. This proactive approach helps catch issues before deployment, reducing reliance on expensive end-to-end tests.

![Integrating Consumer-Driven Tests into the Test Pyramid](TestPyramid2.svg)

#### **Pact**  

Pact is a widely-used consumer-driven testing tool initially developed at realestate.com.au and now open source. Originally built for Ruby and HTTP protocols, Pact has expanded to support multiple languages and platforms, including JavaScript, Python, .NET, and JVM-based ecosystems. It also works with messaging interactions.

With Pact, consumers define their expectations of the producer using a domain-specific language (DSL). These expectations are run locally against a mock Pact server to generate a Pact specification file (in JSON format). This file serves as both a formal contract and a stub for the local environment, eliminating the need for additional stubbing tools like mountebank.

Producers use the Pact file to verify that their microservice meets the defined expectations. This requires the producer to access the Pact file, which can be stored in a CI/CD artifact repository or managed with the Pact Broker. The Pact Broker supports versioning, validation tracking, and visualization of dependencies between microservices, making it a powerful tool for managing contracts.



#### **Other Options**  

While Pact is a popular choice, other tools like Spring Cloud Contract are available. However, Spring Cloud Contract is best suited for JVM-based ecosystems, whereas Pact offers greater flexibility across different technology stacks.



#### **More Than Tests—Conversations**  

Much like Agile stories serve as placeholders for conversations, CDCs encapsulate discussions about service APIs. They formalize these discussions into testable contracts and serve as triggers for further collaboration when tests fail. 

Effective communication and trust between consumer and producer teams are essential for CDCs to succeed. When teams are tightly integrated, this process is relatively straightforward. However, for external services or APIs with a large number of consumers, managing these contracts can be more challenging. In such cases, the producer may need to define contracts on behalf of consumers or collaborate with a representative subset to ensure compatibility. Regardless, CDCs remain critical for minimizing disruptions and maintaining reliable microservice interactions.

## Developer Experience

As developers increasingly work on multiple microservices, the overall developer experience can begin to decline. This often happens because they attempt to run an increasing number of microservices locally, particularly when they need to execute broad-scoped tests that require multiple non-stubbed services.

The extent of this challenge depends on several factors, including the number of microservices a developer must run locally, the technology stack used, and the capacity of their local machine. Some technology stacks, like those based on the JVM, are more resource-intensive, requiring significant computational power. Conversely, lighter-weight stacks can enable developers to run more microservices simultaneously on their local machines without performance issues.

One common solution to this problem is shifting development and testing to a cloud environment. With cloud resources, developers can access the computational power needed to run numerous microservices simultaneously. However, this approach has drawbacks. It requires consistent internet connectivity and can introduce delays in feedback cycles. For instance, making a local code change, building the artifact, and uploading it to the cloud can significantly slow down the development process—particularly in regions with limited internet bandwidth.

Cloud-based development environments, such as AWS Cloud9, offer a potential solution by enabling full development in the cloud. While promising, this approach hasn’t yet become a widespread standard for most developers. The complexity and cost associated with running numerous microservices in the cloud often outweigh the benefits.

Ideally, developers should only need to run the microservices they are actively working on. For example, if a team owns five microservices, each developer should be able to run those five services locally for efficient testing and fast feedback. Local development remains preferable for speed and responsiveness.

But what if these microservices need to interact with others owned by different teams? Without access to those additional services, local development environments may be incomplete. This is where stubbing becomes invaluable. Developers can set up local stubs to mimic the behavior of external microservices that are outside their scope. This ensures the local environment functions correctly without requiring all dependencies to be fully operational. 

In short, developers should only focus on running the microservices they are directly responsible for. If your organization expects developers to work across hundreds of microservices simultaneously, the real issue lies in the organizational structure—a deeper problem that warrants further exploration, as discussed in *“Strong Versus Collective Ownership.”*

## From Preproduction to In-Production Testing

Traditionally, the majority of testing efforts have been focused on preproduction environments. Through testing, we aim to create models that help us evaluate whether our system functions as expected, both in terms of functionality and nonfunctional requirements. However, no model is perfect, and when systems are used in real-world scenarios, unforeseen issues inevitably arise. Bugs make their way into production, unexpected failure modes emerge, and users interact with the system in ways we could never have anticipated.

A common response to these challenges is to increase the number and sophistication of tests, refining our models to catch more issues earlier in the development process. While this approach can reduce the frequency of problems, it eventually reaches a point of diminishing returns. No amount of preproduction testing can entirely eliminate the risk of failure once the system is live. The inherent complexity of distributed systems makes it nearly impossible to anticipate and address every potential issue before production.

The primary purpose of testing, broadly speaking, is to provide feedback on whether the software meets quality expectations. Ideally, this feedback should be delivered as quickly as possible, enabling developers to address issues before they impact users. This emphasis on early detection is why so much effort is devoted to predeployment testing.

However, restricting testing to preproduction environments limits our ability to identify issues effectively. It also prevents us from assessing the software’s quality in its most critical environment: the production system where it will ultimately be used.

To address this limitation, we should consider incorporating testing into the production environment. When done carefully and thoughtfully, in-production testing can be both safe and highly effective, offering insights that preproduction testing cannot match. In fact, many organizations are already engaging in some form of production testing, even if they don’t explicitly recognize it as such. By embracing this approach, we can uncover issues earlier, improve software quality, and ensure a better experience for end users.

### **Types of In-Production Testing**  

A variety of tests can be performed directly in production, ranging from basic to more advanced approaches. Let’s begin with something straightforward, like a ping check. A ping test verifies that a microservice is operational by confirming it is live. While often categorized as an "operations" task, this activity is fundamentally a test that is run frequently on the software to ensure availability.

Another common form of in-production testing is **smoke testing**, which is typically conducted during deployment. Smoke tests validate that the newly deployed software is functioning as intended. These tests are performed on the actual live system, ensuring that everything is operating correctly before making the software accessible to users.

**Canary releases**, discussed earlier, are also a form of in-production testing. This method involves releasing a new version of the software to a small subset of users to "test" its behavior in real-world conditions. If everything works as expected, the deployment can be expanded to a larger audience, often through automated processes.

Injecting **simulated user behavior** into the system is another type of in-production testing. For example, this might involve placing an order for a fictitious customer or registering a fake user in the production environment to verify that the system responds correctly. Although effective, this type of testing can sometimes raise concerns about its potential impact on the live system. To address such concerns, ensure these tests are designed to be safe and non-intrusive.

In-production testing offers valuable insights that preproduction tests cannot provide. By leveraging these methods responsibly, teams can better ensure the stability and reliability of their systems in real-world use.

### **Ensuring Safe In-Production Testing**  

When incorporating testing into a production environment (and it’s a practice worth considering), it’s essential to ensure these tests do not negatively impact the system. This means avoiding any actions that could destabilize production or compromise the integrity of production data. For example, running a simple ping to confirm a microservice is operational is generally a safe action. If a basic health check like this causes instability, it may indicate deeper systemic issues—unless, of course, the health checks themselves inadvertently become a form of internal denial-of-service attack.

Smoke tests are another safe option for in-production testing. These tests typically validate the functionality of the deployed software before it is made available to end users. As discussed in *“Separating Deployment from Release,”* decoupling the deployment process from the release phase is a valuable strategy. When testing software that is deployed but not yet released, the risks are minimized, making such tests inherently safer.

The greatest concerns about safety often arise when simulating user behavior in production. For instance, creating fake users or placing dummy orders requires extra caution to ensure these tests don’t trigger unintended consequences—like shipping fake orders or processing mock payments. While these challenges demand careful attention, the insights gained from such testing can be incredibly valuable. This topic will be explored further in *“Semantic Monitoring.”*

By designing in-production tests thoughtfully and with safety in mind, teams can reap the benefits of real-world testing without compromising system stability or user trust.

### **Prioritizing Mean Time to Repair Over Mean Time Between Failures**  

Techniques such as blue-green deployments and canary releases allow us to test software closer to, or even directly in, production environments. They also provide tools to handle failures more effectively when they occur. These strategies implicitly recognize that it’s impossible to catch every issue before releasing software.

In some cases, focusing on improving how quickly problems are resolved when they arise can deliver greater value than simply increasing the number of automated functional tests. This trade-off is often described in terms of optimizing for **Mean Time to Repair (MTTR)** versus **Mean Time Between Failures (MTBF)** in the field of web operations.

Reducing recovery time can be achieved through straightforward measures such as enabling rapid rollbacks and implementing robust monitoring systems (which we’ll discuss further in Chapter 10). By detecting issues in production early and responding quickly—such as rolling back problematic changes—the impact on customers can be minimized.

The balance between prioritizing MTBF and MTTR will differ for each organization and depends heavily on the consequences of production failures. However, many organizations that invest heavily in creating extensive functional test suites often neglect to focus on enhancing monitoring or recovery strategies. While this may lower the frequency of defects, it cannot eliminate them entirely, leaving the organization ill-prepared to respond effectively when issues do surface in production.

Other trade-offs can also influence testing strategies. For example, if the primary goal is to validate whether users will adopt a new feature or business model, it may be more valuable to release software quickly to gather real-world feedback rather than investing time in building a robust, defect-free solution. In such cases, the risk of introducing defects into production may be outweighed by the urgency of testing the idea itself. In these scenarios, skipping preproduction testing entirely can be a sensible choice.

## Cross-Functional Testing

While much of this chapter has focused on testing specific functionalities and how testing changes in a microservices architecture, another critical category of testing deserves attention: **cross-functional requirements (CFRs)**. These encompass system characteristics that cannot simply be implemented like standard features. They include aspects such as acceptable web page latency, the number of concurrent users the system should handle, accessibility for users with disabilities, or the security of customer data.

The term "nonfunctional requirements," often used to describe these characteristics, can feel misleading, as many of these aspects are deeply functional in nature. A former colleague, Sarah Taraporewalla, introduced the term **cross-functional requirements (CFRs),** which more accurately captures the idea that these behaviors emerge from cross-cutting efforts across the system.

Many CFRs can only truly be validated in a production environment. However, it is possible to define testing strategies that help track progress toward these goals. These types of tests often fall under the **Property Testing quadrant.** A common example is performance testing, which we’ll explore further later in this discussion.

In some cases, you might track CFRs at the level of individual microservices. For instance, you may demand high durability and uptime for a payment service, while accepting occasional downtime for a music recommendation service, as the latter would have less critical business impact. These trade-offs significantly influence the design and evolution of your system. The granular nature of microservices provides flexibility in making these trade-offs. CFRs for a particular service or team often surface as **service-level objectives (SLOs),** which we will delve into further in *“Are We Doing OK?”*.

Testing CFRs should follow the principles of the test pyramid. Some CFRs require end-to-end testing, such as load tests, but others can be addressed with more targeted tests. For example, if a performance bottleneck is identified during an end-to-end load test, a smaller-scoped test can be created to catch similar issues in the future. Certain CFRs, such as ensuring accessibility compliance in HTML markup, can be validated quickly without requiring extensive infrastructure or network calls.

Unfortunately, CFR considerations are often addressed too late in the development process. It’s essential to identify and review these requirements early and regularly to ensure they align with the system's evolving needs. By doing so, you can proactively design tests that ensure your system meets these critical cross-functional goals.

### **Performance Testing**  

Performance testing is an essential tool for validating whether cross-functional requirements, such as speed and efficiency, are being met. In a microservices architecture, where systems are broken into smaller components, the number of network calls increases. What might have previously required a single database call in a monolithic system could now involve multiple network calls to other services, each with its own database interactions. This added complexity can significantly impact system performance. Identifying and addressing sources of latency becomes critical, especially in synchronous call chains, where a single slow service can create a cascading effect, degrading the entire system's performance. This makes performance testing even more vital in a microservices setup than in traditional monolithic architectures. Unfortunately, performance testing is often delayed or neglected until just before the system goes live, if it’s done at all—an approach that should be avoided.

As with functional testing, a balanced approach is ideal for performance testing. You might decide to conduct tests on isolated individual services while also testing core system workflows. End-to-end journey tests can often be scaled to evaluate performance under heavier loads.

Effective performance testing requires running scenarios with an increasing number of simulated users to observe how latency varies as the load grows. These tests typically need time to execute and work best when conducted in environments that closely resemble production. This includes using production-like data volumes and infrastructure. Achieving this can be challenging, but even if the environment isn’t a perfect replica of production, these tests can still uncover performance bottlenecks. However, in such cases, be aware of the potential for false positives or negatives.

Given the time-intensive nature of performance tests, it may not be practical to run them after every code check-in. A common practice is to run a smaller subset of tests daily and a more comprehensive set weekly. Regardless of the schedule, it’s important to run them consistently. Delaying performance tests makes it harder to trace issues back to specific changes in the code, increasing the difficulty of resolving them.

It’s also crucial to actively review the results. Surprisingly, many teams invest considerable effort in implementing and running performance tests but fail to examine the output. This often happens because teams lack clear performance benchmarks. To avoid this, establish clear performance goals. For instance, if your microservice is part of a larger architecture, define service-level objectives (SLOs) that specify expected performance metrics. Automated tests should then provide feedback on whether these targets are being met or exceeded.

Even without specific performance benchmarks, automated tests can still highlight performance trends and serve as a safety net. For instance, you could configure tests to fail if there’s a significant performance drop between builds. This approach ensures that performance issues are identified early, reducing the risk of drastic degradation going unnoticed.

Lastly, performance testing should align with real-world system behavior, which can be better understood through production monitoring (discussed further in Chapter 10). Using the same tools for visualizing performance in both test and production environments helps you compare results effectively, ensuring the testing process remains relevant and actionable.

### **Testing for Robustness**  

In a microservice architecture, the overall system's reliability often hinges on its weakest component. To enhance reliability, many microservices incorporate features aimed at improving their robustness. These mechanisms, which we’ll explore further in *“Stability Patterns,”* include strategies such as deploying multiple instances of a microservice behind a load balancer to handle instance failures or implementing circuit breakers to manage situations where downstream services become unreachable.

Robustness testing involves simulating failure scenarios to ensure that the microservice can maintain its overall functionality despite disruptions. These tests can be challenging to implement because they require creating controlled failure conditions, such as introducing artificial network timeouts between the microservice under test and a stubbed external service. Despite the complexities, these tests are particularly valuable when developing shared features used by multiple microservices, such as a standardized service mesh that handles circuit breaking.

By investing in robustness tests, teams can validate that their microservices perform reliably under adverse conditions, ensuring a stronger and more resilient system overall.
