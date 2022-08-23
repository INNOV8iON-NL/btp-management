# Exercise 03 - Setting up autocomplete and initial account exploration

At the end of this exercise, you'll have autocomplete turned on for the btp CLI and have a general overview of your BTP account and what resources are available.

## Understand btp CLI command scope

If you've had a look at the output from `btp help` you'll know that there are multiple commands that the CLI supports, and different groups of objects upon which these commands operate:

```text
user: user $ btp help
Connecting to CLI server at https://cpcli.cf.eu10.hana.ondemand.com...
SAP BTP command line interface (client v2.14.0)

Usage: btp [OPTIONS] ACTION [GROUP/OBJECT] [PARAMS]

Each GROUP contains multiple OBJECTS, on which you can perform ACTIONS.

ACTIONS:
    list, get, create, update, delete, add, remove, assign, unassign, enable,
    move, register, unregister, subscribe, unsubscribe, share, unshare

GROUPS:
    accounts  Objects related to the account model, subscriptions, and environments
    security  Authorization objects and users
    services  Objects related to SAP Service Manager

Example help calls:
    btp help list                                 Commands for ACTION "list"
    btp help accounts                             Objects in GROUP "accounts"
    btp help accounts/available-environment       Actions for OBJECT "available-environment"
    btp help list accounts/available-environment  Command-specific help
    btp help all                                  Overview of all commands

General actions:
    help                  Display help
    login                 Log in to a global account of SAP BTP
    logout                Log out from SAP BTP
    target                Set the default context for command execution
    enable autocomplete   Enable command autocompletion
    disable autocomplete  Disable command autocompletion

Options:
  --config   Specify location of configuration file
  --format   Change output format (valid value: json)
  --help     Display help
  --info     Show version and current context
  --verbose  Print tracing information for support
  --version  Print client version

OK
```

It's possible to get an overview of what the possible targets of each command are, by asking for help on that particular command. Here's an example:

```text
user: user $ btp list --help
Connecting to CLI server at https://cpcli.cf.eu10.hana.ondemand.com...
SAP BTP command line interface (client v2.14.0)

Usage: btp [OPTIONS] ACTION GROUP/OBJECT [PARAMS]

Available "list" commands:
    btp list accounts/available-environment  Get all available environments for a subaccount
    btp list accounts/available-region       Get all available regions
    btp list accounts/custom-property        Deprecated. Show custom properties assigned to a subaccount or directory
    btp list accounts/entitlement            Get all the entitlements and quota assignments
    btp list accounts/environment-instance   Get all environment instances of a subaccount
    btp list accounts/label                  Show user-defined labels assigned to a subaccount or directory
    btp list accounts/resource-provider      List all resource provider instances
    btp list accounts/subaccount             List all subaccounts in a global account
    btp list accounts/subscription           Get all applications to which a subaccount is entitled to subscribe
    btp list security/app                    List all apps
    btp list security/role                   List all roles
    btp list security/role-collection        List all role collections
    btp list security/user                   List all users
    btp list services/binding                List all service bindings
    btp list services/broker                 List all service brokers
    btp list services/instance               List all service instances
    btp list services/offering               List service offerings
    btp list services/plan                   List service plans
    btp list services/platform               List all platforms

Example of command-specific help:
    btp help list accounts/available-environment

OK
```

> You can also use this style of help invocation too: `btp help list`.

But there's a much more comfortable way, and that's having the btp CLI itself suggest what's possible to you as you feel your way into what you want to do. This is a very common pattern and feature in shells such as Bash, and many command line programs offer such a facility, which is commonly referred to as "autocomplete".

## Set up autocomplete

In the output from `btp help` you'll have already seen how to do this: `btp enable autocomplete`. Asking for help on this (`btp enable autocomplete --help` or `btp help enable autocomplete`) will give you lots of details, one of which is that you need to specify the shell for which to enable the autocomplete feature. Within our Dev Space in App Studio, we're enjoying The-One-True-Shell™, i.e. [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) :-).

👉 Set up autocomplete now, selecting `/home/user/.bashrc` (option 2) for the RCFile:

```bash
btp enable autocomplete bash
```

Here's an example of such a setup:

```text
user: user $ btp enable autocomplete bash
This will install the autocomplete plugin script for bash to /home/user/.config/btp/autocomplete/scripts. Do you want to continue? [no]>yes
Which RCFile should be used for the installation?
1: /home/user/.bash_profile
2: /home/user/.bashrc
3: /home/user/.profile
4: Custom
Enter option>2
Autocomplete script for bash has been installed to /home/user/.config/btp/autocomplete/scripts.
You must start a new terminal session to activate the installed script.

Tips:
  Use the TAB key to complete commands and provide valid command actions, options, and parameters.
  Use the TAB key to cycle through the suggestion lists and the ENTER key to select.

OK
```

