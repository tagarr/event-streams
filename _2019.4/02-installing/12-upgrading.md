---
title: "Upgrading to 2019.4.1"
excerpt: "Upgrade your installation to the latest version."
categories: installing
slug: upgrading
toc: true
---

Upgrade your installation to the latest version of {{site.data.reuse.long_name}} as follows.

You can upgrade to {{site.data.reuse.short_name}} version 2019.4.1 from version 2019.2.1. If you have an earlier version, you must first upgrade your {{site.data.reuse.short_name}} version [to 2019.2.1](../../2019.2.1/installing/upgrading/), before following these steps to upgrade to version 2019.4.1.

**Important:** {{site.data.reuse.short_name}} only supports upgrading to a newer chart version. Do not select an earlier chart version when upgrading. If you want to revert to an earlier version of {{site.data.reuse.short_name}}, see the instructions for [rolling back](../rolling-back/).

## Prerequisites

- Ensure you have {{site.data.reuse.icp}} version 3.2.1. If you are using the {{site.data.reuse.openshift_short}}, ensure you have version 3.11 or later. See the [prerequisites](../prerequisites/#container-environment) for supported container environments.
- The minimum resource requirements have changed. Ensure you have the required [resources](../prerequisites/#helm-resource-requirements) available.
- If you are upgrading {{site.data.reuse.long_name}} (not {{site.data.reuse.ce_short}}), download the package for the version you want to upgrade to, and make it [available](../installing/#download-the-archive) to your {{site.data.reuse.icp}} instance.
- If you have [encryption between pods](../planning/#securing-communication-between-pods) enabled, then ensure you disable it before starting the upgrade. After the upgrade completes successfully, you can [enable](../../security/encrypting-data/#enabling-encryption-between-pods) the encryption again.

## Upgrading on {{site.data.reuse.icp}}

If you are using {{site.data.reuse.icp}}, you can  upgrade by using the UI or the command line.

### Using the UI

1. {{site.data.reuse.icp_ui_login321}}
4. Click **Workloads > Helm Releases** from the navigation menu.
5. Locate the release name of your installation in the **Name** column, and click ![More options icon](../../images/more_options.png "Three vertical dots for the more options icon at end of each row."){:height="30px" width="15px"} **More options > Upgrade** in the corresponding row.
6. Select the chart version to upgrade to from the **Version** drop-down list.
7. Ensure you have **Using previous configured values** set to **Reuse Values**.\\
   **Note:** Do not change any of the settings in the **Parameters** section. You can modify configuration settings after upgrade, for example, [enable encryption](#enable-encryption-between-pods) between pods.
8. Click **Upgrade**.\\
   The upgrade process begins and restarts your pods. During the process, the UI shows pods as unavailable and restarting.
9. After the upgrade completes, [perform the post-upgrade](#post-upgrade-tasks) tasks.


### Using the CLI

1. Ensure you have the latest Helm chart version available on your local file system.\\
   - You can [retrieve](../../administering/helm-upgrade-command/) the charts from the UI.
   - Alternatively, if you downloaded the archive from IBM Passport Advantage, the chart file is included in the archive. Extract the PPA archive, and locate the chart file in the `/charts` directory, for example: `ibm-eventstreams-prod-1.4.0.tgz`
2. {{site.data.reuse.icp_cli_login321}}

   **Important:** You must have the Team Administrator or Cluster Administrator role to upgrade the chart.

3. Run the helm upgrade command as follows, referencing the Helm chart you want to upgrade to:\\
   `helm upgrade <release-name> <latest-chart-version>`

   For example, to upgrade the {{site.data.reuse.ce_short}}:\\
   `helm upgrade eventstreams1 /Users/admin/upgrade/ibm-eventstreams-dev-1.4.0.tgz`

   For example, to upgrade by using a chart downloaded in the PPA archive:\\
   `helm upgrade eventstreams1 /Users/admin/upgrade/ibm-eventstreams-prod-1.4.0.tgz`

   **Note:** Do not set any parameter value during the upgrade, for example, `helm upgrade --set <parameter>=<value> <release_name> <charts.tgz> --tls`. You can modify configuration settings after upgrade, for example, [enable encryption](#enable-encryption-between-pods) between pods.

   The upgrade process begins and restarts your pods. During the process, the UI shows pods as unavailable and restarting.

4. After the upgrade completes, [perform the post-upgrade](#post-upgrade-tasks) tasks.

## Upgrading on {{site.data.reuse.openshift_short}}

If you are using the {{site.data.reuse.openshift_short}}, you can only  upgrade by using the command line.

1. Ensure you have the latest Helm chart version available on your local file system.

   **Note:** The {{site.data.reuse.short_name}} chart name no longer includes `rhel` in the name. For example, the {{site.data.reuse.ce_short}} chart name is now `ibm-eventstreams-dev` (not `ibm-eventstreams-rhel-dev`), and similarly the paid-for version chart name is now  `ibm-eventstreams-prod` (`ibm-eventstreams-rhel-prod`).

   - You can [retrieve](../../administering/helm-upgrade-command/) the charts from the UI.
   - Alternatively, if you downloaded the archive from IBM Passport Advantage, the chart file is included in the archive. Extract the PPA archive, and locate the chart file in the `/charts` directory, for example: `ibm-eventstreams-prod-1.4.0.tgz`

2. {{site.data.reuse.icp_cli_login321}}

   **Important:** You must have the Team Administrator or Cluster Administrator role to upgrade the chart.

3. Delete the Kafka StatefulSet as follows:

   `kubectl delete statefulset <release-name>-ibm-es-kafka-sts --cascade=false -n <namespace>`

   The `cascade=false` flag deletes the StatefulSet, but leaves the Kafka pods running.

4. Run the helm upgrade command as follows, referencing the Helm chart you want to upgrade to:\\
   `helm upgrade <release-name> <latest-chart-version>`

   For example, to upgrade the {{site.data.reuse.ce_short}}:\\
   `helm upgrade eventstreams1 /Users/admin/upgrade/ibm-eventstreams-dev-1.4.0.tgz`

   For example, to upgrade by using a chart downloaded in the PPA archive:\\
   `helm upgrade eventstreams1 /Users/admin/upgrade/ibm-eventstreams-prod-1.4.0.tgz`

   **Note:** Do not set any parameter value during the upgrade, for example, `helm upgrade --set <parameter>=<value> <release_name> <charts.tgz> --tls`. You can modify configuration settings after upgrade, for example, [enable encryption](#enable-encryption-between-pods) between pods.

   The upgrade process begins and restarts your pods. During the process, the UI shows pods as unavailable and restarting.

5. After the upgrade completes, [perform the post-upgrade](#post-upgrade-tasks) tasks.


## Post-upgrade tasks

Additional steps required after upgrading are described in the following sections.

### Set up access management

If you have {{site.data.reuse.icp}} teams set up for [access management](../../security/managing-access/#assigning-access-to-users), you must associate the teams again with your {{site.data.reuse.long_name}} instance after successfully completing the upgrade.

To use your upgraded {{site.data.reuse.short_name}} instance with existing {{site.data.reuse.icp}} teams, re-apply the security resources to any teams you have defined as follows:

1. Check the teams you use:\\
   1. {{site.data.reuse.icp_ui_login321}}
   3. From the navigation menu, click **Manage > Identity & Access > Teams**. Look for the teams you use with your {{site.data.reuse.short_name}} instance.
2. Ensure you have [installed](../../installing/post-installation/#installing-the-command-line-interface-cli) the latest version of the {{site.data.reuse.short_name}} CLI.
3. Run the following command for each team that references your instance of {{site.data.reuse.short_name}}:\\
  `cloudctl es iam-add-release-to-team --namespace <namespace> --helm-release <helm-release> --team <team-name>`

<!--Without this rerun of the command, customers will find that creating service IDs in the UI will fail unless you're running as cluster administrator.-->

### Update browser certificates

   If you trusted certificates in your browser for using the {{site.data.reuse.short_name}} UI, you might not be able to access the UI after upgrading.

   To resolve this issue, you must delete previous certificates and trust new ones. Check the browser help for instructions, the process for deleting and accepting certificates varies depending on the type of browser you have.

### Update cluster certificates

Upgrading your {{site.data.reuse.short_name}} version also updates your internal certificates automatically. You can manually change your internal certificates later. Internal certificates can only be self-generated certificates, they cannot be provided.

You can also [update](../../security/updating-certificates) your external certificates manually after an upgrade, for example, if you want to change from using self-generated certificates to provided certificates.

**Note:** If you update your cluster certificates you might also need to update the certificates of any connecting [clients](../../getting-started/client/#securing-the-connection).

### Enable encryption between pods

As mentioned in the [prerequisites](#prerequisites), encryption between pods must be disabled before upgrading {{site.data.reuse.short_name}}.

If you had pod to pod encryption enabled before upgrading, [enable](../../security/encrypting-data/#enabling-encryption-between-pods) it again.

### Switch to routes

If you are using the {{site.data.reuse.openshift_short}} and have upgraded to {{site.data.reuse.short_name}} version 2019.4.1, you can switch to using OpenShift routes. Routes allow access to your cluster through a host name instead of the host IP address and node port combination.

{{site.data.reuse.openshift_short}} [routes](https://docs.openshift.com/container-platform/3.11/architecture/networking/routes.html){:target="_blank"} are the standard way of accessing applications inside an OpenShift cluster, and provide the benefit of being pre-determined and understandable.

**Tip:** If you installed {{site.data.reuse.short_name}} version 2019.4.1 without upgrading from a previous version, you will already be using routes.

**Important:** Switching to routes will alter the {{site.data.reuse.short_name}} certificates, and will therefore require redistributing the certificates to all your clients and updating their connection settings to use the new bootstrap route for their bootstrap server. This means the switch over will cause your clients to disconnect, resulting in downtime.

To switch to routes:

1. {{site.data.reuse.icp_cli_login321}}

   **Important:** You must have the Team Administrator or Cluster Administrator role to upgrade the chart.

2. Ensure you have routes available in your container platform by running the following command:

   `kubectl api-resources | grep routes`

   If the command returns `routes route.openshift.io true Route`, then you have OpenShift routes available.
3. Ensure you have the latest Helm chart version [available](../../administering/helm-upgrade-command/) on your local file system.
4. Run the following command to configure your {{site.data.reuse.short_name}} installation to use routes:

   `helm upgrade --reuse-values --set global.security.externalCertificateLabel=upgraded --set proxy.upgradeToRoutes=true <release-name> <latest-chart-version>`
5. After the upgrade completes successfully, and all pods are ready, run the following command to update your installation with the new routes.

   `NAMESPACE=<namespace>; kubectl patch $(kubectl get cm -n $NAMESPACE -l component=proxy -o name) -n $NAMESPACE -p "{\"data\":{\"externalListeners\":\"\",\"revision\":\"$(date +%s999999999)\"}}";`

6. The {{site.data.reuse.short_name}} UI address changes to a route, which can be retrieved by using the following command:

   `echo https://$(kubectl get route -n <namespace> <release-name>-ibm-es-ui-route -o 'jsonpath={.spec.host}')`

7. The {{site.data.reuse.short_name}} schema registry uses the REST API. Configuring {{site.data.reuse.short_name}} to use routes changes this endpoint value to a route, so ensure you update the clients that use schemas to use the route name. To retrieve the REST route name, run the following command:

   `echo https://$(kubectl get route -n <namespace> <releasename>-ibm-es-rest-route -o 'jsonpath={.spec.host}')`

8. Update the [connection settings](../../getting-started/client/#securing-the-connection) for your clients that interact with {{site.data.reuse.short_name}}.
   This will require your clients to disconnect, and connect again after the changes are made, resulting in downtime.
