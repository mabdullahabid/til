# Webmail Internal Server Error after migration

After backing up and restoring a user with multiple domains to a new host, and configuring the DNS to point to the correct IP address, the webmail interface displayed an internal server error. After going through HestiaCP forums, I found that this is a permissions issue.

Here's the fix:

Change permissions to php files in /etc/roundcube/

    find /etc/roundcube/ -type f -iname "*php" -exec chmod 644 {} \;

Once done, try to access webmail.

Edit: As there are passwords inside config files, this should be a better approach to fix the issue:

    chown -R hestiamail:hestiamail /etc/roundcube/
    find /etc/roundcube/ -type f -iname "*php" -exec chmod 640 {} \;

You should be able to access webmail after.

Reference

https://forum.hestiacp.com/t/webmail-internal-server-error/12073/7