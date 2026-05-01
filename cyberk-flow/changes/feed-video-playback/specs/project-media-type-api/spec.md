## ADDED Requirements

### Requirement: Project image response exposes media type

The system SHALL include a `type` field on each item of `ProjectImage[]` returned by project endpoints (`GET /project`, `GET /project/:id`, bookmarked project lists, trending feed). The value SHALL be mapped from `Media.type` in the database.

#### Scenario: Project with mixed media

- **WHEN** the response includes a project that has 1 IMAGE attachment and 1 VIDEO attachment
- **THEN** `images[0].type === "IMAGE"` and `images[1].type === "VIDEO"`

#### Scenario: Project with only images

- **WHEN** the response includes a project whose attachments are all IMAGE
- **THEN** every entry in `images[]` has `type === "IMAGE"`

#### Scenario: Backward compatibility

- **WHEN** an old client that does not read `type` parses the response
- **THEN** all pre-existing fields (`id`, `url`, `mediumUrl`, `thumbnailUrl`, `order`, `width`, `height`, `aspectRatio`) are unchanged and the response parses successfully

### Requirement: ProjectImageEntity documents `type` in Swagger

The `ProjectImageEntity` class SHALL declare the `type` field with `@ApiPropertyOptional({ enum: MediaType })` and `@Expose()` so that `/docs` reflects the new contract.

#### Scenario: Swagger renders enum

- **WHEN** the developer opens `/docs` and inspects `ProjectImageEntity`
- **THEN** the `type` property is documented as enum `IMAGE | DOCUMENT | VIDEO`

### Requirement: Project response exposes hasVideo

The system SHALL include `hasVideo` on `ProjectEntity`, computed from `_count.attachments` where attachment `type === VIDEO` and `usage === PROJECT_IMAGE`.

#### Scenario: Project has video attachment

- **WHEN** the response includes a project with at least one video attachment
- **THEN** `hasVideo === true`

#### Scenario: Project has no video attachment

- **WHEN** the response includes a project with no video attachments
- **THEN** `hasVideo === false`
