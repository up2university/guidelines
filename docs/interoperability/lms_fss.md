# Interoperability between LMS and FSS

Interoperability between LMS (Moodle) and FSS (Nextcloud)
results in users having access to their FSS files from Moodle courses.
The integration is to be configured as follows.

In Nextcloud, go to "Administration"->"Security" (`/settings/admin/security`) and register Moodle as OAuth client.
Set a visible service name, e.g. "Up2U Moodle", and the Moodle's redirect URL.
OAuth client identifier and secret will be generated, and these are put to the configuration in Moodle.

In Moodle, go to "Site administration/Server/OAuth 2 services" and create OAuth 2 service. 
Set the service name, e. g. "Nextcloud repository", CliendID, Service base URL (Nextcloud URL) and scopes.
Then activate the Nextcloud plugin in "Site administration/Plugins/Repositories". 
From drop-down list choose "Enabled and visible". 
Open settings and select the previously configured OAuth 2 service as Issuer. 
Check "Allow users to add a repository instance into the course".
