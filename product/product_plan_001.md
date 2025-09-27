
# Product Plan 

## Brainstorming & Backlog

- Public pages : guests can explore groups and events
- Membership feature 
- - Member profile
- - Display name (FirstName L, Full Name, etc)
- - - Member history/timeline, statistics (how many events hosted, attended, etc)
- Vetting Process
- Role based permissions 
- Groups 
- - Badges (trusted member, top contributer, loyal member, etc)
- - - membership reputation score
- - do we want private groups? private invite links, etc.
- Events 
- - Create Event
- - - trusted members can suggest events, which will be reviewed & approved by an admin. 
- - View All Events
- - 'Events for Me'
- - Show Event
- - - Event share feature (markdown export \w cool info emojis and such) 
- - - Metadata for previews.
- Notifications 
- - Email, Mobile Push Notifications
- - Daily Email: A summary of the personalized events. (Configurable)
- Chat 
- - Group Chats
- - Event Chat
- - DM (Live Chat?)
- - - DM Policies. Who can DM who, and so on.
- Report Abuse 
- - Report bad behavior, bad events, places, whatever. Abuse reports should be polymorphic. 
- Search feature
- Mobile-first UI
- Privacy
- - group based profile visibility settings
- - 
- - - 


# Personas / Roles
- Users
- - Guests 
- - Registered users 
- - - Group members 
- - - Group admins 
- - - Event Hosts 
- - - Event Participant
- Superadmin
- Developer


# MVP (Minimum Viable Product) Plan
- Basic landing page: smileys logo, upcoming events, featured groups, smileys social links
- User Registration, Signin, Forgot/Reset Password, Logout. 
- - Social Signup/Login Options. (Facebook/Google)
- Role Based Access Control 
- - Superadmin 
- - Groupadmin 
- Groups CRUD (API, Front-end () )
- - Superadmins can create groups. Superadmins can add members to groups and assign groupadmins.
- Events CRUD
- - Superadmins and group admins can create events
- - - (Title, Description, Maps URL, ...)
- - Any user can view a single event page. 
- - - Event page must display Google map
- - - Any group member can RSVP to an event.
- - - - Event participants get a QR Code ticket. 
- - - - Event Hosts/Organizers can scan the QR code to verify participation.
- My Profile
- - My Groups, My Events
- - Account Settings (edit email, password, phone number)
- Terms of Use, Privacy Policy
