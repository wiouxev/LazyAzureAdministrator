# Welcome to the Lazy Azure Administrator Project!

Azure Administrator (AA) is a simple GUI application designed to handle most of your help desk needs. It currently focuses on user and group management, but future updates I plan to bring will add even more features such as mailbox management and AAD/Intune device management!

# How to Install
AA is lightweight and extremely simple to install. Launch the MSI manually and click through the setup wizard, or install it programatically using the following command:
`msiexec /i AzureAdmin.msi /qn`

# Environment deployment pre-requisite(s)
Currently the only pre-requisite to deploying AA to your environment is you'll need to set up a registered application in your AAD tenant. Microsoft has a really easy and straightforward guide on this: https://docs.microsoft.com/en-us/graph/auth-v2-user as well as this article on how to set up delegated API permissions: https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-configure-app-access-web-apis NOTE: The redirect I use for AA in my environment: https://login.microsoftonline.com/common/oauth2/nativeclient
Once you have your application set up, note your ApplicationID (also called ClientID) and enter it in on first run.

# First Run
The first time you launch the application, it will do the following tasks:
* Check for log folder and create if not exist
* Check for MSAL.PS module and install if not installed

All modules will be locked until you Log In. On first run, clicking the Log In button will open the GetClient form for the user to paste the ClientID into (Author's note: While I wouldn't share your application's ID with the world, it doesn't necessarily need to be kept in a top-secret vault. As long as you have the *Who can use this application or access this API?* setting for your registered application set to *Accounts in this organizational directory only*, only accounts from your tenant can authenticate to the app or API.)
Once you've successfully authenticated to your AAD tenant, the other modules will be unlocked. Do note that if MFA is enabled on your account, you'll need to approve the login request.

# Module Functionality
* Assign License: Accepts user's email as input and assigned checked licenses to the user. (BUG NOTE: Currently when trying to assign multiple licenes at once, the task may fail due to Graph API limiting. Currently investigating.)
* Add User to Group: On form load, gets all groups in your organization. Accepts user's email as input and add's user to any of the selected groups (can use ctrl click to select multiple groups)
* Create Assigned Group: Accepts incipient group's displayName (required), mail nickname (required; no spaces is best, but use %20 for space if needed). Based on what options are checked, can create one of three types of groups: 1. a standard, statically assigned security group. Mail not enabled, because not an M365 group. 2. M365 group, mail disabled, or 3. M365 group, mail enabled
* Create New User: Simple to fill out form to input incipient user's information on. Only requires first and last name to create the account. Allows for assignment of licenses (subject to the same bug mentioned above) and assignment to group(s). Random 12-character password will be generated and copied to your clipboard
* Disable User: Accepts user's email address and disables user
* Edit User: Currently able to edit 7 fields for user: Given name, surname, title, office, manager email, moible phone, and department
* Enable User: Accepts user's email address and enables user
* Remove from Group: Accepts user's email as input, loads user's groups, and removes user from any selected groups
* Reset Password: Accepts user's email address as input, as well as new password (if you're not using the Random buttonthat is, which is generally more secure than most help desk assigned passwords I've run into in my times)
* Terminate User: Accepts user's email address as input and terminates user. Following actions are taken: 1. User disabled 2. Z_Term_ added to the front of the user's display name. 3. Removes user from all static groups (cannot remove from dynamic groups). 4. Removes all assigned licenses to the user. (Author's note: As I introduce mailbox management, I plan on adding an optional parameter to let you convert user boxes to shared mailboxes for termed users.)

# Functionality Notes
* All forms requiring user email address for input will have the action button disabled until the email address field contains a string with '.com', '.edu', or '.gov'. If there are any other TLDs you'd like to see added, please submit a feature request!
* Microsoft license options available in AA can be expanded; I have selected the most used (in my experience) to start with, but if you have other licenses you'd like to see, please submit a feature request *along with the license SkuID.* The SkuID can be found in several ways, including querying with Graph: `GET https://graph.microsoft.com/v1.0/subscrikedSkus`, or with the Microsoft.Graph.Identity.DirectoryManagement module command `Get-MgSubscribedSku | select skuPartNumber, SkuId | Format-List`

# Required delegated Graph API permissions
Note: only one is required from each list. Each list is also sorted from least to most privileged permissions for better clarity. See https://docs.microsoft.com/en-us/graph/permissions-reference for more/better information on Graph API permissions
* Assigning licenses: User.ReadWrite.All, Directory.ReadWrite.All
* Adding user to group(s): GroupMember.ReadWrite.All, Group.ReadWrite.All, Directory.ReadWrite.All, Directory.AccessAsUser.All
* Creating assigned groups: Group.ReadWrite.All, Directory.ReadWrite.All, Directory.AccessAsUser.All
* Creating new user: 	User.ReadWrite.All, Directory.ReadWrite.All, Directory.AccessAsUser.All
* Disable user, Edit user, Enable user, Reset password:: User.ReadWrite, User.ReadWrite.All, User.ManageIdentities.All, Directory.ReadWrite.All, Directory.AccessAsUser.All
* Remove from group: GroupMember.ReadWrite.All, Group.ReadWrite.All, Directory.ReadWrite.All, Directory.AccessAsUser.All
* Terminate user: each of the above 
