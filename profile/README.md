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

You may have heard about the apps in apps pattern and be thinking: "Another
standard, really?!". I understand your concern, but stick around and we'll show
you why this is just so cool.

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

## Show me an example!

Of course! Have a look at:

* [An example apps repo for a platform team](https://github.com/apps-repositories/example-infra-apps)
* [An example apps repo for a product team](https://github.com/apps-repositories/example-team-apps)
* [The Apps Repositories blog](https://skip.kartverket.no/blog/introducing-apps-repositories)
* [Argo CD on easymode - the presentation](slides.eliine.dev/argocd-easymode)
