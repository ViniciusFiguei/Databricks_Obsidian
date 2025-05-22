Topics:

- [[#Cluster policies|Cluster policies]]
## Cluster policies

**Cluster Policies** in Databricks are a way to **define and enforce rules** for creating and managing clusters in Databricks environment. These policies allow administrators to set specific configurations and restrictions, ensuring that users and teams adhere to best practices, cost control, and resource management when creating clusters. Basically, a Cluster Policy allows you to limit the options of configurations that a user or group can set when creating a new cluster.  Without Cluster Policies, users might unknowingly create giant clusters that burn the budget in few hours, or minutes! 

### How Cluster Policies Work:

- **Creating Policies**: Databricks administrators define **Cluster Policies** from the **Admin Console**. Each policy is composed of rules that specify valid values for various cluster parameters (e.g., node types, instance sizes, and autoscaling configurations).
- **Applying Policies**: Once created, policies can be **applied** to users, groups, or teams. These users/groups can only create clusters that **adhere to the defined rules** of the policy.
- **Default "Personal Compute" Policy**: By default, Databricks includes a **Personal Compute** policy that is available to all users. This allows users to create basic, personal-use clusters but with limited configurations. However, administrators can create more restrictive or customized policies that apply to specific users or teams.

### Benefits of Cluster Policies:

- **Cost Control**: Prevents overspending by limiting users to smaller and more cost-efficient clusters.
- **Simplification**: Helps standardize cluster creation and reduce human error by enforcing predefined settings.
- **Security**: Restricts the creation of clusters with specific settings that might pose security or compliance risks.
- **Operational Efficiency**: Helps manage resources across an organization and ensures efficient use of clusters.