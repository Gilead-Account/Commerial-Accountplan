# Post Launch Checklist implementation

The Post Launch Checklist is available from the selected account plan's sidebar. CRM records belong to the Account; reach belongs to the selected Account Plan. The page presents one checklist, with no Commercial/Medical switch or function-specific heading. It retains the existing Commercial / Market Access record scope. Sample tasks and dates are confined to the mock fixtures and excluded from production bundles.

The table supports sorting all five columns, immediate status saves, and comments saved on blur with a 255-character limit. Failed status writes roll back. Failed comment writes keep the draft and offer a retry. Same-record writes are queued, different records save independently, and cache updates merge only the saved fields. Checklist reads wait for pending Account writes and reconcile successful edits that finish during a read. Updates finishing after an Account switch cannot change the newly selected Account's cache.

The page follows the user's complete visual reference: four pale-blue dashboard cards with blue top borders and circular line icons, followed by a yellow Guidance & Resources panel in the same desktop summary row. The checklist sits below, with a rounded light-blue header, alternating white/pale-blue rows, navy task text, rounded status pills, MM/DD/YYYY due dates, and a wider comments column with “Add comments” placeholders. On smaller screens the summary and resources wrap into usable rows. Comment save feedback and character counts appear during interaction, keeping untouched rows compact. The function switcher and its labels are absent. The user explicitly confirmed keeping the current account tasks.

## CRM integration

Read `KPH_Post_Launch_Checklist_Item__c`, filtering `KPH_Account__c` to the selected plan's Account and `KPH_Checklist_Function__c` to the two supported functions. Only `KPH_Status__c` and `KPH_Comments__c` are writable. The page never provisions tasks, due dates, or roles.

Reach reads active `KPH_Stakeholder_Map__c` rows through `KPH_Account_Plan__c`, using `KPH_Account__c`, `KPH_MM_Target__c`, `KPH_MM_HCP_Classification__c`, and `KPH_Is_Deleted__c`. It deduplicates equivalent 15/18-character IDs and counts eligible calls through `Call2_vod__c.Account_vod__c` and `Call_Date_vod__c`. The call date and association fields follow [Veeva's mobile object documentation](https://crmhelp.veeva.com/doc/Content/CRM_topics/Getting_Started/MobileDeviceSetup/Sync/VMOCs.htm). Queries use the shared online/offline IN and ID helpers and avoid relationship fields.

## Scope choices and open release decisions

- The checklist follows the application's existing approval/profile view-only flag. Final business approval of that policy and the function permission matrix is still required; CRM must enforce object/field/record permissions.
- The Commercial/Medical function selector was removed at the user's request. Medical records are not exposed by this page.
- Reach retains the PRD baseline: January 4, 2027 start, fewer than 60 calendar days old, no future calls, no call-status filter. Dates use the device's local calendar day. The constants live in `src/lib/postLaunchChecklist.js`; date, status eligibility, and time-basis policy need business confirmation.
- Trifecta retains exact-description matching and the documented matching-record count, including duplicates, with a denominator of three. Stable identifiers and duplicate policy remain business decisions.
- MM Activation date and 1st Aph show unavailable placeholders until authoritative CRM fields are supplied. The dashboard can display supplied `milestones.mmActivationDate` and `milestones.firstApheresis` values, with MM-DD-YYYY activation dates; the screenshot's example values are not hard-coded into production.
- The four resource categories show noninteractive “Work in Progress” text until approved destinations are provided.
- The production catalog, accountable roles, eligibility, due dates, CRM permissions, and supported iPad synchronization must be supplied or validated through the CRM release process.

## Verification

Run `node --test test/postCallExecution.test.mjs` for mapping, sorting, validation, isolation, write ordering/failure recovery, read/save races, and reach boundary checks. Run `npm run build` for the live-data-only MyInsights package. Use development/mock mode to review sample content; use the built production package inside Veeva to validate actual reads, writes, and device sync. Browser mocks and local packaging checks do not establish live CRM or native iPad compatibility.

Local verification on November 12, 2026: all 31 checks passed across `postCallExecution`, `submittedApprovalReadOnly`, `navigationResponsiveness`, `tableSortingBehavior`, and `queryInFlightCoalescing`. The production build passed. Package verification confirms a single root `index.html`, byte equality with the built HTML, both embedded runtime scripts in order, exclusion of the preview catalog and removed selector, and no matches in the secret-indicator scan. Evidence is in `outputs/post-launch-package-verification.json`.

The local Chrome browser flow checks the single checklist with no function selector/heading, reference styling and numeric dates, dashboard and resources above the table, status rollback, comment retry, concurrent field saves, navigation persistence, 255-character and empty comments, unchanged-comment suppression, sort controls, and absence of horizontal page overflow or browser errors. Screenshots are in `outputs/post-launch-checklist-*.png`, including an isolated page and table preview.

The final full-page layout check passed at 2048×1300 (2× device pixel ratio) and at 1500, 1024, and 820 pixels wide: four icon cards plus the yellow resources panel share one desktop row above the table; untouched table rows are 76 pixels high; all nine current tasks remain; the function switcher is absent; unavailable milestones remain placeholders; no page overflow or browser errors were detected.

The additional `sidebarNavigationState` check has two existing failures: “sidebar Account Planning Playbook is marked TBD without a document link” expects a link to be absent although the existing sidebar links the bundled playbook, and “sidebar Account Plan Legal and Compliance Guidance uses the bundled PDF asset” expects different filename casing. These document settings were present before checklist work and were not changed.
