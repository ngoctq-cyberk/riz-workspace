# project-approval-ui Specification

## Purpose
TBD
## Requirements

### Requirement: Admin Projects Management uses moderation tabs

The Projects Management screen SHALL default to Pending and SHALL expose Pending, Published, and Rejected tabs. Pending SHALL offer Publish and Reject actions. Published SHALL offer Reject action. Rejected SHALL offer Publish action.

#### Scenario: Admin opens Projects Management

- **GIVEN** admin opens the Projects Management screen
- **THEN** the Pending tab is selected by default

#### Scenario: Admin moderates selected pending projects

- **GIVEN** admin selected projects in Pending tab
- **WHEN** admin clicks Publish
- **THEN** selected ids are sent only in `publishIds`
- **WHEN** admin clicks Reject
- **THEN** selected ids are sent only in `rejectIds`

### Requirement: App communicates pending review

Project creation success copy SHALL say the project is pending review. Creator Studio SHALL show a project status badge for the current user's projects.

#### Scenario: User creates project

- **GIVEN** project create request succeeds
- **THEN** app shows pending-review success copy

#### Scenario: User opens Creator Studio

- **GIVEN** a project response includes `status = PENDING_APPROVAL`
- **THEN** the project card shows a Pending approval badge

