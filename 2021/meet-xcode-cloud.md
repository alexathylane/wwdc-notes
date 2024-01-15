# Meet Xcode Cloud (WWDC 2021)

Xcode Cloud is an easy-to-use continuous integration and delivery service for Apple developers.

## Agenda

- Xcode Cloud overview
- Set up your project
- View Results
- Collaborate with your team

## Xcode Cloud overview

CI is the practice of integrating code changes regularly so that issues are caught and changed early.

A typical CI workflow is a set of automated states that runs when you or a team member push a change to a repository.

![Xcode Cloud flow](assets/xcode-cloud-flow.png)

Xcode Cloud builds upon the idea of CI while connecting the dots between Apple developer tools to provide you with the complete development pipeline to build, test, distribute, collect feedback, and quickly iterate on your projects.

A workflow is a configuration that tells Xcode Cloud which actions to perform and when to perform them.

The result of running a workflow is called a build. Xcode Cloud runs builds in Apple managed Cloud infrastructure that provides code signing and access to multiple operating system versions and Xcode releases. When you click on your app in the Cloud tab of the report navigator, you can view the status of all of your workflows and the latest builds in the sidebar.

![Xcode Cloud UI](assets/xcode-cloud-ui.png)

Everything we just saw is not only available in Xcode, but it’s also available in App Store Connect. This includes starting and viewing builds, managing workflows, viewing and downloading artifacts, sharing results with your team, and managing notification settings. And if you’re already working in TestFlight, Xcode Cloud is the tab right next door in App Store Connect for quick access.

## Setup your project

The process begins by visiting the Xcode Cloud section of the product menu and selecting Create Workflow.

Next, I select which app I’d like to onboard from these options detected for my local project. Today, I’ll pick this new smoothie ordering app my team is developing called “Fruta.” The app supports both iOS and macOS, and we’ll onboard both platforms together at once.

Our app begins with a default first workflow created automatically for me. By inspecting my local project, Xcode Cloud can tailor these initial workflow settings to match my team’s existing configurations.

Workflows are made up of a start condition, an environment, the set of actions to be performed, and post-actions such as deployments and notifications.

And I can see Xcode Cloud selected every push to the main branch as the start condition, the latest released Xcode for my environment, and archive actions for both iOS and macOS. I have the option of changing these settings, but this looks good to me, so I’ll continue on. For a deep dive into workflow editing, check out the “Explore Xcode Cloud Workflows” session later in the conference.

Next, I’ll authorize Xcode Cloud to access my source code. This is a one-time action covering all source repositories required to build my project including the primary repo, any submodules, and private Swift packages. For any publicly accessible repositories, no additional authorization is necessary.

Xcode Cloud discovered the two private repos within my project, so, next, we’ll grant explicit permissions to GitHub where the source is hosted. Clicking Grant Access takes me to App Store Connect with more details about the next steps. It’s important to note that this process will vary depending on the source provider, and I can revoke access at any time for any reason.

In the final step, Xcode Cloud will offer to register my application and bundle ID to App Store Connect. Our application is already created, so I’ll just confirm the details here. Everything looks good.

## View Results

The build group overview shows my active and completed builds at a glance, and clicking the first entry opens up the overview page for our build.

This overview shows me brief details about the build such as durations and environment configurations, with the lower section showing me the status of all actions and post-actions involved.

![xcode-cloud-overview-page](assets/xcode-cloud-overview-page.png)













