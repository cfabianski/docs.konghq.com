---
title: Distributed Configuration for Kong using decK
no_version: true
---

decK can operate on a subset of configuration instead of taking care
of managing the entire configuration of Kong.

This can be very useful in a variety of scenarios. For example:

- Your organization uses a single Kong installation whose configuration is
  built from multiple teams. For example, one team manages the inventory API,
  while another team manages the users API and both APIs are exposed via Kong.
  In such a case, you want team inventory to manage it's configuration without
  worrying about the configuration of the other team. The two teams should
  be able to push out configuration changes as they see fit and when they see
  fit.
- Kong's configuration is very large, and you would like to manage it using
  different files, where you different parts take care of different problems.
- You want to declaratively manage all of the configuration for Kong except
  consumers and their credentials. The consumers are being managed by another
  service or you are dynamically creating them or the number of consumer is just
  so large that it makes no sense to manage them in a declarative fashion.

In such cases, you can use decK's `select-tag` feature to export, sync, reset
only a sub-set of configuration.

## Tags

Tags, introduced in Kong 1.1, provide a way to associate metadata with entities
in Kong. You can also filter entities by tags on the list endpoints in Kong.

Using this feature, decK associates tags with entities and can manage a group
of entities which share a common tag(s).

When multiple tags are specified in decK, decK `AND`s those tags together,
meaning only entities containing all the tags will be managed by decK.
You can specify a combination of up to 5 tags, but it is recommended to use
fewer or only one tag, for performance reasons in Kong's codebase.

## Dump

You can export a subset of entities in Kong by specifying common tags
using the `--select-tag` flag.

For example:

```shell
$ deck dump --select-tag foo-tag --select-tag bar-tag
# generates a kong.yaml file with all entities which have both the tags
```

If you observe the file generated by decK, you will see the following section:

```yaml
_info:
  select_tags:
  - foo-tag
  - bar-tag
```

This sub-section tells decK to filter out entities containing select-tags during
a sync operation.

## Sync

You don't need to specify `--select-tag` in `sync` and `diff` commands.
The commands will use the tags present in the state file and perform the diff
accordingly.

Since the state files contain the tagging information, different teams can
make updates to the part of configuration in Kong without worrying about
configuration of other teams. You no longer need to maintain Kong's
configuration in a single repository, where multiple teams need to
co-ordinate.

> The `--select-tag` flag is present on those two commands for use cases where
the file cannot have `select_tags` defined inside it. It is strongly advised
that you do not supply select-tags to sync and diff commands via flags.
This is because the tag information should be part of the declarative
configuration file itself in order to provide a practical declarative file.
The tagging information and entity definitions should be present in one place,
else an error in supplying the wrong tag via the CLI can break the
configuration.

## Reset

You can delete only a subset of entities sharing a tag using the `--select-tag`
flag on the `reset` command.

## Initial setup problem

When you initially get started with a distributed configuration
management, you will likely run into a problem where the related entities
you would like to manage don't share a single database.

To get around this problem, you can use one of the following approaches:

- Go through each entity in Kong, and patch those entities with the common
  tag(s) you'd like, then use decK's `dump` command to export by different
  tags.
- Export the entire configuration of Kong, and divide up the configuration
  into different files. Then, add the `select_tags` info to the file.
  This will require re-creation of the database now, since decK will not
  detect any of the entities present (as they are missing the common tag).
