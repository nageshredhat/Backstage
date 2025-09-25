# Project Progress Tracker

Use this file to capture ongoing updates from the Backstage-on-Minikube project. Each section below represents a snapshot of the current status, responsibilities, and next steps.

---

## Team Updates

| Name       | Work Completed                                                                                                                                                                                                     | Current Tasks                                                                                                                                                                                                                                         | Blockers / Issues                                                                    | Target / ETA                                                      |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| **Nagesh** | - **25 Sep 2025:** Created bare-minimum manifests for Backstage and Postgres, including dependent files like `backstage.yaml`. <br>- Achieved successful deployment with both Backstage and Postgres pods running. | - Investigate GitHub credentials not being fetched inside Backstage pod (credentials already defined in ConfigMap `app-config`). <br>- **3 Oct 2025:** Resolved scaffolder GitHub issue and successfully fetched the first template into the cluster. | GitHub credentials were not being read from the Backstage pod environment initially. | Scaffolder GitHub template integration complete: **Oct 03, 2025** |

---

## Overall Progress Summary

* **Cluster Setup:** ✅ Completed on both Windows WSL and macOS test machines.
* **GitHub Credentials:** ⚠️ Initial issue: Backstage pod did not pick up credentials from ConfigMap `app-config`. Resolved on **Oct 03, 2025** by Nagesh.
* **Backstage Deployment:** ✅ Running with 1 replica.
* **Postgres Database:** ✅ Running with 1 replica.
* **Testing & Verification:** Scaffolder GitHub template successfully fetched.

---

## Blockers & Risks

* **OAuth Credentials Delay:** Need GitHub admin to provide final Client ID & Secret.
* **Resource Constraints:** Local laptops may need >4GB RAM for stable pods if additional plugins are added.
* **Image Pull Rate Limits:** Docker Hub throttling could slow down fresh environment setup.
* **GitHub Credentials Issue:** Initially, Backstage pod did not correctly consume credentials from the `app-config` ConfigMap; fixed by Nagesh on **Oct 03, 2025**.

---

## Release Plan

* **Release Name:** `backstage-local-alpha`
* **Targeted Release Date:** **Oct 05, 2025** (adjust based on blockers)
* **Scope:**

  * Backstage running locally on Minikube.
  * GitHub OAuth login functional.
  * Postgres backend persistent volume configured.
  * Basic documentation (README & progress) finalized.

---

## Next Steps

1. Finalize and distribute GitHub OAuth credentials.
2. Apply updated manifests and verify pods.
3. Run smoke and integration tests.
4. Document lessons learned and any improvements for future iterations.

---

> **Update Protocol:**
>
> * Nagesh should add a dated row in **Team Updates** whenever a milestone is completed or a blocker is encountered.
> * Keep **Overall Progress Summary** current at least once per sprint or key milestone.
