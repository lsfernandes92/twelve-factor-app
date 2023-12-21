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

* If there are multiple codebases, it does not qualify as a single codebase but rather constitutes a distributed system. Each component in a distributed system is considered an app, and each can individually adhere to the Twelve-Factor principles.
* Having multiple apps sharing the same codebase violates the Twelve-Factor approach. The recommended solution is to modularize shared code into libraries that can be included through the dependency manager.

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

* System resources like databases, Memcached, and backing services;
* Credentials for external services such as Amazon S3 or Twitter;
* Environment-specific values such as the hostname.

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

* The **build stage** involves a process that converts the code repository into an executable bundle, known as the build. This stage, utilizing a root commit specified by the deployment process, retrieves vendor dependencies and compiles binaries and assets.
* The **release stage** takes the output produced by the build stage and combines it with the deployment's configurations. The result of this combination creates a package ready for immediate execution within the execution environment.
* The **run stage** (also referred to as "runtime") executes the app within the execution environment by launching a set of the app’s processes against a chosen release.

**The Twelve-Factor app strictly emphasizes the separation between the build, release, and run stages.** For instance, making changes to the code at runtime is impossible, as there's no mechanism to propagate those changes back to the build stage.

Every release should have its unique identification associated, such as a timestamp, and each release is an increment of the previous version. This implies that any modification must generate a new release.

The build process is initiated by developers when the code is pushed to an environment. In contrast, runtime execution can occur automatically, such as during a server reboot. Consequently, it's crucial to keep the run stage with as few moving parts as possible, as issues at this stage might prevent the app from running, especially during off-hours when support might not be readily available. The build stage can be more complex, as developers actively manage the deployment, keeping errors in the foreground.


