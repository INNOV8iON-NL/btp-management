# Exercise 10: Manually (ClickOps) Provisioning a SAP BTP Landscape

In this **Exercise 10**, you’ll create a SAP BTP landscape **by hand**—often referred to as _clickops_. In the upcoming **Exercise 11**, you’ll see how to accomplish the same setup using Terraform. By going through these manual steps first, you’ll understand what Terraform automates for you.

---

## 1. Goal of This Exercise

By the end of this exercise, you will have:

1. A **Directory** within your **Global Account**.  
2. Two **Subaccounts** (one for “Development,” one for “Acceptance”) inside that Directory.  
3. **Entitlements** assigned to each Subaccount (for the Cloud Foundry environment and an extra service).  
4. A **Cloud Foundry** environment enabled in each Subaccount.  
5. A **Cloud Foundry Org** (implicitly created when you enable Cloud Foundry).  
6. A **Cloud Foundry Space** within each Subaccount.  
7. An **XSUAA** service instance (plan `apiaccess`) in each Space.  
8. **Roles** assigned to your user to manage the Cloud Foundry spaces.

You’ll be doing all of this via the SAP BTP Cockpit (and the Cloud Foundry UI if needed). Think of it as a step-by-step guide to provisioning your BTP resources _manually_.

---

## 2. Prerequisites

- **SAP BTP Global Account**: You need a global account (and the appropriate role collections) to create directories, entitlements, subaccounts, etc.
- **Entitlements Available**: Ensure your global account is entitled to use the Cloud Foundry environment and the `credstore` (or whatever extra services you want to add).
- **Admin or Sufficient Privileges**: You must be able to manage directories, subaccounts, entitlements, and roles.

> The UI may differ based on your SAP BTP Cockpit version or your role settings, but the overall steps are similar.

---

## 3. Create a Directory

