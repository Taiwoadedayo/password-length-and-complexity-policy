# password-length-and-complexity-policy
Sample Service Desk Ticket

Ticket: User cannot open the Helpdesk shared folder.
Cause: User is not a member of Helpdesk-Team.
Resolution: Added the user to the group in Active Directory Users and Computers, asked them to sign out and back in, and confirmed access to the folder.

Account Lockout Lab

Goal: Practise one of the most common service desk tickets, a locked-out user, from setup to resolution.

Setup (on the domain controller):

	1.	Opened Group Policy Management and edited the Default Domain Policy.
	2.	Went to Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Account Lockout Policy.
	3.	Set Account lockout threshold to 3 invalid logon attempts and accepted the suggested duration and reset values (30 minutes).
	4.	Ran gpupdate /force to apply the policy.

Reproduce the problem (on the Windows 11 client): entered a wrong password for a domain user 3 times. The next login attempt showed: "The referenced account is currently locked out and may not be logged on to." Even the correct password was refused.

Resolve it (on the domain controller):

	1.	Opened Active Directory Users and Computers and located the user.
	2.	On the Account tab, ticked Unlock account, then clicked Apply and OK.
	3.	Confirmed the user could log in again with the correct password.

Ticket write-up:

	•	Ticket: "My password stopped working."
	•	Cause: Account locked out after too many failed logon attempts.
	•	Resolution: Verified the caller's identity, unlocked the account in Active Directory, and confirmed login. If the user had forgotten the password, reset it as well.

Things I noticed:

	•	Lockouts are enforced by the domain controller, so the policy applies to every domain-joined machine.
	•	The login name (asmith) differed from the display name (angeli smith). Typing the wrong username is a common cause of "my password doesn't work" tickets.
	•	Always verify identity before unlocking. Attackers can trigger lockouts on purpose, and some call the help desk pretending to be staff.

Group Policy Mapped Drive Lab

Goal: Give the Helpdesk team an automatic network drive without touching each PC.

Setup (on the domain controller):

	1.	Created a new GPO named Map Helpdesk Drive and linked it to the taiwo staff OU.
	2.	Went to User Configuration > Preferences > Windows Settings > Drive Maps and created a mapped drive pointing at \\WIN-SERVER02\helpdesk with drive letter H:.
	3.	Used Item-level targeting to apply it only when the user is a member of the TAIWO\helpdesk-team security group.

Testing (on the Windows 11 client):

	•	A user in Helpdesk-Team signed in and H: appeared under Network locations.
	•	A user outside the group signed in and no H: drive appeared.

Troubleshooting: gpupdate /force initially failed with a clock synchronisation error. Active Directory and Group Policy need the client and domain controller clocks to match, and the server was on a different time zone. Fixing the time zone and time resolved it. This is a good example of a ticket where "Group Policy isn't applying" is not a policy problem.

Why it matters: One policy now controls drive access for a whole group. New staff only need adding to the group and the drive appears at their next login.

Password Policy Lab

Goal: Enforce sensible password rules across the whole domain and confirm weak passwords are blocked.

Setup (on the domain controller):

	1.	Opened Group Policy Management and edited the Default Domain Policy.
	2.	Went to Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy.
	3.	Set Minimum password length to 10, confirmed Password must meet complexity requirements was Enabled, and set Maximum password age to 90 days.
	4.	Ran gpupdate /force.

Testing:

	•	Reset a user's password to a weak value (abc123). Active Directory refused it with: "The password does not meet the password policy requirements. Check the minimum password length, password complexity and password history requirements."
	•	Reset it to a strong password. Active Directory accepted it: "The password for the user has been changed."

Service desk note: this error is what a user sees when they pick a password that is too short or too simple. The fix is to explain the rules (length, mixed characters) and have them choose a stronger one.
