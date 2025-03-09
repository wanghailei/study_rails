# Deploy

## Container

==A container is a lightweight virtual machine.== It has its operating system and all, except that unlike virtual machines they don’t simulate the entire computer, but rather create a sand boxed environment that pretends to be a virtual machine. It runs the smallest possible version of the software.

You can theoretically install an operating system like Ubuntu 20 with all its bells and whistles in a container, but what is typically done by professionals is that they build a stripped down version of the operating system that can do one job only.

==When time comes to deploy my app to production, I don’t need to set up separate servers==. I just save my containers into image files then just go to Amazon or Google and deploy with docker to the server instances, without caring what operating system is running on those servers.

## Docker

Docker is a tool that lets you deploy apps in containers. ==Docker is a way to build and run those containers==.

==Docker wipes differences across production, staging and development environments.==

The same applies to the development environments of different developers. In other words: no more “works on my machine”.

Ensure that the exact same environment can be deployed wherever Docker is installed, no matter the hardware or operating system. Good old “build once, run everywhere”.

A docker file is a simple instruction file that tells docker to download an image, then run some commands on it, such as install additional software, etc.

## Kamal

If you’d like the freedom to move between cloud and your own hardware, or even mix the two, Kamal is much simpler. 

Kamal offers zero-downtime deploys, rolling restarts, asset bridging, remote builds, accessory service management, and everything else you need ==to deploy and manage your web app in production with Docker==. 

You can see everything that’s going on, it’s just basic Docker commands being called. 

No need to ensure that the servers have just the right version of Ruby or other dependencies you need. That all lives in the Docker image now. ==You can boot a brand new Linux server, add it to the list of servers in Kamal, and it’ll be auto-provisioned with Docker, and run right away.== And the images built for Kamal can be used for CI or later introspection.



## If there is Docker, why Kamal?

Docker is a well-established platform for creating, deploying, and running applications in containers, but Kamal is a more specialized deployment tool designed to make it easier to manage containers in production, particularly for Ruby on Rails applications. While Docker is about containers in general, Kamal focuses on orchestrating the entire deployment process with simplicity in mind.

While Docker is a general-purpose containerisation tool, **Kamal** is a higher-level, Rails-focused deployment tool that simplifies the process of shipping apps to production by combining Docker’s power with built-in features for handling typical Rails app deployment tasks. If you need fine-grained control over containers for various apps, Docker is a strong choice. If you’re deploying Rails apps and want to automate much of the deployment complexity, Kamal may be a better fit.

Here's why you might choose **Kamal** over Docker alone:

1. **Simplified Deployment**: Kamal abstracts away much of the complexity involved in deploying applications with Docker. It combines Docker’s power with an opinionated setup that streamlines the process for deploying Ruby on Rails apps to production servers.

2. **Focus on Ruby on Rails**: Kamal is tightly integrated with Rails and optimised for typical Rails workflows. It knows how to handle things like asset precompilation, database migrations, and background jobs in a Rails-specific manner, whereas Docker requires you to handle these tasks manually or set up additional automation scripts.

3. **Zero-Downtime Deployments**: Kamal focuses on making zero-downtime deployments easier to achieve. It manages server processes in a way that ensures your Rails app stays live even during the deployment process.

4. **Built-in Rollbacks**: Kamal has built-in features for rolling back to a previous version if something goes wrong during a deployment, which simplifies failure recovery compared to manually managing container versions in Docker.

5. **Simple Configuration**: Kamal’s configuration is simple and focused on production deployments, whereas Docker and Docker Compose require more flexibility and boilerplate to manage things like services, environments, and networking.

6. **Server Management**: Kamal handles server management tasks that Docker itself doesn’t cover, like restarting the app server after deployment, health checks, and even SSHing into the production server. It wraps around these tasks in a way that integrates them tightly with the deployment flow.



Not exactly. Kamal is not a specialised version of Docker itself, but rather a **deployment tool** that **leverages Docker** to handle containerisation and orchestrate deployments. While Docker provides the foundation by packaging applications into containers, Kamal builds on top of that by simplifying the deployment process, particularly for Ruby on Rails applications.

You could think of Kamal as a tool that **specialises in using Docker** for managing and deploying web apps (especially Rails apps) in production environments, but Kamal is not a container runtime or platform like Docker. Instead, it manages Docker containers, handles server orchestration, and streamlines tasks like deployment, rollback, and service restarts, using Docker as a key part of its workflow.

So a more accurate description might be:
- "Kamal is a specialised deployment tool that uses Docker to streamline the management and deployment of applications, particularly for Ruby on Rails."

Does that distinction make sense?