1. **Log in** to the [SAP BTP Cockpit](https://cockpit.btp.cloud.sap).
2. From the left-side navigation, ensure you’re viewing your **Global Account**.
3. Navigate to **Directories** (might be under **Subaccounts & Directories**).
4. Click **Create Directory** or **New Directory**.
5. Provide:
   - **Name**: For example, **Julian** or **ParentDirectory** (any name you prefer).
   - **Description**: e.g., “Parent directory for the subaccounts we’ll create.”
6. (Optional) Add **Labels** (e.g., `ManagedBy=Terraform`, `Contact=your@mail.com`).
7. Click **Create**.

**Result**: You have a new directory in your global account.

---

## 4. Create Two Subaccounts

Next, create two subaccounts in that directory—one for development (Dev) and one for acceptance (Acc).  

### 4.1 Subaccount for Dev

1. In your newly created directory, click **Create Subaccount**.
2. In the **Create Subaccount** dialog:
   - **Name**: `yourname-dev` (replace `yourname` with something unique).
   - **Subdomain**: `i8-yourname-dev` (must be globally unique).
   - **Region**: e.g., `eu10`.
   - **Description**: “My development subaccount.”
   - **Labels**: e.g., `ManagedBy=Terraform`, `Contact=your@mail.com`.
3. Save or **Create**.

### 4.2 Subaccount for Acc

Repeat the same steps:

1. Still in the directory, click **Create Subaccount** again.
2. **Name**: `yourname-acc`.
3. **Subdomain**: `i8-yourname-acc`.
4. **Region**: same as Dev, e.g., `eu10`.
5. **Description**: “My acceptance subaccount.”
6. (Optional) **Labels** as desired.
7. Save or **Create**.

**Result**: Two subaccounts (`Dev` and `Acc`) created in your directory.

---

## 5. Assign Entitlements

Each subaccount needs entitlements for the services they’ll use. In our scenario, we’ll assign:

- **Cloud Foundry environment** (plan: `standard`).
- **credstore** (plan: `free`).

### 5.1 Add Entitlements to the Dev Subaccount

1. Go to your **Global Account** → **Entitlements** (the name may differ slightly).
2. Find **Configure Entitlements** or **Entity Assignments** (varies with cockpit version).
3. Select the **Dev** subaccount from the list.
4. Add or configure:
   - **Cloud Foundry Runtime** (plan `standard`).
   - **credstore** (plan `free`).
5. Assign the needed quotas/amounts (if applicable).
6. Save.

### 5.2 Add Entitlements to the Acc Subaccount

Repeat for the **Acc** subaccount:

1. Return to **Global Account** → **Entitlements**.
2. Select the **Acc** subaccount.
3. Assign **Cloud Foundry Runtime** (plan `standard`) and **credstore** (plan `free`).
4. Save.

**Result**: Both subaccounts are now entitled to use Cloud Foundry and credstore.

---

## 6. Enable Cloud Foundry in Each Subaccount

### 6.1 Enable CF in Dev

1. Go to your **Dev** subaccount.
2. Click **Environments** → **Cloud Foundry** (or similar).
3. Click **Enable Cloud Foundry** (or **Create**).
4. Provide a name (often the subdomain), plan `standard`, and the region if prompted.
5. Confirm.

### 6.2 Enable CF in Acc

Repeat in the **Acc** subaccount:

1. Go to **Environments** → **Cloud Foundry**.
2. Enable or **Create** Cloud Foundry environment.
3. Confirm with plan `standard`.

**Result**: Each subaccount has an active Cloud Foundry environment.

---

## 7. Create a Cloud Foundry Space

### 7.1 Dev Space

1. In the **Dev** subaccount, go to **Cloud Foundry** → **Spaces**.
2. Click **Create Space**.
3. Name it `dev`.
4. Save.

### 7.2 Acc Space

1. In the **Acc** subaccount, go to **Cloud Foundry** → **Spaces**.
2. Click **Create Space**.
3. Name it `acc`.
4. Save.

**Result**: Each subaccount has a dedicated space.

---

## 8. Assign Roles in Each Space

Grant your user (the one you’re logged in with) the roles of **Space Manager** and **Space Developer** so you can manage services.

1. In the **Dev** subaccount, **Cloud Foundry** → **Spaces** → `dev` space.
2. Look for **Members** or **Role Assignments**.
3. Assign your user as:
   - Space Manager
   - Space Developer
4. Repeat the same in the **Acc** subaccount → `acc` space.

**Result**: You can now manage service instances in both spaces.

---

## 9. Create XSUAA Service Instance (Plan: `apiaccess`)

We’ll add the `credstore` plan a bit later if you’d like; for now, let’s do the XSUAA instance.

### 9.1 Dev XSUAA Instance

1. In the **Dev** subaccount, go to **Services** → **Service Marketplace**.
2. Find **XSUAA** (may be labeled “Authorization & Trust Management”).
3. Click **Create** or **Add Instance**.
4. Choose:
   - **Space**: `dev`.
   - **Plan**: `apiaccess`.
   - **Instance Name**: `xsuaa-apiaccess` (or similar).
5. Confirm creation.

### 9.2 Acc XSUAA Instance

Repeat in the **Acc** subaccount:

1. **Services** → **Service Marketplace** → **XSUAA**.
2. **Space**: `acc`.
3. **Plan**: `apiaccess`.
4. **Instance Name**: `xsuaa-apiaccess`.
5. Confirm creation.

**Result**: Each CF space has an XSUAA instance using the `apiaccess` plan.

---

## 10. Verify Your Setup

You’ve now created:

- **1 Directory** in your global account.
- **2 Subaccounts** (Dev & Acc) in that directory.
- **Entitlements** for CF & `credstore`.
- **Enabled Cloud Foundry** in both subaccounts → each has its own CF Org.
- **1 CF Space** in each subaccount (`dev` in Dev, `acc` in Acc).
- **Roles** assigned so you can manage the spaces.
- **XSUAA Service Instances** in each space.

You can view these resources in the SAP BTP Cockpit:
- **Directory**: in the global account’s Directories view.
- **Subaccounts**: nested in the directory.
- **Spaces** and **Service Instances**: within each subaccount’s Cloud Foundry tab.

---

## 11. Reflecting on Manual vs. Automated

Doing all of this by hand can be time-consuming and prone to errors. In **Exercise 11**, we’ll see how to automate this entire process using Terraform, ensuring consistency, reusability, and easier maintenance. But it’s helpful to see the manual steps at least once to appreciate what Infrastructure as Code accomplishes under the hood.

---

## Next Steps

- Proceed to **Exercise 11** to learn how to automate this exact setup with Terraform.  
- Compare the steps you’ve done here (clickops) with what Terraform does in a few lines of code.  
- Understand how storing the environment state in code improves collaboration and consistency.

**Congratulations!** You have now manually provisioned your SAP BTP landscape.