---
title: Sprint Turnover
doc_status: outdated
last_reviewed:
reviewed_by:
review_notes:
---
# Sprint Turnover

The following process for sprint turnover should be followed to make the process robust and repeatable.

## Close Previous Sprint

1. Ensure that all issues in sprint are updated to reflect their current state.
2. Close sprint and select what to do with unfinished issues (push to next sprint or top of backlog).
3. Review Jira sprint report:
   - Document reasons for issues added during sprint. Common reasons are:
     - Emergent, high priority work, e.g. bug that blocks progress on issue in sprint.
     - Gap discovered in issues to accomplish sprint goals.
   - Document reasons for unfinished issues. Common reasons are:
     - Not enough time.
     - Underestimated complexity.
     - Blocked (document blockers).
4. Assign unfinished issues to upcoming sprints, or send to backlog.

## Review Upcoming Sprint

1. Estimate any unestimated issues.
2. Review sprint goals and ensure they are up-to-date.
3. Ensure issues in sprint align and fully achieve sprint goals.
4. Review sprint story point total and ensure sprint is achievable.
   - If sprint has too many story points then try to descope issues that do not contribute to sprint goals. If no suitable issues can be descoped then select issue to descope but inform stakeholders of impact to sprint goals.
   - If sprint has too few story points then bring in issues that are achievable and will contribute to upcoming sprint goals.
5. When sprint is finalised, take a confidence vote from developers.
   - If confidence is low then ask reasons. Repeat step 4 and 5 until confidence is >=3 or until no further rescoping can take place without affecting sprint schedule. It may be necessary to seek more resource to achieve critical sprint goals.
6. When sprint is approved, start the sprint and add the sprint start and end times (default is a 2-week sprint). Issues for the sprint are now locked in and any modifications to sprint scope will be tracked by Jira.

## Plan Deployment to Test Cluster

1. Discuss current state of the EODHP platform and whether it is ready for deployment to `test` cluster.
2. If it cannot be deployed in its current state then:
   - Scope the updates so that the `test` cluster will be operational while maximising functionality.
   - Avoid regressions as a priority.
3. Nominate a developer to be responsible for the deployment to `test` cluster.

## Post Sprint Activities

1. The scrum master should prepare a succinct Sprint Review Report detailing:
   - Sprint goals achieved:
     - A short description of new stories.
     - A list of bugs fixed.
   - Sprint goals not achieved:
     - Incomplete issues and reasons for slip.
     - Updated schedule for incomplete issues.
   - Justification for issues added during sprint.
   - Sprint velocity:
     - Story points achieved in sprint.
     - Rolling 3 month average.
     - Detail any factors that may have affected velocity, e.g. holidays.
2. Deploy agreed functionality to `test` cluster.