Note that a script has been installed to your btp CLI configuration directory, and the `.bashrc` file was chosen to contain the autocomplete setup (the invocation of that script on shell startup). This is recommended.

👉 Have a look what was added by looking at the last three lines of the file like this:

```bash
tail -3 $HOME/.bashrc
```

You'll see that there's an extra line that follows the two lines you yourself added in earlier exercises:

```bash
export PATH=$PATH:$HOME/bin
export BTP_CLIENTCONFIG=$HOME/.config/btp/config.json
SAP_BTP_CLI_AUTOCOMPLETE="/home/user/.config/btp/autocomplete/scripts/sapbtpcli-autocomplete.plugin.sh" && source $SAP_BTP_CLI_AUTOCOMPLETE
```

## Guard against permission expansion

Currently, if the Dev Space is restarted, permissions on the btp CLI configuration files are extended to add read/write access for the group; we don't want this. (This is not relevant if you're not working through this CodeJam in a Dev Space in the SAP Business Application Studio).

👉 So while you're looking at your `.bashrc` file, append a line like this:

```bash
echo 'chmod 600 $BTP_CLIENTCONFIG $SAP_BTP_CLI_AUTOCOMPLETE' >> $HOME/.bashrc
```

This will set the permissions (also known as "modes") on the btp CLI config files to 600, i.e. read and write for the user, and nothing for anyone else. For more on understanding these permissions, see [chmod](https://en.wikipedia.org/wiki/Chmod).

## Try out autocomplete

As with previous additions to your `.bashrc` file, you'll only see the effect on the start of a new instance of the shell. So to have autocomplete activated, you'll either have to restart your shell (close the terminal and start a new one) or you can also do what that last line does. Let's do the latter, to see how you can avoid having to close and start a new shell.

👉 Type this in right now:

```bash
SAP_BTP_CLI_AUTOCOMPLETE="/home/user/.config/btp/autocomplete/scripts/sapbtpcli-autocomplete.plugin.sh" \
  && source $SAP_BTP_CLI_AUTOCOMPLETE
```

Now you're ready to try the autocomplete feature out.

👉 Start by typing `btp` (followed by a space) and then hitting `Tab`. You'll first see the possible commands. Choose one by starting to type it and autocompleting it with `Tab`, and then choose from the list of GROUP / OBJECTs presented. You should spend a minute or two exploring like this - it's much easier to get a feel for autocomplete by trying it, rather than reading about it. But in case you want to read more about it, see the link to the blog post in the [Further reading](#further-reading) section below, which is from where this animated demonstration comes:

![animated demonstration of btp CLI autocomplete](https://blogs.sap.com/wp-content/uploads/2021/09/autocomplete.gif)

## Explore your BTP account

You have everything set up to be able to explore your BTP account; you can invoke the btp CLI from wherever you are, you're logged in, you have your global account and "trial" subaccount targeted, and you have autocomplete turned on for the most comfortable experience.

> The authentication token has a limited lifetime, so you may have to re-authenticate at some stage with `btp login`.

It's time to explore your account and related BTP resources.

👉 Spend a few minutes trying some of these commands, and examining the output:

|Command|Group/Object|Description|
|-|-|-|
|`get`|`accounts/global-account`|A high level summary of the key information for your global account; add the parameter `--show-hierarchy` to get a depiction of the relationship between your global account and directories & subaccounts within|
|`list`|`accounts/entitlement`|A list of the entitlements for your targeted subaccount|
|`list`|`accounts/available-region`|A list of the regions & providers available for your global account|
|`list`|`security/role-collection`|A list of role collections available in your account|

> Notice that the general pattern for the GROUP and OBJECT components is that where possible, GROUP is plural, but OBJECT is singular. There are a few exceptions but despite that, it's helpful to remember.

## Summary

At this point you have autocomplete working and you should feel comfortable invoking the btp CLI from the command line and exploring resources relating to your BTP account.

## Further reading

* [SAP Tech Bytes: btp CLI – autocompletion](https://blogs.sap.com/2021/09/21/sap-tech-bytes-btp-cli-autocompletion/)
* [SAP Help topic: Enable Command Autocompletion](https://help.sap.com/products/BTP/65de2977205c403bbc107264b8eccf4b/46355fab22814944bedf449a6c953369.html)

---

If you finish earlier than your fellow participants, you might like to ponder these questions. There isn't always a single correct answer and there are no prizes - they're just to give you something else to think about.

1. How would you discover or confirm that the shell you're actually using is indeed a Bash shell?
1. Output for some of the commands you used in exploring your account probably wrapped over multiple lines. How might you remedy that?
