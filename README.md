# The Twelve-Factor app (https://12factor.net/pt_br/)

## Introduction

The Twelve-Factor app aims to provide the following benefits to your modern application:

- A well-organized or **declarative** setup automation that aims to simplify the onboarding process for new developers joining the project and reduce the time and costs associated with that task;
- Provides a **clean contract** with the operating system, **maximizing portability** between execution environments;
- Aims to facilitate easy **deployment** on modern **cloud platforms**, eliminating the need for servers and system administration;
- **Minimizes discrepancies between environments**, ensuring all environments have the latest features up and running, enabling **continuous deployment** for maximum agility;
- Enables **scaling up** without requiring major changes, such as altering architecture, tooling, or development practices.

This can be implemented in apps written in any programming language and utilizing any combination of backing services (database, queue, memory, cache, etc).

The document leverages developers' experiences and observations regarding the wide variety of software-as-a-service available in the industry. The document was created by developers who have been involved in the development, deployment, operation, and scaling of hundreds of apps through their work on the Heroku platform. It focuses on ideal practices for app development, particularly emphasizing the organic growth of an app over time, fostering collaboration dynamics among developers working on the codebase, and avoiding the costs associated with software erosion.

The motivation behind this document was to raise awareness of systemic problems encountered in modern application development, establish a shared vocabulary for discussing these issues, and offer solutions to address these challenges.

## I. Codebase

### One codebase tracked in revision control, many deploys

---

A codebase refers to the main code definition that serves as shared code among developers for ongoing feature development. Typically, a codebase is stored in a code repository managed by a version control system like `Git`, `Mercurial`, or `Subversion`.

The codebase is a single repository or any set of repositories that share a root commit.

There exists a one-to-one correlation between the codebase and the app:

- If there are multiple codebases, it does not qualify as a single codebase but rather constitutes a distributed system. Each component in a distributed system is considered an app, and each can individually adhere to the Twelve-Factor principles.
- Having multiple apps sharing the same codebase violates the Twelve-Factor approach. The recommended solution is to modularize shared code into libraries that can be included through the dependency manager.

Per app, there exists only one codebase, but numerous deployments of the app can exist. A deployment refers to a running instance of the app, which commonly includes instances in production, staging, testing, and an additional local instance for each developer.

While the codebase remains consistent across all these environments, there might be different active versions in each environment due to ongoing feature development during the development cycle.

## II. Dependencies

### Explicitly declare and isolate dependencies

---

Most programming languages have their designated platforms for sharing and posting system packages. For instance, Ruby utilizes `Rubygems`.

Each library can be installed through what we refer to as "site packages," which serve system-wide, or by explicitly declaring them within our app's scope, a practice known as "vendoring."

**The Twelve-Factor methodology does not rely on the implicit existence of system-wide packages. **Instead, dependencies should be declared within our application via a declaration manifest. Additionally, it employs dependency isolation tools during execution to prevent any implicit dependencies from seeping in from the surrounding system. For instance, Ruby employs the `Gemfile` manifest format for dependency declaration and `bundle exec` for dependency isolation.

Explicitly declaring dependencies brings the benefit of simplifying setup for new developers joining the app. A new developer can clone the app's codebase onto their local machine, needing only the language runtime and the installed dependency manager as prerequisites. Consequently, they can set up everything required with a single command, such as `bundle install` for Ruby/Bundler.

The Twelve-Factor methodology also does not rely on the implicit existence of system tools like `curl` or `ImageMagick`, for example. Even though these tools may be prevalent in many systems, there's no assurance that they will exist on all systems where the app may run in the future, or that they will be compatible with the app. All of those should be "vendored" into the app.

## III. Config

### Keep your configs for each environment

---

The app's configuration is everything that may vary between environments. This includes:

- System resources like databases, Memcached, and backing services;
- Credentials for external services such as Amazon S3 or Twitter;
- Environment-specific values such as the hostname.

