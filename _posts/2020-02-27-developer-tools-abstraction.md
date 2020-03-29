---
layout: post
title:  "Good Developer Tools Support Abstraction"
date:   2020-02-27
---

When I stopped writing CloudFormation and picked up [CDK][cdk], I expected a
_slightly_ better experience. The tooling and type checking from TypeScript
seemed useful, but I wondered if it was worth the extra layer of abstraction.
After four months of using CDK, the productivity leap is surprising. Technical
solutions that used to be expensive and daunting are now fun to build.

I often hear developers ask, "Why would I want to move from a
declarative language like YAML to an imperative one like JavaScript?"
Developers that [gave it five minutes][give-it-5] though, never looked
back.

For me, this is a reminder that supporting abstraction is critical for humane
developer tools. The value from a DSL that embraces [The Rule of Least
Power][least-power] is inconsequential if it stifles abstraction.

I've seen inhumane tools for working with cloud infrastructure. They
veer into some common ditches.

### Overreliance on templates

When programming becomes repetitive and developers complain that tasks take
too long, templates are the first tools that emerge. But because templates
don't create abstraction, they leave a wake of technical debt and provide
little value over copy-and-pasting.

### Building Workflow GUIs

At first glance, a GUI that helps developers with a common workflow seems
useful. Unfortunately, filling out a form is hard to repeat, test, review,
and deploy to multiple environments. GUIs are great for providing information
but are an abstraction bottleneck when receiving information.

### Abstraction spikes

Rather than craft the fundamental elements needed to solve a problem, it can
be faster to create a one-off solution. Time passes, the problem shifts,
and developers are stuck waiting for another one-off solution.

For instance, [SAM templates][sam-templates] lower the complexity of using
CloudFormation but only address a narrow problem space. Developers must eject
from the paradigm when their needs expand or the landscape changes. Golang
famously provides two data structures that implement generics as a special
case (map and slice), but doesn't let developers do the same.

---

These three pitfalls share something important: they demo well. However
they all thwart abstraction.

## Humane Developer Tools

Humane developer tools are composed, composable, and usable.

### Composed

The alternative to "abstraction spikes" are solutions built on smaller
primitives. And, perhaps counterintuitively, it's important to expose those
primitives to developers.

A good example is the evolution of `async/await` in JavaScript. Arguably,
being able to write `await fetch()` is the most important feature, but
developers can write their own awaitable functions with the `Promise`
primitive. This allows developers to build with a consistent approach for all
asynchronous code they use.

### Composable

Using a developer tool should not prohibit the use of others. You can only
use a single application template to setup your project, but you can combine
many libraries. A good developer tool should know where its value lies and
where to stop.

### Usable

A product is not usable in a vacuum. User abilities and context determine a
tool's usability.

And developers tend to have specific needs that the end users they serve
don't. Developers need to reliably automate and repeat tasks. They need
artifacts for peer review and to facilitate rollbacks. Likely, they have
skills that a tool could exploit instead of introducing new concepts.

## CDK Affords Abstraction

Let's look at a simplified example of a cluster that runs two related
services.

```yaml
Type: AWS::AutoScaling::AutoScalingGroup
Properties:
  DesiredCapacity: 3

Type: AWS::ECS::Service
Properties:
  DesiredCount: 1

Type: AWS::ECS::Service
Properties:
  DesiredCount: 2
```

When we increase the `DesiredCount` of the first service, we have to
remember to change the `DesiredCapacity` on the autoscaling group. This is a
hidden concept that would be difficult to detect in a large project.

Terraform provides more expressive power than CloudFormation, so we could solve
that problem with HCL.

```hcl
resource "aws_ecs_service" "service_1" {
  desired_count = 1
}

resource "aws_ecs_service" "service_2" {
  desired_count = 3
}

resource "aws_autoscaling_group" "asg" {
  desired_capacity = "${aws_ecs_service.service_1.desired_count + aws_ecs_service.service_2.desired_count}"
}
```

That reifies our hidden concept. So what does CDK bring to the table?

We could take the same approach:

```ts
const service1 = Service(this, 'Service1', {
  desiredCount: 1,
});

const service2 = Service(this, 'Service2', {
  desiredCount: 2,
});

const asg = AutoScalingGroup(this, 'Asg', {
  desiredCapacity: service1.desiredCount + service2.desiredCount,
});
```

