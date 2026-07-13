**WORK INSTRUCTION \| AZURE GOVERNMENT**

Controlled copy when viewed in the approved document repository \| Page 1

**Restore an Azure Virtual Machine from Backup

Azure Government \| Recovery Services Vault \| Portal Procedure

| \*\*Document ID\*\* | WI-AZGOV-VM-RESTORE | \*\*Version\*\* | \\1.0 |
| :----- | :----- | :----- | :----- |
| \*\*Owner\*\* | \\\[Cloud Operations\\\] | \*\*Effective Date\*\* | \\\[YYYY-MM-DD\\\] |
| \*\*Approved By\*\* | \\\[Name / Role\\\] | \*\*Review Cycle\*\* | Annual |
| \*\*Classification\*\* | \\\[Organization-defined\\\] | \*\*Environment\*\* | Azure Government |
| \*\*Related Ticket\*\* | \\\[Incident / Change #\\\] | \*\*Last Validated\*\* | July 2026 |

\1. Purpose**

This work instruction provides a controlled, step-by-step procedure for restoring an Azure virtual machine (VM) from an Azure Backup recovery point stored in a Recovery Services vault in Azure Government. It covers recovery planning, portal execution, validation, and cleanup.

**\2. Scope**

Use this procedure for Windows or Linux Azure Resource Manager VMs protected by Azure Backup. The primary procedure restores the VM by creating a new VM from a selected recovery point. Alternative instructions are included for replacing the disks of an existing VM or restoring disks for manual reconstruction.

**\3. Roles and Required Access

| \*\*Role\*\* | \*\*Responsibility\*\* | \*\*Typical access required\*\* |
| :----- | :----- | :----- |
| Requester / Application Owner | Confirms recovery point, outage window, application validation, and business acceptance. | Read access to affected resources; no restore permission required. |
| Cloud Operator | Performs the restore and documents evidence. | Permissions to read backup items and recovery points and create or update target compute, network, disk, resource group, and staging resources. |
| Security / Change Approver | Approves recovery where required and confirms security controls. | Per organizational RBAC and change-control policy. |

Note: Use organization-approved Azure RBAC roles or custom roles. Do not grant Owner solely to perform a restore.

**\4. Restore Method Selection

| \*\*Method\*\* | \*\*Use when\*\* | \*\*Impact\*\* | \*\*Important limitation\*\* |
| :----- | :----- | :----- | :----- |
| Create new VM (recommended default) | You need the safest recovery, a test restore, or the original VM is unavailable. | Creates a separate VM and related resources. Original VM is unchanged. | Avoid duplicate hostname/IP/domain identity conflicts. Isolate the restored VM until validation is complete. |
| Replace existing disks | The existing VM object must be retained and a point-in-time rollback is required. | Replaces the VM disks with disks from the selected recovery point. | The source VM must still exist. Azure Backup creates a snapshot of current disks before replacement. This is a destructive change to the active VM. |
| Restore disks | You need manual control, unsupported VM recreation, special networking, or selective reconstruction. | Creates managed disks and a deployment template for manual recovery. | Requires additional steps to create or repair the VM and attach disks. |
| File recovery | Only files or folders are needed. | Mounts recovery-point volumes temporarily by using a downloaded script. | Not a full VM recovery; use a separate file-recovery procedure. |

\5. Prerequisites and Pre-Restore Checklist

☐** An approved incident, service request, or emergency change record identifies the VM, requested restore date/time, reason, and business owner.

**☐** The VM is protected by Azure Backup and has at least one usable recovery point in the correct Recovery Services vault.

**☐** The target subscription, region, resource group, virtual network, subnet, VM name, and availability requirements are known.

**☐** Required resource providers are registered, quotas are sufficient, and Azure Policy will allow the restored resources.

**☐** The target subnet has sufficient IP capacity and required DNS, route table, network security group, firewall, and private endpoint connectivity.

**☐** For encrypted disks, the required Key Vault, keys, secrets, disk encryption set, permissions, and identities remain available.

**☐** The application owner has identified post-restore validation tests and the acceptable recovery point objective.

**☐** A plan exists to prevent duplicate computer names, domain identities, IP addresses, scheduled jobs, outbound interfaces, and application processing.

**☐** The current VM configuration has been recorded or exported, including VM size, zones, NICs, private IPs, public IPs, load balancer membership, tags, extensions, managed identities, disk layout, and boot diagnostics.

**\6. Procedure - Create a New VM from Backup

\6.1. Sign in to Azure Government**

Open a browser and go to https://portal.azure.us.

Sign in with the approved Azure Government account and complete multifactor authentication.

Confirm the correct Microsoft Entra tenant and subscription context before continuing.

**Expected result:** The Azure Government portal opens in the correct tenant.

**\6.2. Open the protected VM or Recovery Services vault**

Preferred path: search for and open the source Virtual machine. Under Backup + disaster recovery, select Backup.

Alternative path: search for Recovery Services vaults, open the vault that protects the VM, and select Backup items \> Azure Virtual Machine.

Select the protected VM backup item.

**Expected result:** The backup item page displays backup status, policy, and recovery points.

**\6.3. Confirm backup health and select Restore VM**

Confirm the most recent backup job completed successfully or identify the approved older recovery point.

Review the backup item status for warnings, soft-delete state, or suspended protection.

Select Restore VM.

**Expected result:** The Restore virtual machine workflow opens.

**\6.4. Select the recovery point**

Select Select restore point.

Choose the approved recovery point by date, time, consistency type, and recovery-point tier.

Confirm that the recovery point predates the incident but is not older than the approved recovery point objective.

Select OK or Select to return to the restore configuration page.

| \*\*CAUTION:\*\* The selected recovery point determines the state of the operating system and all included disks. Data written after that point will not exist on the restored VM. |
| :----- |

**Expected result:** The chosen recovery point appears in the restore configuration.

**\6.5. Choose Create new virtual machine**

For Restore configuration or Restore type, select Create new virtual machine.

Enter a unique restored VM name that follows the organization naming standard, for example \<source\>-RSTR-\<date\>.

Select the approved subscription and target resource group.

Select the target virtual network and subnet.

Choose the required staging location or storage account if prompted. Use an approved account in the target region and ensure the operator and Azure Backup have required access.

Review any displayed availability zone, availability set, disk, network, identity, or encryption settings.

| \*\*CAUTION:\*\* Do not connect a duplicate domain member or production application clone to the production network until identity and application conflicts have been addressed. |
| :----- |

**Expected result:** Azure validates the target settings and enables the Restore action.

**\6.6. Start the restore**

Review all selected values against the ticket and pre-restore checklist.

Select Restore.

Record the submission time, recovery point, restore method, target VM name, resource group, and operator in the ticket.

**Expected result:** Azure submits a restore job and displays a notification.

**\6.7. Monitor the restore job**

Open the Recovery Services vault.

Select Backup jobs or Jobs, and filter for the affected VM and Restore operation.

Open the restore job and monitor each task until the status is Completed.

If the job fails, record the error code and correlation details. Do not repeatedly resubmit without correcting the cause.

**Expected result:** The restore job shows Completed and the restored resources exist.

**\6.8. Inspect restored Azure resources before startup**

Open the restored VM and review Overview, Disks, Networking, Identity, Extensions + applications, Tags, Availability + scale, and Boot diagnostics.

Confirm the expected OS and data disks are attached and have the correct logical unit numbers where applicable.

Confirm the NIC is attached to the approved subnet and does not conflict with the source VM private IP or public IP.

Apply required tags, resource locks, backup policy, monitoring, endpoint protection, update management, and policy assignments that were not recreated automatically.

Keep the VM isolated or restrict network security group rules until operating-system and application checks are complete.

**Expected result:** The restored VM configuration is safe to start for controlled validation.

**\6.9. Start and validate the operating system**

Start the restored VM.

Use Azure Bastion, approved jump host, serial console, or other authorized administrative method to connect.

Confirm the VM boots without repair mode, disk errors, or extension failures.

Windows: verify Event Viewer, disk letters, services, time synchronization, activation status, domain relationship, and application dependencies.

Linux: verify journal/system logs, filesystems and mount points, network configuration, services, package state, and application dependencies.

Confirm endpoint security, vulnerability management, logging, monitoring agents, and backup extensions are healthy.

**Expected result:** The operating system is stable and administrative access works.

**\6.10. Validate the application and data**

Have the application owner run the approved smoke test and data validation checklist.

Confirm databases, services, file shares, scheduled tasks, certificates, secrets, service accounts, and external integrations are correct for the selected recovery point.

Confirm restored data timestamps and record counts match expectations.

Do not enable production traffic, message consumption, scheduled jobs, or outbound transactions until the owner authorizes cutover.

**Expected result:** The application owner documents pass/fail results and recovery acceptance.

**\6.11. Cut over or retain as a recovery copy**

For a recovery copy or forensic restore, keep the VM isolated and document the retention and deletion date.

For production cutover, stop or isolate the failed source VM, then implement the approved DNS, load balancer, private IP, public IP, route, firewall, and dependency changes.

Enable production traffic in a controlled sequence and monitor health, logs, and transactions.

Obtain business-owner acceptance and document the actual recovery time and recovery point.

| \*\*CAUTION:\*\* Never run the source and restored VM simultaneously when doing so could create duplicate identity, split-brain, data corruption, or duplicate transaction processing. |
| :----- |

**Expected result:** The restored service is operational or the recovery copy is retained under control.

**\7. Alternative Procedure - Replace Existing VM Disks

\7.1. Prepare the existing VM**

Confirm the original VM resource still exists and that Replace existing disks is supported for the VM and selected backup configuration.

Obtain explicit approval for the destructive rollback.

Stop application services cleanly where possible, then stop/deallocate the VM.

Record the current disk IDs, NIC configuration, VM size, zone or availability set, encryption settings, and dependencies.

**Expected result:** The existing VM is deallocated and ready for disk replacement.

**\7.2. Run Replace existing disks**

From the VM Backup page or its vault backup item, select Restore VM.

Select the approved recovery point.

Choose Replace existing or Replace existing disks.

Select the required staging location if prompted.

Review the warning and select Restore.

| \*\*CAUTION:\*\* This operation replaces the disks attached to the current VM with disks from the recovery point. Azure Backup takes a snapshot of the current VM disks in the specified staging location before replacement, but the active VM state is still being rolled back. |
| :----- |

**Expected result:** Azure submits the disk replacement restore job.

**\7.3. Monitor, start, and validate**

Monitor the restore job in the vault until it completes.

Confirm the VM is deallocated after the operation, then start it.

Perform all operating-system, application, data, security, monitoring, and business validation steps in Sections 6.9 through 6.11.

**Expected result:** The original VM object is running with restored disks and has passed validation.

**\8. Alternative Procedure - Restore Disks for Manual Reconstruction

\8.1. Restore managed disks**

From the protected VM backup item, select Restore VM and choose the approved recovery point.

Select Restore disks.

Select the target subscription, resource group, and staging location as prompted.

Select Restore and monitor the job to completion.

**Expected result:** Azure creates restored managed disks and, when applicable, a deployment template in the staging location.

**\8.2. Reconstruct the VM**

Review the generated template and restored disks.

Create a VM from the restored OS disk or deploy and modify the generated template.

Attach restored data disks in the original order and apply the required caching settings and logical unit numbers.

Recreate NICs, network settings, identities, extensions, availability configuration, encryption, tags, monitoring, and backup protection as required.

Continue with the validation and cutover controls in Sections 6.8 through 6.11.

**Expected result:** A manually reconstructed VM is operational and validated.

**\9. Post-Restore Validation Checklist

☐** Azure restore job status is Completed and evidence is attached to the ticket.

**☐** VM power state, provisioning state, boot diagnostics, and guest agent are healthy.

**☐** All expected OS and data disks are attached, accessible, and mapped correctly.

**☐** Network, DNS, routes, NSGs, firewall rules, load balancers, private endpoints, and IP addressing are correct.

**☐** Managed identities, Key Vault access, certificates, encryption, and secrets function correctly.

**☐** Critical OS and application services are running; logs show no unresolved restore-related errors.

**☐** Security, endpoint protection, vulnerability scanning, logging, monitoring, alerting, and patch management are active.

**☐** Azure Backup protection is enabled and a successful post-restore backup has been scheduled or completed.

**☐** Application owner completed data and functional validation and approved production use.

**☐** Duplicate or temporary resources are isolated, labeled, and assigned a cleanup date.

**\10. Failure Handling and Rollback

| \*\*Condition\*\* | \*\*Required action\*\* |
| :----- | :----- |
| Restore job fails | Capture the full job error, activity log correlation ID, resource IDs, time, and screenshots. Check permissions, policy denials, quota, target naming, staging location, encryption dependencies, region support, and networking. Escalate through the approved support path. |
| Restored VM does not boot | Do not repeatedly restart. Review boot diagnostics and serial console. Use VM repair or restore disks for offline repair. Preserve the failed restore for evidence until the incident owner approves deletion. |
| Application validation fails | Keep production traffic disabled. Compare the recovery point, attached disks, dependencies, secrets, time, DNS, and application configuration. Select a different recovery point only with owner approval. |
| Cutover causes service degradation | Reverse DNS/load balancer/routing changes, stop or isolate the restored VM, and return traffic to the approved prior service state when safe. Record the rollback and continue incident management. |
| Replace-disks rollback is required | Use the pre-replacement disk snapshots created in the staging location or another approved recovery point, following change approval and Microsoft guidance. |

\11. Cleanup and Closure

☐** Delete temporary restored VMs, disks, NICs, public IPs, snapshots, staging blobs, and resource groups only after the retention period and owner approval.

**☐** Confirm no required rollback snapshot or forensic evidence is deleted prematurely.

**☐** Remove temporary firewall, NSG, route, DNS, load-balancer, and privileged-access changes.

**☐** Confirm backup protection and policy assignments for the production VM.

**☐** Update the ticket with recovery point, start/end times, restore method, validation evidence, cutover actions, issues, and final approver.

**☐** Create a problem record or lessons-learned action when the event exposes backup, monitoring, access, quota, policy, or documentation gaps.

**\12. Evidence to Retain**

Retain the following in the incident or change record according to organizational policy: approval; source VM and vault identifiers; selected recovery point; restore method; target resource names; restore job ID/status; screenshots or exported logs; validation results; cutover and rollback actions; business acceptance; and cleanup confirmation.

**\13. References**

_Microsoft Learn - Restore Azure virtual machines by using the Azure portal

Microsoft Learn - Manage and monitor Azure VM backups

Microsoft Learn - Support matrix for Azure VM backups

Microsoft Learn - Recover files from an Azure virtual machine backup

Azure Government portal_

**\14. Revision History

| \*\*Version\*\* | \*\*Date\*\* | \*\*Author\*\* | \*\*Description\*\* |
| :----- | :----- | :----- | :----- |
| \\1.0 | July 2026 | OpenAI draft for organizational review | Initial Azure Government VM restore work instruction. |

