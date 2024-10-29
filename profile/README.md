<h1 align="center">The Apps Repository Architecture</h1>

<p align="middle">
  <img width="30%" src="https://github.com/user-attachments/assets/7a0af8d0-d275-4e7d-8740-6e6b0b13fc73" />
</p>

> [!NOTE]  
> This document is currently a work in progress

The apps repo architecture is an emerging standard for formatting and storing
your Kubernetes manifests. 

It provides:

* Battle tested best practices for Argo CD
* Templates for `ApplicationSets`
* Patterns that for facilitate self-service for developers and multi-cluster managment
* Preview environments like deployments for pull request branches
* Built-in security and autonomy for development teams

## Another standard?!

You may have heard about the [apps of apps](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/#app-of-apps-pattern)
pattern and be thinking: "Another standard, really?!". I understand your concern, but
stick around and we'll show you why this is worth your time!

First: The Apps in apps pattern mostly concerns using Argo apps to hold other Argo
apps. This is not mutually exclusive with the apps repo architecture and both can be
used together. 

The apps repo architecture covers the structure of the manifest repositories (apps
repositories), and how they can be used to solve a set of user needs. This is not
something that is widely standardised today and because of this everyone builds their
manifest repositories slightly differently. This is fine, but it means there is a
huge amount of redundant work out there as people are writing their own 
`ApplicationSets` from scratch.

## What problems are we solving for?

Platform teams exist to support product teams, so the needs of the product teams should
always come first. Modern autonomous product teams need enabling tools to support their
work, and as platform engineers we need to make sure the tools we provide are user
friendly.

Platform teams will generally want the deployment time to be as short as possible. This
is obviously so they can move things from dev to production as quickly as possible, but
also so they can create and test hypothesis with user testing. We must therefore
to make the feedback loop as tight as possible.

In order tighten the feedback loop even further we want to allow product teams to
deploy ephemeral "preview environments". This might be a pull request with a new 
feature that the developers want to test live before merging, allowing a more in-depth
review of changes like design and accessibility. We must therefore provide these with
as little friction as possible.

Kubernetes is a complex beast, and a product team will generally want to focus on
delivering features and fixes more than the infrastructure. So we don't want to burden
the teams with more configuration than necessary. The default behavior should be more
magic than not, giving a near frictionless experience to deploy an app. At the same
time we want to provide just enough configuration to the teams to be able to for
example disable auto-sync or enable istio for a given deployment.

Finally, giving the product teams self service and autonomy must not come at the cost
of security. We need to ensure that the principle of least privilige is maintained, and
that one team cannot affect another team's deployments in a multi-tenant cluster.
Security and autonomy must go hand in hand.

## The apps repository

<p align="middle">
  <img width="50%" src="https://github.com/user-attachments/assets/859bb108-4c8d-4905-a107-5923be301056" />
</p>

The Apps repo architecture mainly concerns itself with the structure of the manifest
repositories, known here as the apps repositories. Every apps repo will have the
following directories nested within each other:

* `env`: This is a directory that groups all the subsequent folders
* `cluster`: Inside the `env` directory we will have a set of directories which are
  named the same as the clusters which you would like to deploy to. In other words,
  these directory names match up with the cluster names in Argo CD
* `namespace`: Inside the `cluster` directory is where you will find the namespaces
  that are deployed to that cluster. New directories are picked up by Argo and
  automatically added as apps
* Finally the contents of `namespace` can be anything that Argo CD supports. For
  example this can be plain yaml files that you want to have synced, or a
  `Chart.yaml` that describes a helm chart with template files

In addition to this directory structure, optional directories for holding templates
and reusable elements are recommended to be put on the root. In the above example
you will see a kustomize `bases` directory added that is referenced by each 
kustomize file in the namespace directories to reduce duplication.

## Clear ownership

<p align="middle">
  <img width="50%" src="https://github.com/user-attachments/assets/88d33d95-d72b-4fb5-b761-60a41df62939" />
</p>

The apps repository structure enables a clear ownership of resources between
infrastructure and product teams. In this system the product teams do not burden
themselves with writing applications to set up deployment, all they do is create
directories and populate them with yaml files. This gives them full ownership
over their namespaces they have created by making directories, but not any other
namespaces or cluster scoped resources.

Infrastructure teams own cluster scoped resources and ApplicationSets that 
generate Argo Applications. The ApplicationSets monitor the product teams'
apps repositories and create namespaces when they detect new directories. This
means that the infrastructure teams also implicitly own the structure of the
namespaces that are created, whereas the product teams control the instantiation 
of them.

<p align="middle">
<a href="https://www.youtube.com/watch?v=8Zwftqf8g8w">
  <img width="50%" src="https://github.com/user-attachments/assets/5bda7118-97a0-44e3-9015-fa2bc9ca7378" />
</a>
</p>

Marco De Benedictis held a good talk on KubeCon Europe 2024 that recommends
exactly this structure. Namespaces should be assigned to tenants, but managed
by a platform team.

A challenge is automating the creation of ApplicationSets as new teams are
onboarded.

## What about security?

<p align="middle">
  <img width="40%" src="https://github.com/user-attachments/assets/f6dcfd2c-9c49-4a9d-8769-eda317837122" />
</p>

Security is obviously important, and since this architecture defines 
self-service provisioning of namespaces we need to make sure we're not allowing
teams to for example overwrite other teams' namespaces.

The way this is done is to assign each team one or several unique "prefixes".
Let's say your product team owns two products called Oslo and Bergen. These
two prefixes would be assigned to the product team by the infrastructure team
so that the product team could create as many namespaces they like that start
with `oslo-` or `bergen-`. This way the team can create namespaces for preview
environments like pull requests and long-lived namespaces like what is
currently on the main branch.

Your organization can define what follows the dash. A common pattern is to have
the branch name of the source code repo so that there is a 1:1 mapping between
an app and a branch, but this this is just a recommendation. The dash itself,
however, is required, as it helps prevent name colliions like the prefix 
`car` and `carrier` overlapping.

The way this is done in practice is by defining an Argo CD [AppProject](https://github.com/apps-repositories/example-infra-apps/blob/main/examples/2/appproject.yaml)
that requires target namespaces to be part of the prefix pattern.

## Show me an example!

Of course! Have a look at:

* [An example apps repo for a platform team](https://github.com/apps-repositories/example-infra-apps)
* [An example apps repo for a product team](https://github.com/apps-repositories/example-team-apps)
* [The Apps Repositories blog](https://skip.kartverket.no/blog/introducing-apps-repositories)
* [Argo CD on easymode - the presentation](slides.eliine.dev/argocd-easymode)