But with CDK we can go further[^1]. We can use object-oriented programming to express
intent and hide information.

```ts
const service1 = Service(this, 'Service1', {
  desiredCount: 1,
});

const service2 = Service(this, 'Service2', {
  desiredCount: 2,
});

const asg = MyCluster(this, 'Cluster', {
  services: [service1, service2]
});
```

The caller no longer knows what the cluster does with the services. It may just
add their `desiredCounts` together or it may use some other information to
decide how many machines it needs.

## Going Big

Code reuse in the small is easy to pull off. Could we realize the promise of
integrating large, reusable modules with a tool like CDK? I'm still finding
out but I'm optimistic.

Consider [this heroic project][ecs-airflow] which creates hundreds of AWS
resources to deploy Airflow with CloudFormation templates. Using this
this project involves a mix of manual steps and ad hoc scripts. If you need
to maintain this system over time, you're likely to modify the CloudFormation
templates, breaking the abstraction. Every new extension point for this
project requires a new top-level parameter for users to consider.

Contrast that with a hypothetical solution using CDK. All of the messy
deployment and installation concerns are dealt with by CDK, npm, and other
common developer tools, freeing a library author to focus on the
implementation and user interface of the solution. A CDK user wouldn't even
need to look at README to get started. They would just type and learn
how to setup Airflow:

<img width="827" src="/images/developer-tools-abstraction/ts-guiding.png" />

Maybe all you want is to get started with some sane defaults.

```ts
export class AirflowStack extends core.Stack {
  constructor(scope: core.App, id: string, props?: AirflowStackProps) {
    super(scope, id, props);

    new airflow.System(this, 'Airflow');
  }
}
```

Or maybe you need robust replication on the Airflow database and want to use MySQL
instead of Postgres. If you can adhere to an the interface for `Storage`, you
could meet requirements that the library author never intended.

```ts
export class AirflowStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const storage = new mystoragelib.AuroraMySQLStorage(this, 'DB', {
      replication: new airflow.CrossRegionReplication({
        regions: ['us-west-2', 'us-east-1'],
        replicasPerRegion: 2,
      }),
    });

    new airflow.System(this, 'Airflow', {
      storage,
    });
  }
}
```

Of course object-oriented programming is a demanding discipline and
abstraction can hinder understanding. With more power, we risk creating a
confusing system that is hard to adapt. But when we're building something
complex, it is the risk we have to take. Our only choice is to build
abstractions well and embrace [Software's Primary Technical
Imperative][code-complete].

---

#### Footnotes

[^1]:
    For version 0.12 Terraform discusses [module composition][tf-module-composition]
    and I'm confident an experienced user could
    create an example roughly equivalent to the object-oriented approach shown
    for CDK. However I've never seen a Terraform example like this in the wild and
    the larger concern is the patterns a DSL like HCL _affords_. The key-value
    slant of HCL and its readme make its original intentions clear:

    > Full programming languages such as Ruby enable complex behavior a
    > configuration language shouldn't usually allow...
    >
    > -- [HCL readme][hcl-readme]

    And without user-defined functions, Terraform errs on the side of simplicity:

    > The Terraform language does not support user-defined functions, and so only
    > the functions built in to the language are available for use.
    >
    > -- [Terraform functions documentation][tf-fn-docs]

    Like YAML, HCL is easier to analyze than a general-purpose programming
    language making it a good compilation target for a tool like CDK.

[hcl-readme]: https://github.com/hashicorp/hcl
[give-it-5]: https://signalvnoise.com/posts/3124-give-it-five-minutes
[least-power]: https://www.w3.org/2001/tag/doc/leastPower.html
[code-complete]: https://www.microsoftpressstore.com/articles/article.aspx?p=2222451&seqNum=2
[tf-module-composition]: https://www.terraform.io/docs/modules/composition.html
[tf-fn-docs]: https://www.terraform.io/docs/configuration/functions.html
[cdk]: https://github.com/aws/aws-cdk
[sam-templates]: https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification.html
[ecs-airflow]: https://github.com/FreckleIOT/ecs-airflow