The Twelve-Factor methodology strongly advocates for separating configuration from the code. This is primarily because configuration values may differ between environments while the code remains constant. A useful way to determine if configurations are properly extracted from the code is by considering the scenario of making the codebase open source. If the credentials need to be managed, the configurations might be compromised.

Another approach involves using configuration files not included in the version control system, such as 'config/database.yml' in Rails. While this is a significant improvement, it is not recommended because there's a risk of accidentally pushing these files. Additionally, configuration files often scatter across different locations, making maintenance challenging. Moreover, these config files tend to be specific to a particular language or framework.

Another practice to avoid is grouping configurations into environment-specific sets. Apps tend to categorize configurations into named groups for different environments. However, this method doesn't scale well. As the project expands, developers might introduce their own custom environments, leading to an explosion of configurations that complicates deployment management.

**The Twelve-Factor methodology suggests storing configurations in environment variables.** Environment variables are easy to modify between deployments without requiring changes to the code. They compel setting up independent configurations for each deployment environment, they minimizing the chance of accidental check them in the codebase repository. They don't aggregate into 'environments' groups, they are language- and OS-agnostic standard, and provide a scalable model as the app naturally expands to more deployments.

## IV. Backing services

### Treat backing services as attached resources

---

Backing services classify as any resources required by the app to fulfill its functionalities. Typically accessed via a network or other locations stored within the configuration. These services include databases, SMTP clients, third-party integrations such as `Amazon S3`, `Twitter`, or `Last.fm` APIs, and caching systems. These backing services are usually managed by the individual responsible for deploying the app and its resources.

**The Twelve-factor app doesn't differentiate between local and third-party services.** For the app, both types of resources are accessed via URLs. Thus, swapping out a local resource, like a database, for one managed by a third party requires no code changes.

Resources can be attached to or detached from deployments at will. For instance, if a database malfunctions, the app's administrator might spin up a new database backup, detach the current database, and then the restore the new database instance by requiring no code alterations.

## V. Build, release, run

### Strictly separate build and run stages

---

A codebase goes through several stages before transforming into an execution environment:

- The **build stage** involves a process that converts the code repository into an executable bundle, known as the build. This stage, utilizing a root commit specified by the deployment process, retrieves vendor dependencies and compiles binaries and assets.
- The **release stage** takes the output produced by the build stage and combines it with the deployment's configurations. The result of this combination creates a package ready for immediate execution within the execution environment.
- The **run stage** (also referred to as "runtime") executes the app within the execution environment by launching a set of the app’s processes against a chosen release.

**The Twelve-Factor app strictly emphasizes the separation between the build, release, and run stages.** For instance, making changes to the code at runtime is impossible, as there's no mechanism to propagate those changes back to the build stage.

Every release should have its unique identification associated, such as a timestamp, and each release is an increment of the previous version. This implies that any modification must generate a new release.

The build process is initiated by developers when the code is pushed to an environment. In contrast, runtime execution can occur automatically, such as during a server reboot. Consequently, it's crucial to keep the run stage with as few moving parts as possible, as issues at this stage might prevent the app from running, especially during off-hours when support might not be readily available. The build stage can be more complex, as developers actively manage the deployment, keeping errors in the foreground.

## VI. Processes

### Execute the app as one or more stateless processes

---

The app operates within the execution environment as one or more processes.

Consider the code as a stand-alone script, the execution environment as the developer's local machine with the installed language runtime, and the process launched via the command line, for example, `ruby my_file.rb`.

**The Twelve-factor app dictates that processes should be stateless and share nothing.** Any additional data requiring persistence must be stored in a backing service, typically a database.

However, the memory space or filesystem of the process can be utilized as a brief, single-transaction cache. For instance, downloading a large file, performing operations on it, and then storing the results of the operation in the database. Nonetheless, the Twelve-Factor app never assumes that anything cached in memory or on disk will be available for future requests or jobs, as future requests might be served by different processes.

