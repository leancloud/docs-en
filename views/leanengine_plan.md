{% import "views/_helper.njk" as docs %}

# LeanEngine Plans and Prices

In this documentation, you will learn how you can choose the plan that best fits your needs, as well as how you can manage your instances and use the group management function.

## Trial Instances

Each application created on LeanCloud comes with a free trial instance with 0.5 CPU and 256 MB memory. You can use this instance for learning and testing purposes.

A trial instance will stop responding to requests when deployment is in progress. It will also start [hibernating](#hibernating) if no request is made in the past 30 minutes (there will be a delay for processing requests when resuming), and can run at most 18 hours each day.

## Standard Instances

You can purchase standard instances for projects that demand high availability. A standard instance runs 24 hours a day and will not hibernate even there are no requests. A staging environment will also be provided for testing. If you have 2 or more standard instances, you can even enable functions like load balancing and rolling deployment, which further increases the availability of your application.

**Staging Environment**<br>A trial instance will be provided in the staging environment so you can test your project before having it published.

**Increased Availability**<br>Using 2 or more instances not only increases the computing power of your application but also enables *load balancing* and *disaster recovery* as different instances of an application will be running on different servers. If one of these instances runs into error or the server containing that instance encounters hardware failures, failover will be performed automatically so that further requests will be diverted to other instances.

{% call docs.noteWrap() %}
If you have only one instance and the server containing that instance runs into an error, we will still migrate your instance to another server within several minutes. However, your application will not be able to process requests while the migration is in process. {{ docs.alertInline("If your application demands high availability, we recommend that you enable 2 or more instances for it.") }}
{% endcall %}

**Rolling Deployment**<br>When deploying your project, the system will first open up new instances to run the newer version of it. The ones running the older version will not be shut down until the new instances are properly started.

**Group Management**<br>You can create multiple groups of instances with different domains assigned to each of them. All of these groups can access the same data in LeanStorage. See [Group Management](#group-management).

If you don't want to be billed anymore, please make sure that all the standard instances in the production environment of your application are deleted. You will still have your trial instance available.

Each instance has its own RAM and file system.
Therefore, data stored in the global variables or the file system of a single instance will not be accessible from other instances.
To share data among instances, you can use LeanCache.

## Hibernating

{% call docs.noteWrap() %}
Standard instances will never hibernate.
{% endcall %}

Trial instances will **hibernate** under certain circumstances.

- A trial instance will start hibernating if no request is made in the past 30 minutes.
- A hibernating instance will resume when a request comes in. It may take 2 to 10 seconds for the instance to get back running and have the request processed. The requests made after that will be responded immediately.
- If a trial instance has run more than 18 hours in the past 24 hours, it will be forced to hibernate and incoming requests will receive `503` as the error code. You can view error logs in your app's [Dashboard > LeanEngine > Statistics](https://console.leancloud.app/cloud.html?appid={{appid}}#/stat).

{% call docs.noteWrap() %}
If you don't want your trial instance in the staging environment to hibernate, or need to simulate the production environment where there are multiple instances, you can purchase standard instances for the staging environment.
{% endcall %}

## Managing Instances

An instance with **0.5 CPU and 256 MB memory** is defined as a basic instance. The values here indicate the maximum hardware resources that can be used by this instance, that is, 50% of a CPU and 256 MB of memory.

You can create combinations of basic instances according to your needs. For example:

- Create 4 instances with 1 basic instance for each of them (4 instances with 0.5 CPU and 256 MB memory for each).
- Create 2 instances with 2 basic instances for each of them (2 instances with 1 CPU and 512 MB memory for each).

{# TODO #}You can have at most 12 instances for each application. Please contact us at support@leancloud.rocks if you need more.

### Upgrading to Standard Instances

You can go to your app's [Dashboard > LeanEngine > Instances](https://console.leancloud.app/cloud.html?appid={{appid}}#/leannode) and click on **Upgrade to standard** to have your trial instance upgraded to a standard instance.

### Creating and Deleting Instances

After upgrading to standard instance, you can create more instances by clicking on **Create instance**.

To delete an instance, click on the gear icon on the top-right corner of the instance and then click on **Delete**. The data in your application will not be affected, but the application's ability to handle concurrent requests will be impaired.

You can increase the capacity and availability of your application by increasing the number of instances, but it doesn't mean that you should create as many instances as you can. For example, using more than 8 instances will tremendously increase the time needed for each deployment (since more instances need to be restarted). We recommend that you have **2 to 4** instances for each application.

- For an application with low stress, we recommend that you use 2 instances with 0.5 CPU and 256 MB memory for each. This helps you increase the availability of your application.
- For an application with high stress, we recommend that you use 4 to 6 instances with 4 CPU and 2 GB memory for each. This helps you avoid creating too many instances.

See [Usage Data](#usage-data) to learn about how you can view the stress of your application.

### Resizing Instances

A standard instance has 0.5 CPU and 256 MB memory by default. You can change the size of all instances (including trial instances in the staging environment) by clicking on **Resize**.

We recommend that you increase the sizes of your instances when:

- The usage of **CPU** constantly goes above **30%**.
- The usage of **memory** constantly goes above **70%**.

See [Usage Data](#usage-data) to learn about how you can view the usage of your application.

{% call docs.noteWrap() %}
**Utilizing Large-Size Instances**<br>For a Node.js application, a thread can make use of at most 100% CPU and 1.5 GB memory by default. If an instance has a size way larger than this, the instance may not be fully utilized. You may consider implementing [multithreading](leanengine_webhosting_guide-node.html#multithreading) with `cluster`.
{% endcall %}

### Usage Data

You can view the usage data of your application on your app's [Dashboard > LeanEngine > Statistics](https://console.leancloud.app/cloud.html?appid={{appid}}#/stat).

- **CPU**<br>The graph shows the application's CPU usage within a period. If the CPU usage approaches the limit, the response time of the application may be increased.
- **Memory**<br>The graph shows the application's memory usage within a period. If the memory usage approaches the limit, the thread of the application (like Node.js thread or Python thread) may be restarted due to OOM and the application will become unavailable in this period. If you see the line in the graph *frequently approaches the top and suddenly drops to the bottom*, it means that the thread has been restarted due to this reason.
- **Response time**<br>If the line approaches the top, it means that the CPU usage is approaching the limit.
- **Summary** <span class="text-muted">(in dropdown menu)</span><br>The total usage of all instances.
- **Details** <span class="text-muted">(in dropdown menu)</span><br>The usage of each instance.

## Group Management

You can create multiple groups of instances with different domains assigned to each of them. All of these groups can access the same data in LeanStorage. Common scenarios include:

- Having sites for users and administrators split into different projects with different domains assigned to each of them.
- Having edge systems separated from the main system so that the problems occurring on edge systems won't affect the main system.
- Having cloud functions and the main website written in different languages. For example, you can write cloud functions in Node.js and the main website in PHP.

Each application will have a primary group that handles cloud functions, hooks, and scheduled tasks. Other groups will be treated as secondary groups which **do not support defining cloud functions (including hooks and scheduled tasks) and can only offer web hosting with second-level domains or custom domains bound**. You can change the primary group at any time through the web console.

Each group has its own staging environment and second-level domain. Configurations like environment variables and Git repositories are also independent. You can deploy your project to a group without affecting the other ones. The methods introduced in [Managing Instances](#managing-instances) can be used to manage instances inside each group as well. If a group doesn't have any instances in it, it won't be able to handle requests. If a group has multiple instances, it will also gain the ability of load balancing and have its availability increased.

See [Pricing for Group Management](#pricing-for-group-management) for pricing information.

### Creating and Managing Groups

You can create, delete, or change primary groups by going to your app's [Dashboard > LeanEngine](https://console.leancloud.app/cloud.html?appid={{appid}}). On the top-left corner, you will see a group selector. Click on it, then click on **Group management**. You will see all the groups you have.

After selecting a group, you will be able to change its settings like Git repository, second-level domain, and environment variables. When a group is first created, it has no instances in it and you can only create a standard instance in its production environment. A staging environment will be provided with a free trial instance if you have at least one instance under the production environment, and it will be deleted if you delete all other instances in the group. A group can be deleted only if it has no instances in it.

If you are managing applications with multiple groups using the command-line interface (CLI), make sure the version of it is `0.7.1` or above. You can check the version of your CLI installed by running `$ lean --version`. See [Command-Line Interface Guide](leanengine_cli.html) for more instructions on CLI.

## Pricing

### Pricing for Instances

Trial instances are free.

Standard instances will be billed after midnight of each day. You can view the billing records of your account [here](bill.html#/bill/cost).

The price of your instances will be based on how many basic instances you are using. Assuming that the price of a basic instance is 1 USD per day (see our [website](https://leancloud.app/pricing/) for the actual price):

- If you have 4 instances with 1 basic instance for each of them, the price will be 4 × 1 USD = 4 USD per day.
- If you have 2 instances with 2 basic instances for each of them, the price will be (2 × 2) × 1 USD = 4 USD per day.

The number of basic instances for each day is the **maximum** number of basic instances you have had at the same time. For example:

- You created 4 instances with 1 basic instance for each of them;
- You realized that you need more instances, so you doubled the size of each of them and now you have 4 instances with 2 basic instances for each of them (8 basic instances in total);
- You realized that you have too many instances, so you removed a instance and now you have 3 instances with 2 basic instances for each of them (6 basic instances in total).

The number of basic instances for that day would be 8.

### Pricing for Group Management

If your application has only one group, you won't be charged for group management.

If your application has 2 or more groups, the price for group management will be the number of basic instances × the price of a basic instance × 20%.

Assuming that the price of a basic instance is 1 USD per day (see our [website](https://leancloud.app/pricing/) for the actual price):

- If you have 4 instances with 1 basic instance for each of them, the price will be 4 × 1 USD × 20% = 0.8 USD per day.
- If you have 2 instances with 2 basic instances for each of them, the price will be (2 × 2) × 1 USD × 20% = 0.8 USD per day.

If you don't want to be billed anymore, please make sure you only have one group in your application.

{% call docs.noteWrap() %}
Groups with no instances in it will also count into the total number of groups, so please make sure to delete the groups you are not using.
{% endcall %}

### Pricing for Extra Bandwidth

Every application has a daily free quota for LeanEngine bandwidth: `max(n, 1)` GB, where `n` is the number of standard instances of all groups of that application.

Extra bandwidth will be billed for 0.1 USD / GB.

LeanEngine is not suitable for scenarios such as distributing big files. We recommend developers to use [file service](leanstorage_guide-js.html#Files) instead.
