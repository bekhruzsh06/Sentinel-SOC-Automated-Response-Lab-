# Phase 3: SOAR Automation (Slack Alerts)

[← Previous: Log Analysis & KQL](02-log-analysis-kql.md) | [Back to README](../README.md) | [Next: Attack Simulation →](04-attack-simulation.md)

---

### Step 1: Prepare Your Slack Workspace

1. Open Slack (or create a free workspace). This lab uses a workspace called **Soc Lab** and the channel `#all-soc-lab`.
2. Keep Slack open in the background so you can authorise the connection in the next steps.

![Create Slack workspace](../images/03-soar/01-slack-create-workspace.png)

![Slack channel](../images/03-soar/02-slack-channel.png)


### Step 2: Create the Logic App (Sentinel Playbook)

1. In the Azure Portal search bar, type **Microsoft Sentinel** and select your `SOC-Lab-LAW` workspace.
2. In the left-hand menu, scroll down to **Configuration** and click **Automation**. It also opens in the Defender portal.

![Sentinel Automation menu](../images/03-soar/03-sentinel-automation-menu.png)

![Automation page](../images/03-soar/04-automation-page.png)

3. At the top, click **+ Create** and select **Playbook with incident trigger**.
4. **Basics tab:**
    - **Resource group:** Select your existing `SOC-Lab-RG` resource group.
    - **Playbook name:** `Post-Alert-to-Slack`
    - Click **Review + create**, then click **Create**.

![Playbook basics](../images/03-soar/05-playbook-basics.png)

5. Once deployment is complete, click **Go to Playbook**. This opens the Logic App Designer.


### Step 3: Build the Logic App Workflow

1. In the Logic App Designer you will see the trigger block **Microsoft Sentinel incident**. Click the **+** under it to add a new step.

![Logic App trigger](../images/03-soar/06-logic-app-trigger.png)

2. In the search box, type **Slack** and select the Slack connector.
3. Choose the action **Post message (V2)**.

![Slack actions](../images/03-soar/07-slack-actions.png)

![Post message (V2)](../images/03-soar/08-post-message-v2.png)

4. **Connection setup:**
    - Name the connection (e.g. `SlackSOC`).
    - Click **Sign in**. A pop-up will ask you to authorise Azure to access your Slack workspace. Allow it.

5. **Configure the message:**
    - **Channel Name:** Select your `#all-soc-lab` channel from the dropdown.
    - **Message Text:** Click inside the box and use the **Dynamic content** menu to build a message like this (Slack uses single `*` for bold):

```plaintext
‼️ **New Sentinel Alert Detected** ‼️
**Title**: [Incident Title]
**Severity**: [Incident Severity]
**Description**: [Incident Description]
**URL**: [Incident URL]
```

![Slack message configuration](../images/03-soar/09-slack-message-config.png)

6. Click **Save**.

![Logic App saved](../images/03-soar/10-logic-app-saved.png)


### Step 4: Link the Playbook to the Analytics Rule

For the playbook to run automatically, we must attach it to the brute-force rule we created in Phase 2.

1. Go back to the **Automation** page (in the Azure Portal or the Defender portal).
2. Click **+ Create** > **Automation rule**.

![Create automation rule](../images/03-soar/11-create-automation-rule.png)

3. **Name:** `Trigger-Slack-Playbook`
4. **Trigger:** When incident is created.
5. **Conditions:** If **Analytic rule name** contains `[RDP Brute Force Detection] High Volume Failed Logins`.
6. **Actions:** Action type **Run playbook**, then select the `Post-Alert-to-Slack` playbook from the dropdown.

![Playbook permission error](../images/03-soar/12-playbook-permission-error.png)

#### Troubleshooting: "No Microsoft Sentinel permissions"

At this point the playbook was greyed out with the error **No Microsoft Sentinel permissions**. Sentinel needs explicit permission to run playbooks in a resource group. To fix it:

1. In the Microsoft Sentinel left-hand menu, scroll to the bottom and click **Settings**.

![Sentinel settings](../images/03-soar/13-sentinel-settings.png)

2. Select the **Settings** tab at the top of the main window.
3. Scroll down to **Playbook permissions** and click **Configure permissions**.

![Configure playbook permissions](../images/03-soar/14-configure-permissions.png)

4. In the side pane, check your resource group (`SOC-Lab-RG`) and click **Apply**.

![Grant permissions to SOC-Lab-RG](../images/03-soar/15-grant-permissions.png)

5. Return to the **Automation** menu and create the rule again. The `Post-Alert-to-Slack` playbook can now be selected.

![Automation rule complete](../images/03-soar/16-automation-rule-complete.png)

Click **Apply** to save the rule.

The SOAR automation is now live. The next time an IP fails 10 logins, Sentinel will create an incident and push the details to the Slack channel.

---

[← Previous: Log Analysis & KQL](02-log-analysis-kql.md) | [Back to README](../README.md) | [Next: Attack Simulation →](04-attack-simulation.md)