Moreover, any language-specific packaging that utilizes the filesystem as a cache for compiled assets violates the Twelve-Factor app. It is preferred to handle this during compilation in the build stage. _Tip: The Rails asset pipeline can be configured to package assets during the build stage._

Certain web systems rely on "sticky sessions" where user session data is cached in the memory of the app’s process, expecting subsequent requests from the same visitor to be routed to the same process. This practice also violates the Twelve-Factor app. Session state data is better suited for a datastore offering time expiration, such as Memcached or Redis.

## VII. Port binding

### Export services via port binding

---

At times, the server functions as a web server container where the web app operates as a module within the container. For instance, a PHP app might run as a module inside Apache HTTPD, or Java apps might function within Tomcat.

**The twelve-factor app is entirely self-contained** and doesn't depend on the runtime injection of a web server into the execution environment to establish a web-facing server. Instead, **the web app exports HTTP as a service by binding to a port**, which serves as a listening port for incoming requests. In a local environment, developers access the app by visiting `http://localhost:3000`. In a deployment environment, a routing layer manages the routing of requests from a public-facing hostname to the port-bound web processes.

This is commonly achieved by using dependency declarations to incorporate the web server library into the app, such as using `Thin` for `Ruby`.

Furthermore, this approach signifies that any service can be exported by port binding, awaiting incoming requests. It's important to note that by doing so, one app can serve as the backing service for another app.

## VIII. Concurrency

### Scale out via the process model

---

Web applications have various process execution forms. PHP, for example, processes function as child processes of Apache, starting based on request volume, while Java processes operate within a massive JVM uberprocess, managing concurrency internally through threads. These running processes are minimally visible to app developers. **In the context of the twelve-factor app, processes are crucial and they are a first-class citizen**, drawing inspiration from the unix process model for service daemons. This model allows developers to allocate different workloads to specific process types, such as handling HTTP requests through web processes and long-running tasks through worker processes.

Moreover, while individual processes can handle internal multiplexing via threads or asynchronous/event-driven models, vertical scaling of an individual VM has limitations. The process model excels in scaling out, as the share-nothing, horizontally partitionable nature of twelve-factor app processes facilitates simple and reliable addition of concurrency.

