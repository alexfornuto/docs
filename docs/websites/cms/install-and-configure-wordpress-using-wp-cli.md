---
author:
    name: Linode Community
    email: docs@linode.com
description: 'How to install and configure WordPress using WP-CLI on Ubuntu 14.04'
keywords: 'install WP-CLI,ubuntu,wordpress'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
published: 'Saturday, October 3rd, 2015'
modified: Saturday, October 3rd, 2015
modified_by:
    name: Linode
title: 'Install and Configure WordPress using WP-CLI'
contributor:
    name: Navjot Singh
    link: https://github.com/navjotjsingh
external_resources:
 - '[WP-CLI Commands](http://wp-cli.org/commands/)'
 - '[WP-CLI Community Commands](https://github.com/wp-cli/wp-cli/wiki/List-of-community-commands)'
---

*This is a Linode Community guide. [Write for us](/docs/contribute) and earn $250 per published guide.*

<hr>


Everyone is probably familiar with WordPress and its famous 5 minute install routine. Its simple and works without fuss. But when you have multiple sites to manage, going over the same routine can take up lot of time which you could have used elsewhere.

This is where WP-CLI comes in. It is a command line tool which comes with lots of powerful features with which you can install, manage, update and do lots more with WordPress. This tutorial will cover how to install WP-CLI and how to perform some common tasks using it.

## Prerequisites
Before we move ahead, make sure you have completed the following guides:

* [Getting Started with Linode](/docs/getting-started)
* [How to Install a LAMP Stack on Ubuntu 14.04](/docs/websites/lamp/how-to-install-a-lamp-stack-on-ubuntu-14-04)
* [Securing your Server](/docs/security/securing-your-server)

{: .note}
>
>This guide is written for a non-root user. Commands that require elevated privileges are prefixed with `sudo`. If you're not familiar with the `sudo` command, you can check our [Users and Groups](/docs/tools-reference/linux-users-and-groups) guide.

## Installing WP-CLI

1.  WP-CLI is available as a PHP Archive file(phar). You can download it using either wget or curl commands.

        curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar

    Or

        wget https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar

2.  You need to make this .phar file executable and move it to `/usr/local/bin` so that it can be run directly.

        chmod +x wp-cli.phar
        sudo mv wp-cli.phar /usr/local/bin/wp

3.  Check if it is installed properly.

        wp --info

    You should see a similar output like the one below which means we can move ahead.

        PHP binary:	/usr/bin/php5
        PHP version:	5.6.11-1ubuntu3.1
        php.ini used:	/etc/php5/cli/php.ini
        WP-CLI root dir:	phar://wp-cli.phar
        WP-CLI global config:
        WP-CLI project config:
        WP-CLI version:	0.21.1

You can use the above procedure for upgrading WP-CLI as well.

### Activating Bash-Completion

Now it is difficult all commands at once and you might want a quick reference on the command you need. While you can always type `wp` and press enter to see the complete list. But wouldn't it be better if you can point to specific command. Like if you want to know what all sub commands are under the `core` parameter. Bash completion script of WP-CLI can help here.

1.  To install the script, make sure you are in the home directory of the user(`username` in this case) you are using.

        cd /home/username
        wget https://github.com/wp-cli/wp-cli/raw/master/utils/wp-completion.bash

2.  Now edit the .bashrc so that it is loaded by the shell every time you login. Open the file and add the following line in the editor assuming you downloaded the file in the home directory.

    {: .file-excerpt}
    /home/username/.bashrc
    :   ~~~ bash
    source /home/username/wp-completion.bash
    ~~~

3.  Now run the following command to reload the Bash profile.

        source ~/.bashrc

That's it. Bash Completion is enabled.

## Basics of WP-CLI
Before we get into the helm of the things, let us learn some essential basics of how exactly WP-CLI works. This would help you feel comfortable with what is going to follow next.

So far we have seen WP-CLI is accessed through the keyword `wp`. What follows that is a set of commands and their own sub commands. For example we have a command to download WordPress which goes like `wp core download`. Here wp is the main command while core and download are its sub-commands. There can be several sub-commands for each sub-command as well.

WP-CLI also comes with a pretty detailed help section which will allow you to keep tab on all the commands you would need. You can access it by typing

    wp help

Do that and you will get a screen similar to

    wp

    DESCRIPTION

    Manage WordPress through the command-line.

    SYNOPSIS

    wp <command>

    SUBCOMMANDS

    cache               Manage the object cache.
    cap                 Manage user capabilities.
    cli                 Get information about WP-CLI itself.
    comment             Manage comments.
    core                Download, install, update and otherwise manage WordPress proper.
    cron                Manage WP-Cron events and schedules.
    db                  Perform basic database operations.
    eval                Execute arbitrary PHP code after loading WordPress.
    eval-file           Load and execute a PHP file after loading WordPress.
    :


`:` is a prompt where you need to enter few commands to navigate through this help menu. Up and down arrow keys will take you through the complete help. And pressing q will exit the help menu. That's all you need to know for now. For complete detail on how to navigate through the complete help section, you can always type `h` at the above prompt.

