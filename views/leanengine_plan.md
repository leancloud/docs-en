{% import "views/_helper.njk" as docs %}

# LeanEngine Plans and Prices

In this documentation, you will learn how you can choose the plan that best fits your needs, as well as how you can manage your instances and use the group management function.

LeanEngine is billed independently regardless of whether you are using the [Developer or Business Plan](https://leancloud.app/pricing/).
Upgrading to the Business Plan or degrading to the Developer Plan will not affect your LeanEngine cost.

## Standard Instances

You can purchase standard instances for projects that demand high availability. A standard instance runs 24 hours a day and will not hibernate even there are no requests. A staging environment will also be provided for testing. If you have 2 or more standard instances, you can even enable functions like load balancing and failover, which further increases the availability of your application.

**Staging Environment**<br>A trial instance will be provided in the staging environment so you can test your project before having it published.

**Load Balancing**<br>LeanEngine's gateway will distribute requests among all instances evenly. Therefore, you can scale up at any time by simply adding more instances.

**Failover**<br>Using 2 or more instances not only increases the computing power of your application but also enables failover. If one of these instances runs into errors, failover will be performed automatically so that further requests will be diverted to other instances.

{% call docs.noteWrap() %}
If you have only one instance and the server containing that instance runs into an error, we will still migrate your instance to another server within several minutes. However, your application will not be able to process requests while the migration is in process. {{ docs.alertInline("If your application demands high availability, we recommend that you enable 2 or more instances for it.") }}
{% endcall %}

**Rolling Deployment**<br>When deploying your project or performing other internal operations, the system will run the instances of the new version and those of the old version simultaneously for a while, then it will shut down the ones of the old version. This mechanism prevents the services provided by your app from being interrupted.

**Group Management**<br>You can create multiple groups of instances with different domains assigned to each of them. You can also bind a separate custom domain to each group. See [Group Management](#group-management).

## Trial Instances

Each application created on LeanCloud comes with a free trial instance with 0.5 CPU and 256 MB memory. You can use this instance for learning and testing purposes.

For every group with standard instances, LeanEngine also offers a trial instance in the staging environment for free.
Therefore, you can test your project before having it published.

A trial instance will stop responding to requests when deployment is in progress. It will also start [hibernating](#hibernating) if no request is made in the past 30 minutes (there will be a delay for processing requests when resuming), and can run at most 18 hours each day.

## Hibernating

Trial instances will **hibernate** under certain circumstances.

- A trial instance will start hibernating if no request is made in the past 30 minutes.
- A hibernated instance will resume when a request comes in. It may take 5 to 30 seconds (depending on the instance start-up time) for the instance to get back running and have the request processed. The requests made after that will be responded immediately.
- If a trial instance has run more than 18 hours in the past 24 hours, it will be forced to hibernate and incoming requests will receive `503` as the error code. You can view error logs in your app's [Dashboard > LeanEngine > Statistics](https://console.leancloud.app/cloud.html?appid={{appid}}#/stat).

{% call docs.noteWrap() %}
If you don't want your trial instance in the staging environment to hibernate, or need to simulate the production environment where there are multiple instances, you can purchase standard instances for the staging environment.
{% endcall %}

## Managing Instances

Four kinds of standard instances are available, with different RAM resources:

| Kind          | RAM     | CPU    |
| ------------- | ------- | ------ |
| standard-512  | 512 MB  | 1 Core |
| standard-1024 | 1024 MB | 1 Core |
| standard-2048 | 2048 MB | 1 Core |
| standard-4096 | 4096 MB | 1 Core |

To avoid the resources of instances being exhausted, it is recommended that:

- If daily average **RAM** usage exceeds **70%** of resources available (for example, 717 MB in a standard-1024 instance), upgrade to an instance with more RAM.
- If daily average **CPU** usage exceeds **30%** of resources available (for example, 30% CPU in a standard-1024 instance), add more instances.

See [Usage Data][#usage-data] to learn about how you can view the stress of your application.

You can have at most 12 instances for each application. Please contact us at leancloud-support@xd.com if you need more.

### Change Instance Quota

Go to your app's [Dashboard > LeanEngine > Resources](https://console.leancloud.app/cloud.html?appid={{appid}}#/leannode) and click on the **Change** button in the **Instance quota** section, then choose the quota you want.

If you are upgrading to the standard mode from the trial mode, you also need to choose the number of standard instances.

### Creating and Deleting Instances

Go to your app's [Dashboard > LeanEngine > Resources](https://console.leancloud.app/cloud.html?appid={{appid}}#/leannode) and click on the **Change** button in the **Production** or **Staging** environment section, then choose the number you want.

### Downgrade to Trial Mode

To downgrade to the trial mode, you need to change instance quota in all groups to **Trial mode**.
All standard instances will be deleted after that, and there will be a free trial instance with 0.5 CPU and 256 MB memory remaining in the last group.

### Multi-Instance Mode

Multi-instance mode means there are more than one instances running per environment per group.
It includes two conditions:

1. Persistent multi-instance mode, which occurs when you have purchased multiple instances for an environment of a group.
2. Temporary multi-instance mode, which occurs during rolling deployment.

Each instance has its own RAM and file system.
Therefore, data stored in the global variables or the file system of a single instance will not be accessible from other instances.
To share data among instances, you can use LeanCache.

### Usage Data

You can view the usage data of your application on your app's [Dashboard > LeanEngine > Statistics](https://console.leancloud.app/cloud.html?appid={{appid}}#/stat).

- **CPU**<br>The graph shows the application's CPU usage within a period. If the CPU usage approaches the limit, the response time of the application may be increased.
- **Memory**<br>The graph shows the application's memory usage within a period. If the memory usage approaches the limit, the thread of the application (like Node.js thread or Python thread) may be restarted due to OOM, and the application will become unavailable in this period. If you see the line in the graph *frequently approaches the top and suddenly drops to the bottom*, it means that the thread has been restarted due to this reason.
- **Response time**<br>If the line approaches the top, it means that the CPU usage is approaching the limit.
- **Details** <span class="text-muted">checkbox</span><br>The usage of each instance.

## Group Management

You can create multiple groups of instances with different domains assigned to each of them. All of these groups can access the same data in LeanStorage. Common scenarios include:

- Having sites for users and administrators split into different projects with different domains assigned to each of them.
- Having edge systems separated from the main system so that the problems occurring on edge systems won't affect the main system.
- Having cloud functions and the main website written in different languages. For example, you can write cloud functions in Node.js and the main website in PHP.

Every group can handle cloud functions, hooks, and scheduled tasks.
If a cloud function is defined on more than one group, the deployment will be interrupted by default, to avoid accidentally duplicated definitions.
Although you can force the deployment with the `--overwrite-functions` option of the command-line interface, it is recommended that you keep only one copy of the definition, to reduce the unnecessary mental and maintaining load. 


Each group has its own staging environment and can be bound to different custom domains. Configurations like environment variables and Git repositories are also independent. You can deploy your project to a group without affecting the other ones. The methods introduced in [Managing Instances](#managing-instances) can be used to manage instances inside each group as well. If a group doesn't have any instances in it, it won't be able to handle requests. If a group has multiple instances, it will also gain the ability of load balancing and have its availability increased.

See [Pricing for Group Management](#pricing-for-group-management) for pricing information.

### Creating and Managing Groups

You can create, delete, or change primary groups by going to your app's [Dashboard > LeanEngine](https://console.leancloud.app/cloud.html?appid={{appid}}). On the top-left corner, you will see a group selector. Click on it, then click on **Group management**. You will see all the groups you have.

After selecting a group, you will be able to change its settings like Git repository and environment variables. When a group is first created, it has no instances in it and you can only create a standard instance in its production environment. A staging environment will be provided with a free trial instance if you have at least one instance under the production environment.

## Pricing

### Pricing for Instances

Trial instances are free.

Standard instances will be billed daily.
You can view the billing records of your account [here](bill.html#/bill/cost).

The cost is calculated based on the **maximum usage**, and it will be deducted after the midnight of each day.

Suppose in one day (from 0:00 to 24:00):

- There are originally four `standard-512` standard instances.
- You realized that you need more instances, so you doubled the size of each of them (four `standard-1024` instances).
- You realized that you have too many instances, so you removed two (two `standard-1024` instances).

The cost on that day is calculated against four `standard-1024` instances.


### Pricing for Extra Bandwidth

Every application has a daily free quota for LeanEngine bandwidth: `max(n, 1)` GB, where `n` is the number of standard instances of all groups of that application.

Extra bandwidth will be billed for 0.1 USD / GB.

LeanEngine is not suitable for scenarios such as distributing big files. We recommend developers to use [file service](leanstorage_guide-js.html#Files) instead.
