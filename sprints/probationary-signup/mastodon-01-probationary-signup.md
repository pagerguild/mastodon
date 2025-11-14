# Add Probationary Signup Mode to Mastodon

**Story ID:** mastodon-01-probationary-signup
**Difficulty:** Hard
**Estimated Time:** 60 minutes

## User Story

As an instance administrator, I want a probationary signup mode so that new users can register immediately while their posts are held for moderation, providing a balance between open registration and spam prevention.

## Description

Implement a third registration mode called "probationary signups" alongside the existing open and approval-required options. New users in probationary mode can register immediately, customize their profiles, and draft content. However, their posts remain queued for moderator review until their account is validated. Once approved, queued content publishes normally.

## Background

The current binary choice forces administrators to choose between:
- **Open registration**: Enables rapid user harassment and spam proliferation
- **Manual approval**: Creates onboarding friction causing legitimate user abandonment (81% of users abandon forms they start)

This feature provides a middle ground that protects marginalized communities proactively while maintaining accessibility.

## Acceptance Criteria

### Administrator Settings
- [ ] Add new registration mode option: "Probationary signups"
- [ ] Admin panel has option to enable/disable probationary mode
- [ ] Admin can configure probationary mode alongside existing open/approval-required modes
- [ ] Configuration is persisted in the database

### New User Registration
- [ ] Users can register immediately without waiting for approval
- [ ] New users can customize their profile (avatar, bio, display name)
- [ ] New users can draft posts and replies
- [ ] New users see indicator that their account is in probationary status
- [ ] New users see message explaining posts will be reviewed before publishing

### Post Moderation Queue
- [ ] Posts from probationary users are held in moderation queue
- [ ] Moderators can view queued posts in admin interface
- [ ] Moderators can approve individual posts
- [ ] Moderators can reject posts with reason
- [ ] Posts only publish locally (no federation) until account approved

### Account Approval
- [ ] Moderators can approve probationary accounts
- [ ] When account approved, all queued posts are published
- [ ] Approved users transition to full member status
- [ ] Moderators can reject probationary accounts
- [ ] Rejected accounts are deleted along with queued content

### User Experience
- [ ] Probationary status indicator visible on user profile
- [ ] Clear messaging about content review process
- [ ] Email notification when account is approved
- [ ] Email notification when posts are approved/rejected

### API Requirements
- [ ] New API endpoint to check account probationary status
- [ ] API returns probationary flag in user account responses
- [ ] API returns queued posts for probationary users
- [ ] API endpoint for moderators to approve/reject posts
- [ ] API endpoint for moderators to approve/reject accounts

## Technical Requirements

- **Backend:** Ruby on Rails
- **Database:** PostgreSQL
- **Frontend:** React/TypeScript
- **Authentication:** Devise/Doorkeeper

## Implementation Notes

Consider these approaches:

### Database Changes
- Add `registration_mode` enum to settings (open, approval_required, probationary)
- Add `probationary_status` column to accounts table (null, pending, approved, rejected)
- Add `moderation_status` column to statuses table (null, queued, approved, rejected)
- Add `probationary_approved_at` timestamp to accounts

### Backend Implementation
- Create `ProbationaryAccountService` to handle account lifecycle
- Create `PostModerationService` to handle post approval workflow
- Update `RegisterAccountService` to support probationary mode
- Add moderation queue views and controllers
- Update email templates for notifications
- Prevent federation for probationary account posts

### Frontend Implementation
- Add probationary mode toggle in admin settings
- Create moderation queue interface
- Add probationary status indicator components
- Update compose interface to show review notice
- Add approval/rejection action buttons for moderators

### Testing
- Unit tests for probationary account creation
- Integration tests for post queueing and approval
- Feature tests for moderation workflow
- API tests for all new endpoints

## Evaluation Criteria

### Code Quality (25%)
Clean implementation following Rails patterns and conventions. Proper service objects and separation of concerns. No security vulnerabilities (proper authorization checks).

### Completeness (35%)
All acceptance criteria met. Registration flow works correctly. Moderation queue functional. Account approval lifecycle complete. Federation properly restricted.

### User Experience (20%)
Clear messaging about probationary status. Intuitive moderation interface. Appropriate notifications. No confusion about post visibility.

### Testing (20%)
Comprehensive test coverage for all scenarios. Edge cases handled (account deletion, post cleanup, status transitions). API tests cover error cases.

## Files Likely Modified

### Models
- `app/models/account.rb` - Add probationary status
- `app/models/status.rb` - Add moderation queue support
- `app/models/setting.rb` - Add registration mode configuration

### Services
- `app/services/register_account_service.rb` - Support probationary mode
- `app/services/probationary_account_service.rb` (new) - Account lifecycle
- `app/services/post_moderation_service.rb` (new) - Post approval workflow

### Controllers
- `app/controllers/admin/settings_controller.rb` - Registration mode config
- `app/controllers/admin/moderation_queue_controller.rb` (new) - Queue management
- `app/controllers/api/v1/accounts_controller.rb` - Include probationary status

### Views
- `app/views/admin/settings/registrations.html.haml` - Add mode selector
- `app/views/admin/moderation_queue/` (new) - Queue interface
- `app/javascript/mastodon/features/compose/` - Add review notice

### Database Migrations
- Add probationary_status to accounts
- Add moderation_status to statuses
- Add registration_mode to settings

### Tests
- `spec/services/probationary_account_service_spec.rb`
- `spec/services/post_moderation_service_spec.rb`
- `spec/controllers/admin/moderation_queue_controller_spec.rb`
- `spec/features/probationary_registration_spec.rb`

## Security Considerations

- Ensure probationary users cannot bypass moderation
- Verify authorization checks for moderator actions
- Prevent privilege escalation
- Protect against spam/abuse of registration
- Audit log all moderation actions

## Success Metrics

- Reduced spam from new accounts
- Improved legitimate user retention (baseline: 19% vs target: >40%)
- Sustainable moderator workload
- Positive feedback from marginalized communities
- Successful federation restriction enforcement
