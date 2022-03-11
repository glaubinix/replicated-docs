# Updating an application

## Using the admin console

The simplest way to update an application is through the Version History tab on the Replicated admin console.
This method works for both online and air gapped installations.

### Checking for updates
The admin console automatically checks for updates once every 4 hours.
To manually check for a more recent version, click **Check for updates** in the admin console.
In air gapped instances this button will be replaced with **Upload a new version**.
Air gapped instances cannot check for updates automatically.
When an update has been downloaded (for online) or uploaded (for air gap), a new upstream version will show in the list of released versions.

[![New Version Available](/images/new-version-available.png)](/images/new-version-available.png)

## Comparing changes between releases
When there are multiple versions of a application, you can compare the changes between them by clicking **Diff releases** in the right corner.

[![Diff Releases](/images/diff-releases.png)](/images/diff-releases.png)

Changes can be reviewed between any arbitrary release by clicking the icon in the header of the release column. Select the two versions to compare, and click **Diff releases** to show the relative changes between the two releases.

[![New Changes](/images/new-changes.png)](/images/new-changes.png)


### Preflight checks
Click the **Preflight results** link to run the preflight checks defined by the application vendor.
Based on the outcome of each preflight check, you can decide whether or not to perform the upgrade by clicking **Continue**.

[![Preflight Checks](/images/preflight-checks.png)](/images/preflight-checks.png)

Warnings do not preclude the upgrade to a new version. The installer may elect to ignore warnings and proceed with the upgrade. 

Preflight check failures can be ignored unless they are `strict`. If present, `strict` preflight checks that have a `fail` outcome prevent the installer from deploying the release. For more information about resolving strict preflight checks, see [Resolving Strict Preflight Checks](../enterprise/installing-existing-cluster-online#resolve-strict-preflight-checks).

### Updating
An update is performed by clicking **Continue** on the preflight checks page, or by clicking **Deploy** on the Version History tab.
At this point, the current cluster will be updated to the new version of the application and the Deployed status will be set on that version.

## Using CLI

The kots CLI can be used to install and deploy updates for both online and air gapped instances as well.

### Online installations

In order to download updates from the internet, the following command can be used:

```bash
kubectl kots upstream upgrade <app slug> -n <admin console namespace>
```

Adding the `--deploy` flag will also automatically deploy the latest version.

### Existing cluster air gapped installations

In order to install an update from an air gap file, the following command can be used:

```bash
kubectl kots upstream upgrade <app slug> \
  --airgap-bundle new-app-release.airgap \
  --kotsadm-namespace <registry namespace> \
  --kotsadm-registry <registry host> \
  --registry-username <username> \
  --registry-password <password> \
  -n <admin console namespace>
```

Adding the `--deploy` flag will also automatically deploy this version.

### Embedded cluster air gapped installations

> Introduced in the Replicated app manager v1.34.0

In order to install an update from an air gap file, the following command can be used:

```bash
kubectl kots upstream upgrade <app slug> --airgap-bundle new-app-release.airgap -n <admin console namespace>
```

Adding the `--deploy` flag will also automatically deploy this version.