Furthermore, the text emphasizes that twelve-factor app processes should refrain from daemonizing or writing PID files. Instead, reliance on the operating system’s process manager (like `systemd`` or cloud platform-based distributed process managers) or tools such as `Foreman` during development is encouraged. These tools effectively manage output streams, handle crashed processes and address user-initiated restarts and shutdowns.

## IX. Disposability

### Maximize robustness with fast startup and graceful shutdown

---

**The Twelve-Factor app's processes are designed to be disposable, allowing them to be started or stopped at any moment.** This flexibility accelerates elastic scaling, rapid deployment of code or configuration changes, and ensures the resilience of production deployments.

Efforts should be made to **minimize the startup time of processes.** Swift startups provide greater agility for the release process and scaling up, contributing to robustness as the process manager can easily transfer processes to new physical machines when needed.

**Processes shut down gracefully when they receive a `SIGTERM`** signal from the process manager. In the case of a web process, this entails ceasing to listen on the service port (rejecting new requests), allowing ongoing requests to complete, and then exiting. For instance, in the case of a worker process, graceful shutdown involves returning the current job to the work queue. In RabbitMQ, a worker can send a NACK to accomplish this.

Processes should also be resilient against abrupt termination, which could result from hardware failures. Although less common than graceful shutdowns with SIGTERM, such incidents can occur. An advisable strategy involves utilizing a robust queueing backend like Beanstalkd, which returns jobs to the queue upon client disconnection or timeouts. In any scenario, a twelve-factor app is structured to handle unexpected, non-graceful terminations. The crash-only design embodies this principle to its logical conclusion.

## X. Dev/prod parity

### Keep development, staging, and production as similar as possible

---

Historically, significant disparities have existed between development and production environments, manifesting in several ways:

- The Time Gap: Code often takes days, weeks, or even months to reach production, whereas it should ideally be deployed within hours or minutes.
- The Personnel Gap: Developers create code while DevOps engineers handle deployment. Ideally, developers would be closely engaged in deploying and observing its behavior in the production environment.
- The Tools Gap: Local development setups might utilize stacks like Nginx, SQLite, and OS X, while production environments employ Apache, MySQL, and Linux. Let's keep development and production as similar as possible.

**The Twelve-Factor app is engineered for continuous deployment, aiming to narrow the gap between these environments.**

Developers adhering to **the twelve-factor principles resist the temptation to use different backing services in development versus production.** This stems from the appeal developers find in using lightweight services locally versus more robust ones in production. Discrepancies between these services often lead to minor incompatibilities, causing code that passed testing in dev or staging to fail in production. This discourages continuous deployment and results in considerable friction that incurs significant costs throughout an application's lifecycle. Additionally, modern packaging systems like `Homebrew` and `apt-get` have made installing and running modern backing services less challenging.

Many programming languages offer libraries that streamline access to backing services, providing adapters for various service types. For instance, `ActiveRecord` for `Ruby/Rails` includes database adapters such as `MySQL`, `PostgreSQL`, and `SQLite`. While adapters to different backing services simplify porting to new services, it's essential for all deployments (developer environments, staging, production) to use the same type and version of each backing service.

## XI. Logs

### Treat logs as event streams

---

Logs offer insights into the behavior of a running app, typically stored in log files on the disk containing the app's outputs.

They represent an aggregated, time-ordered stream of events occurring across running processes and backing services, registering each event on a single line. Logs flow continuously without a fixed start or end as long as the app operates.

**A twelve-factor app doesn't concern itself with routing or storing its output stream.** Rather than managing log files, the focus is on directing processes and backing services to output logs to `stdout`.

During local development, developers can observe logs in their terminal's foreground to monitor the app's behavior. In other environments like `test`, `staging`, or `production`, this is captured by the execution environment. All streams from the app, its processes, and backing services are directed to one or more final destinations for viewing and long-term storage, entirely managed by the execution environment.

The app's event stream can be directed to a file or monitored in real-time using terminal-based tail commands. Importantly, it can also be sent to a log indexing and analysis system or a general-purpose data warehousing system like `Hadoop` or `Hive`. These systems offer significant power and flexibility for examining an app's behavior over time, including:

- Find specific time events in the past;
- Large-scale graphing of trends(such as the requests per minute);
- Active alerting according to user-defined heuristics(such as an alert when the quantity of errors per minute exceeds a certain threshold)

## XII. Admin processes

### Run admin/management tasks as one-off processes

---

Besides the app's regular business processes, developers frequently need to perform one-off administrative or maintenance tasks for the app, such as:

- Running database migrations;
- Running a console to run arbitrary code or inspect the app's model against the live database;
- Running one-time scripts committed into the app's repo(e.g. `php scripts/fix_bad_records.php`)

Those tasks should operate within the same environment as the regular long-running processes of the app. They execute against a release, utilizing the identical codebase and configuration. Moreover, to prevent synchronization issues, the code must be shipped alongside the application code.

All process types should employ the same dependency isolation techniques. For instance, if the Ruby web process utilizes the command `bundle exec thin start`, the database migration should also use `bundle exec rails db:migrate`.

The Twelve-Factor app strongly advocates for languages that offer a built-in REPL shell, simplifying the execution of one-off administrative scripts. During development, developers can run these scripts by accessing a local console within the app's container. In a production deployment, developers can access the app's console via `SSH` or another mechanism provided by the execution environment, and then execute the script.

## Conclusion

- The application of those rules is better to "prepare-to-be-ready" your SaaS
- Those rules doesn't make sense in small projects
- Those rules are suggested for a better SaaS service, but they are not require.
  You can apply the one that make sense to you or to your project.
- It's better que know the factors just in case
