# Project Progress Tracker

Use this file to capture ongoing updates from the Backstage-on-Minikube project. Each section below represents a snapshot of the current status, responsibilities, and next steps.

---

## Team Updates

| Name       | Work Completed                                                                                                                                                                                                     | Current Tasks                                                                                                                                                                                                                                         | Blockers / Issues                                                                    | Target / ETA                                                      |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| **Nagesh** | - **25 Sep 2025:** Created bare-minimum manifests for Backstage and Postgres, including dependent files like `backstage.yaml`. <br>- Achieved successful deployment with both Backstage and Postgres pods running. | - Investigate GitHub credentials not being fetched inside Backstage pod (credentials already defined in ConfigMap `app-config`). <br>- **3 Oct 2025:** Resolved scaffolder GitHub issue and successfully fetched the first template into the cluster. | GitHub credentials were not being read from the Backstage pod environment initially. | Scaffolder GitHub template integration complete: **Oct 03, 2025** |

---

