# The twelve-factor app (https://12factor.net/pt_br/)

## Introduction

The Twelve-Factor app aims to provide the following benefits to your modern application:

- A well-organized or declarative setup automation that aims to simplify the onboarding process for new developers joining the project and reduce the time and costs associated with that task;
- Provides a clean contract with the operating system, maximizing portability between execution environments;
- Aims to facilitate easy deployment on modern cloud platforms, eliminating the need for servers and system administration;
- Minimizes discrepancies between environments, ensuring all environments have the latest features up and running, enabling continuous deployment for maximum agility;
- Enables scaling up without requiring major changes, such as altering architecture, tooling, or development practices.

This can be implemented in apps written in any programming language and utilizing any combination of backing services (database, queue, memory, cache, etc).

The document leverages developers' experiences and observations regarding the wide variety of software-as-a-service available in the industry. The document was created by developers who have been involved in the development, deployment, operation, and scaling of hundreds of apps through their work on the Heroku platform. It focuses on ideal practices for app development, particularly emphasizing the organic growth of an app over time, fostering collaboration dynamics among developers working on the codebase, and avoiding the costs associated with software erosion.

The motivation behind this document was to raise awareness of systemic problems encountered in modern application development, establish a shared vocabulary for discussing these issues, and offer solutions to address these challenges.

## I. Codebase

A codebase refers to the main code definition that serves as shared code among developers for ongoing feature development. Typically, a codebase is stored in a code repository managed by a version control system like Git, Mercurial, or Subversion.

The codebase is a single repository or any set of repositories that share a root commit.

There exists a one-to-one correlation between the codebase and the app:

* If there are multiple codebases, it does not qualify as a single codebase but rather constitutes a distributed system. Each component in a distributed system is considered an app, and each can individually adhere to the twelve-factor principles.
* Having multiple apps sharing the same codebase violates the twelve-factor approach. The recommended solution is to modularize shared code into libraries that can be included through the dependency manager.

Per app, there exists only one codebase, but numerous deployments of the app can exist. A deployment refers to a running instance of the app, which commonly includes instances in production, staging, testing, and an additional local instance for each developer.

While the codebase remains consistent across all these environments, there might be different active versions in each environment due to ongoing feature development during the development cycle.

## II. Dependencies