In our last step, we enabled Bash Completion feature for WP-CLI. To use that type `wp` and press tab twice. You would see the list of available commands. You can repeat the same by typing `wp core` and then pressing tab twice. Now you will see a list of commands that can be used with `core`.

## Installing WordPress

### Setting up Database

1.  Before you proceed, you need to setup a database first. For that, login to the MySQL server first. Replace `user` with your MySQL user.

        mysql -u user -p

2.  Next step is to create a database.

        create database wordpress;

3.  Now we need to grant all the required privileges for the database to the mysql user.

        grant all on wordpress.* to 'user' identified by 'password';

4.  Type `quit` to exit the MySQL command line.

### Main Install

1.  Move to the user directory where you would be installing the blog. Traditionally it should be in `/var/www/html` directory. Create a new user directory for your user if it doesn't exist already. Also proper group and ownership permissions need to be given to the user.

        cd /var/www/html
        sudo mkdir wpblog
        sudo chown -R user:user wpblog
        cd wpblog

    {: .note}
    >
    >This guide for the sake of simplicity assumes that you will be hosting only one site i.e. the blog on your VPS and therefore doesn't go into the detail of setting up a virtual host file. Should you need to host multiple sites, you will need to refer to our [guide on Apache Virtual hosts](/docs/websites/apache/apache-web-server-on-ubuntu-14-04#configure-name-based-virtual-hosts) for the same.

2.  Next, download the WordPress files.

        wp core download

3.  Create a wp-config.php file.

        wp core config --dbname=Database --dbuser=user --dbpass=password --dbhost=localhost --dbprefix=wp_

    dbhost and dbprefix are entirely optional and can be omitted unless you need to change their default values.

4.  Run the installation.

        wp core install --url="<yourdomain>" --title="Blog Title" --admin_user="adminusername" --admin_password="password" --admin_email="emailid"

Your WordPress blog is ready for use.

## Common Commands

### Installing and Updating Plugins

Let's say we need to install Yoast SEO plugin. First step is to search for it to find the slug.

    wp plugin search yoast

You will get an output similar to this.

    Success: Showing 10 of 259 plugins.
    +---------------------------------+----------------------------------+--------+
    | name                            | slug                             | rating |
    +---------------------------------+----------------------------------+--------+
    | Yoast SEO                       | wordpress-seo                    | 90     |
    | SO Clean Up Yoast SEO           | so-clean-up-wp-seo               | 96     |
    | All Meta Stats Yoast SEO Addon  | all-meta-stats-yoast-seo-addon   | 100    |
    | Google Analytics by Yoast       | google-analytics-for-wordpress   | 80     |
    | Import Settings into WordPress  | yoast-seo-settings-xml-csv-impor | 0      |
    | SEO by Yoast                    | t                                |        |
    | Surbma - Yoast Breadcrumb Short | surbma-yoast-breadcrumb-shortcod | 84     |
    | code                            | e                                |        |
    | Meta Box Yoast SEO              | meta-box-yoast-seo               | 0      |
    | Keyword Stats Addon for Yoast S | keyword-stats-addon-for-yoast-se | 100    |
    | EO                              | o                                |        |
    | Meta Description Stats Addon fo | meta-description-stats-addon-for | 100    |
    | r Yoast SEO                     | -yoast-seo                       |        |
    | Title Stats Addon for Yoast SEO | title-stats-addon-for-yoast-seo  | 100    |
    +---------------------------------+----------------------------------+--------+

You can see more than 10 per page by modifying the command as

    wp plugin search yoast --per-page=20

Now that you know the slug of the plugin we want to install, install and activate it.

    wp plugin install wordpress-seo
    wp plugin activate wordpress-seo

If you want to update any plugin, you can use

    wp plugin update wordpress-seo

Or if you want to update all, then

    wp plugin update --all

To list all the installed plugins on your blog, use

    wp plugin list

To uninstall a plugin, use

    wp plugin uninstall wordpress-seo

### Installing and Updating Themes

The procedure for installing and activating a theme is quite similar to that of the plugin. Just the keyword `plugin` changes to `thene` in all the commands.

So to search for the theme, you would use

    wp theme search twentytwelve

To install and activate, you need to use

    wp theme install twentytwelve
    wp theme activate twentytwelve

And to update one or all themes, you can use

    wp theme update twentytwelve
    wp theme update --all

To list all the themes in a tabular form, you can use the command `list`.

    wp theme list

To uninstall a theme, use

    wp theme uninstall twentytwelve

### Update WordPress

You can update your blog via following commands.

    wp core update
    wp core update-db

First command updates the files and second one completes the database upgrade.

## Conclusion

This wraps up our tutorial on WP-CLI. These commands are just tip of the iceberg of what you can do with it. You can write or edit posts, perform database queries, manage user capabilities, manage cron events, import or export content, manage attachments and even manage multi-site installations.
