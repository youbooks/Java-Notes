# Six reasons to version control your database

> 转载：[Six reasons to version control your database](https://www.red-gate.com/blog/database-devops/database-version-control-3)

For most application developers, it’s unthinkable to work without version control. The benefits of tracking and retaining an incremental history of code changes are long understood in the world of software development. No surprise then that 80% of respondents in our[ **2020 State of Database DevOps survey**](https://www.red-gate.com/solutions/database-devops/report-2020) confirmed they’re already using this practice for their application code.

But it was a different picture when we asked about database version control.

![2021-04-16-BSOCe1](https://image.ldbmcs.com/2021-04-16-BSOCe1.jpg)

Only 56% of those same people stated that they used version control for their database changes. In a way it’s understandable, as database version control was, for a long time, seen as unfeasible. But now that’s no longer the case, it’s time the database was treated in the same way as the application. The percentage of respondents implementing the practice is slowly increasing each year and, among sectors like Financial Services which are ahead of the curve in adopting database DevOps, the percentage is even higher at 63%.

Standardizing such processes across teams, projects, and across database and application code unlocks significant quality improvements and timesaving, as [**this blog post**](https://www.red-gate.com/blog/database-development/4-steps-to-laying-the-foundations-for-standardized-database-development) about laying the foundations explains. It outlines the benefits of standardized team-based development and talks about the four steps you can take to introduce it.

It’s important because databases are increasingly in the spotlight due to the recent proliferation of data breaches, a multitude of new regulations, and the increased pace of database development. In this climate, the need for an incremental history of changes and more efficient processes is more compelling than ever, so if you’re not already versioning your database code, here are some of the reasons why you really should be, and some of the benefits you’ll discover:

## 1. Ease collaboration across distributed teams

Putting database code into a version control system makes it much easier to share code changes and coordinate the work of the various team members who are responsible for the database.

The ability to rapidly share and manage changes makes it particularly important for teams based in different locations, and evidence shows that teams are increasingly distributed. The [**Stack Overflow 2019 Developer Survey**](https://insights.stackoverflow.com/survey/2019), for example, found that the majority of developers work remotely more than once a month, and 12% are full time remote workers. And in examples of the global Covid-19 situation, organizations have had to rapidly respond and adapt to distributed working.

With [**SQL Source Control**](https://www.red-gate.com/products/sql-development/sql-source-control/), team members can choose to work on a shared database, safe in the knowledge that they won’t be over-writing each other’s work. With features like object locking, conflicts are easily avoided and developers can check code in and out with confidence.

Once the database code is in version control, the path is also paved for**[ adopting a dedicated development environment](https://www.red-gate.com/blog/database-devops/easing-the-transition-from-shared-to-dedicated-development)** approach. Giving each developer their own dedicated, up-to-date copy of the latest version of the database brings in the freedom to try out new things, risk-free.

## 2. Gain better visibility of the development pipeline

A version control system provides an overview of what development work is going on, its progress, who’s doing it, and why. It also maintains detailed change histories and can be associated with issue tracking systems. For example, SQL Source Control lets you associate database tasks with Microsoft’s Azure DevOps Server work items so that you have a complete view of your workflow.

## 3. Have the ability to roll back or retrieve previous versions of the database

While you should always have a reliable backup strategy in place, getting a database into version control also provides an efficient mechanism for backing up the SQL code for your database. Because the history it provides is incremental, version control lets developers explore different solutions and roll back safely in the case of errors, maintaining the referential integrity of your database while giving you a risk-free sandbox. SQL Source Control makes it easier by allowing users to simply roll back and resolve conflicts straight from the Object Explorer.

## 4. More readily demonstrate compliance and auditing

The change tracking provided by version control is the first step to getting your database ready for compliance, and an essential step in maintaining a robust audit trail and managing risk. Compliance auditors will require an organization to account for all changes to a database, and detail all those with access to it.

New data protection regulations are emerging all the time, and requests to demonstrate compliance or view the history of a database are increasingly frequent. With SQL Source Control, you can look through the full revision history of a database or database object and see exactly who made the changes, when they made them, and why.

## 5. Lay solid foundations for automating database deployments

The State of Database DevOps Survey also revealed that there is a growth of developers moving into full stack development, taking on tasks for developing both the application and the database: 

![2021-04-16-MNOTLw](https://image.ldbmcs.com/2021-04-16-MNOTLw.jpg)

This is encouraging because version controlling database code opens the door for automated deployments, which is common practice in application development.

This in turn removes the bottleneck created by not synchronizing application and database deployments. Complex processes become easier to automate and more repeatable, and deployments much more predictable because you’re working with a stable version of the database, which is being developed alongside the application.

Using code checked into SQL Source Control as the basis for the automated builds and tests run by [**SQL Change Automation**](https://www.red-gate.com/products/redgate-deploy/), for example, means that problems are found earlier, and higher quality code is shipped and deployed.

## 6. Synchronize database and application code changes

The 2020 State of Database DevOps Report found that the top two challenges in integrating database changes into a DevOps process are *synchronizing application and database changes* and *overcoming different approaches to application and database development*.

Having the database in version control alongside the application immediately addresses both concerns, because you always know the version of the database being deployed directly corresponds to the version of the application being deployed. This close integration helps to ensure better coordination between teams, increases efficiencies, and helps when troubleshooting issues. To make it easier, SQL Source Control plugs into version control systems used for storing application code changes like Git, Azure DevOps Server and Subversion.

## 7. Summary

While it’s true that database version control wasn’t always achievable, the availability of tools like SQL Source Control means it’s now easier than ever to version database code alongside application code across your organization.

Take your next step towards standardizing and automating your software delivery and become a truly high performing IT organization. If you’re among the 44% not yet version controlling your database, hopefully at least one of the six reasons above will have resonated enough for you to explore it further.