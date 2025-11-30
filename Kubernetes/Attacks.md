## Exploiting Overly-Permissive Service Accounts
• In many CI/CD setups, tools like ArgoCD, Jenkins, or GitLab Runner are granted excessive permissions via Kubernetes service accounts
• These are often mistakenly given cluster-admin rights or broad RBAC roles that attackers can abuse to compromise the entire cluster

Common Misconfiguration • Service account with cluster-admin or * permissions
• CI/CD pods using default service accounts without restriction
• Lack of role scoping by namespace or action
• Tokens mounted automatically into pods (token auto-mounting enabled)

