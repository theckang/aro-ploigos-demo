# Overview

In this repo, you will deploy the Ploigos Software Factory on Azure Red Hat OpenShift (ARO) and trigger a Ploigos software pipeline using GitHub Actions.

## Prerequisites

* Azure Red Hat OpenShift 4 cluster
* Admin access to OpenShift
* [OpenShift CLI](https://docs.openshift.com/container-platform/4.6/cli_reference/openshift_cli/getting-started-cli.html)

## Prep

Login to the cluster using `oc login` and admin credentials.

## Setup

1. Fork this [repo](https://github.com/theckang/reference-quarkus-mvn_jenkins_workflow-standard).
2. Follow these [instructions](https://github.com/ploigos/ploigos-software-factory-operator#tssc-demo-with-a-reference-app) to deploy the platform for the Ploigos Software Factory.  Stop after Step #11.  You will deploy your own custom pipeline instance.
3. Modify the `config/pipeline.yaml` resource.  Replace the `sourceUrl` with your repo fork.
4. Deploy the pipeline.

```bash
oc create -f config/pipeline.yaml
```

5. Navigate to Jenkins in your browser.  Login with admin credentials.

```bash
echo $(oc get route jenkins --template='http://{{.spec.host}}')
```

6. In Jenkins, navigate to 'Manage Jenkins' -> 'Configure System' -> 'Gitea Servers'.  You should see a checkbox for 'Manage hooks'.  Select the checkbox and make sure to hit `Save`.

![Manage Hook](images/manage_hook.png)

7. In Jenkins, navigate to the 'Platform Gitea Org'

```bash
echo $(oc get route jenkins --template='http://{{.spec.host}}/job/platform')
```

8. On the left, hit 'Scan Gitea Organization Now'.  This will configure a webhook in Gitea for you.

9. The repo has a GitHub action that will mirror your fork repo to the Gitea instance.  You can view the example [here](https://github.com/theckang/reference-quarkus-mvn_jenkins_workflow-standard/blob/main/.github/workflows/mirroring.yaml).  You need to add the Gitea admin's username and password as a secret to your repo.

In GitHub, navigate to your forked repo.  Go to 'Settings' -> 'Secrets'.  Create two repository secrets `GIT_USERNAME` and `GIT_PASSWORD`.  Execute these commands to get the username and password of your Gitea instance and enter these values.

```bash
echo $(oc get secret gitea-admin-credentials -o jsonpath="{.data.username}") | base64 --decode && echo
echo $(oc get secret gitea-admin-credentials -o jsonpath="{.data.password}") | base64 --decode && echo
```

10. Merge a branch to your fork repo to start the action.  This will add a new fruit 'Kiwi' to the reference application.

```bash
git clone <your-fork>
cd <your-fork>
git merge origin/feature/kiwi
git push
```

11. In GitHub, navigate to your fork.  Go to 'Actions'.  You should see the action:

![GitHub Action](images/github_action.png)

12. In Jenkins, navigate to the 'Platform Gitea Org'

```bash
echo $(oc get route jenkins --template='http://{{.spec.host}}/job/platform/job/reference-quarkus-mvn_jenkins_workflow-standard/')
```

13. 

## Troubleshooting

1.  If you push a commit to the main branch but don't see the Jenkins build trigger, make sure the webhook is configured in Gitea.

Navigate to the Gitea repo in your browser

```bash
echo $(oc get route gitea --template='http://{{.spec.host}}'/platform/reference-quarkus-mvn_jenkins_workflow-standard-github)
```

Login with admin credentials

```bash
echo $(oc get secret gitea-admin-credentials -o jsonpath="{.data.username}") | base64 --decode && echo
echo $(oc get secret gitea-admin-credentials -o jsonpath="{.data.password}") | base64 --decode && echo
```

Navigate to 'Settings' -> 'Webhooks' and you should see:

![Gitea Webhook](images/gitea_webhook.png)

If you don't see this webhook, make sure to scan the Gitea org in Step #8 above.

## Resources

* [Ploigos Operator](https://github.com/ploigos/ploigos-software-factory-operator)
* [Reference Application](http://gitea.tssc.rht-set.com/akrohg/reference-quarkus-mvn_jenkins_workflow-standard.git)
* [Setting up webhook in Gitea/Jekins](https://gcube.wiki.gcube-system.org/gcube/Gitea/Jenkins:_Setting_up_Webhooks)
