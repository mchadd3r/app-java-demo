---
name: endor-labs
description: How to use Endor Labs tools and API
---

# Endor Labs Skill

Endor Labs is a comprehensive application security platform which offers Reachability SCA, SAST, and Secrets Scanning.  It also offers a rich API and CLI tooling to integrate into your development workflow.  This skill will help you understand how to use the Endor Labs tools and API to secure your applications.

## Identifying Vulnerabilities in Dependencies
Use the following command to list all Critical and Reachable vulnerabilities which have a fix available.

```
endorctl api list -r Finding --filter="context.type==CONTEXT_TYPE_MAIN and spec.finding_tags not contains [FINDING_TAGS_EXCEPTION] and spec.finding_tags contains [FINDING_TAGS_NORMAL] and spec.finding_categories contains [FINDING_CATEGORY_SCA] and spec.project_uuid==\"68a650281e75c81628aedaa7\" and spec.finding_tags contains [FINDING_TAGS_REACHABLE_FUNCTION] and spec.level contains [FINDING_LEVEL_CRITICAL] and spec.finding_tags contains [FINDING_TAGS_FIX_AVAILABLE]" --field-mask "spec.finding_metadata.vulnerability.meta.name,spec.finding_metadata.vulnerability.meta.description,spec.location_urls" --list-all
```

### Dependency Version Upgrades
Endor Labs can produce a list of safe version upgrades for dependencies which fix vulnerabilities.  The following command will list all of the best upgrades for a project.  The `--field-mask` parameter can be used to specify which fields to return. This feature is called Upgrade Impact Analysis. If prompted to upgrade a dependency, use this command to get a list of safe upgrades. If there are multiple options returned, pick the one which fixes the most vulnerabilities and has the least impact.  Impact is determined by the `spec.upgrade_info.upgrade_risk` field, which can be low, medium, or high.


```
endorctl api list -r VersionUpgrade --filter="context.type==CONTEXT_TYPE_MAIN and spec.project_uuid==\"68a650281e75c81628aedaa7\" and spec.upgrade_info.is_best==true and spec.upgrade_info.worth_it==true" --field-mask="spec.name,spec.upgrade_info.is_best,spec.upgrade_info.is_latest,spec.upgrade_info.from_version,spec.upgrade_info.to_version,spec.upgrade_info.to_version_age_in_days,spec.upgrade_info.total_findings_fixed,spec.upgrade_info.total_findings_introduced,spec.upgrade_info.score_explanation,spec.upgrade_info.worth_it,spec.upgrade_info.upgrade_risk,spec.upgrade_info.direct_dependency_package" --list-all
```

### Upgrade Refactoring
High risk upgrades will always have potentially breaking code-level changes identified through Endor Labs call graphs. If a user wants to consider a high risk dependency upgrade, fetch the `uuid` from the previous command and use it in the following command to get the code-level changes which need to be refactored.

```
endorctl api list -r VersionUpgrade --filter="context.type==CONTEXT_TYPE_MAIN and spec.project_uuid==\"68a650281e75c81628aedaa7\" and uuid==\"68a65225ec7ae1e876f7bd7f\"" --field-mask="spec.upgrade_info.cia_results"
```

## SAST
Use the following command to retrieve all Critical SAST findings for the project. The information returned will be a description, explanation, and location of the finding.

```
endorctl api list -r Finding --filter="context.type==CONTEXT_TYPE_MAIN and spec.finding_tags not contains [FINDING_TAGS_EXCEPTION] and spec.finding_tags contains [FINDING_TAGS_NORMAL] and spec.finding_categories contains [FINDING_CATEGORY_SAST] and spec.project_uuid==\"68a650281e75c81628aedaa7\" and spec.level contains [FINDING_LEVEL_CRITICAL]" --field-mask "meta.description,spec.level,spec.finding_metadata.custom.location,spec.explanation" --list-all
```

## Operational Risk
Use the following command to retrieve all Operational Risk findings for the project. The information returned will be a description, explanation, and location of the finding.

```
endorctl api list -r Finding --filter="context.type==CONTEXT_TYPE_MAIN and spec.finding_tags not contains [FINDING_TAGS_EXCEPTION] and spec.finding_tags contains [FINDING_TAGS_NORMAL] and spec.finding_categories contains [FINDING_CATEGORY_OPERATIONAL] and spec.project_uuid==\"68a650281e75c81628aedaa7\"" --field-mask "meta.description,spec.dependency_file_paths,spec.explanation,spec.finding_metadata.source_policy_info.description,spec.remediation" --list-all
```