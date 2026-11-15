# Acuity WordPress User Import

Connects a Backdrop CMS installation to a legacy WordPress database to migrate user accounts and metadata. It is part of the **Acuity WordPress Migration Centre** and uses that module's shared database connection.

This module is designed to be an idempotent migration tool: it can be run multiple times to sync changes from WordPress to Backdrop, updating existing users and creating new ones as needed.

When all users have been imported and/or your new Backdrop site is live, you can safely uninstall this module.

To manipulate the user accounts we recommend using the **Views Bulk Operations** module, for example, to change the user role from the standard *Authenticated* to a custom role you may have created, or to update an email or newsletter subscription status.

We recommend deleting any unneeded fields we created for the import process once you are happy with the imported data.

## Initial version

This is a beta release for code review and testing.

## Requirements

- Backdrop CMS 1.x
- **Acuity WP Migration Centre (acuity_wpmc)** — provides the WordPress database connection and the migration dashboard this module registers on.
- A secondary database connection to the WordPress database, defined in settings.php.

## Installation

Install this module using the official Backdrop CMS instructions at https://docs.backdropcms.org/documentation/extend-with-modules

Add the WordPress database configuration to your settings.php file (see Documentation below), then enable the module (it will enable the Migration Centre as a dependency).

## Documentation

1. **Database Configuration**

  Before running the migration, define the connection to the WordPress database in your site's settings.php. The prefix must be empty to query `wp_users` directly. `host` may be `127.0.0.1`, `localhost`, or a remote database host.

  ```
  $databases['wordpress']['default'] = array(
    'driver' => 'mysql',
    'database' => 'your_wordpress_db_name',
    'username' => 'db_user',
    'password' => 'db_pass',
    'host' => '127.0.0.1',
    'prefix' => '',
  );
  ```

2. **Running the Migration**

  - Open the Acuity Migration Centre dashboard at Configuration → Acuity Migration Centre (`admin/config/acuity-migration`).
  - Under **WordPress Migrations**, choose **User Migration & Auth**.
  - On the **Settings** tab, confirm the batch size (number of users processed per step).
  - On the **Overview** tab, check the prerequisite checklist, then click **Run User Migration** to begin the batch import. The process is idempotent and can be re-run safely.

3. **Data Mapping**

  The module automatically creates hidden fields to store legacy data.

  | WordPress Field | Backdrop Field | Visibility | Description |
  | --- | --- | --- | --- |
  | ID | field_wp_guid | Hidden | Integer. Used for syncing. |
  | user_login | name | Public | Unique username. |
  | user_email | mail | Private | Unique email. |
  | user_pass | pass | Private | Raw injection of legacy hash. |
  | display_name | field_display_name | Public | Preferred public name. |
  | first_name | field_first_name | Public | Meta field. |
  | last_name | field_last_name | Public | Meta field. |
  | description | field_wp_bio | Public | Biographical info. |
  | wp_user_level | field_wp_user_level | Hidden | Legacy permission level. |
  | wp_capabilities | field_wp_capabilities | Hidden | Serialized roles array. |

## Logging in after migration

**Migrated users log in with their existing WordPress password.**

- At import, each user's WordPress password hash is copied verbatim into Backdrop's `users` table.
- Older WordPress hashes (```$P$```…, or the phpBB ```$H$```… variant) are verified by **Backdrop core itself** at the user's first login, which then re-saves the password as a Backdrop hash (```$S$```…). The user notices nothing.
- **WordPress 6.8 and later** store passwords as bcrypt (```$wp$2y$```…), which Backdrop core does not understand. This module checks those at login the way WordPress does and, when the password is right, re-saves it as a Backdrop hash; core then logs the user in as normal. Flood control applies as usual. **Keep this module enabled until no accounts have such a password** (the Overview tab shows how many are left); a user still on one cannot log in without it, only reset their password.

Safeguards:

- **Non-destructive re-runs:** re-running the migration never overwrites a password that has already been upgraded to a Backdrop ```$S$``` hash.
- **User 1:** the Backdrop Root Administrator (User 1) password is never touched by the import.
- **Blocked members:** bbPress members flagged `bbp_blocked` are imported as **disabled** accounts, not active ones (the block lives in `wp_capabilities`, not `user_status`).

## Security Implications & Safeguards

### Field Access

Technical fields (`field_wp_guid`, `field_wp_capabilities`, `field_wp_user_level`) are restricted via *hook_field_access()*. Only administrators with the *administer users* permission can view or edit these fields.

### Silent Import

The module explicitly suppresses "Account Created" emails (`$account->notify = FALSE`) to prevent mass emailing during import.

### Content Risks (XSS)

Bio data (`field_wp_bio`) is imported raw from WordPress. If the source site contained malicious scripts in user bios, they are now in your database. Ensure the Text Format used to display this field filters HTML tags appropriately.

## Issues:

Bugs and Feature requests should be reported in the Issue Queue: https://github.com/backdrop-contrib/acuity_wpuser/issues

## Current Maintainer(s):
- Steve Moorhouse (albanycomputers) (https://github.com/albanycomputers)
- Seeking additional maintainers and contributors.

## Credits:

- Assisted by AI

## Sponsorship:
- Albany Computer Services (https://www.albany-computers.co.uk)
- Albany Web Design (https://www.albanywebdesign.co.uk)
- Albany Hosting (https://www.albany-hosting.co.uk)

## License
This project is GPL v2 software. See the LICENSE.txt file in this directory for complete text.
