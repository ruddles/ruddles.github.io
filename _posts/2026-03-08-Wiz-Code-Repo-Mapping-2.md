---
title: Mapping Repos with Multuple Admins in Wiz
published: true
---

In the [previous post](https://ruddles.dev/Wiz-Code-Repo-Mapping) we talked about how we can use Resource Tagging Rules to map repos to teams in Wiz. The major limit in the approach outlined there is when a repo has multiple admins. When that happens the value for the `Wiz/repo-admin-team` will get overwritten by each run of the resource tagging rules, so the last one would win.

Tags in Wiz don't allow multiple values, so we need to take a different approach. You can actually have a lot of tagging rules in Wiz (there's a limit in the docs but it's in the 1000's), so instead of setting the team name in the value we can set it in the key and ignore the value for our mappings. And that's exactly what we'll do here.

## The Project File

For this project file we're going to make one small tweak to demonstrate an issue with using team names in the tag key in Wiz. We're still matching team slug to team name in Wiz so you may still get some mismatching until they add the slug, but we're going to tweak Team B so it has an underscore character. It's valid in team slugs in github.

```json
[
{
    "team_slug": "team-a",
    "team_name": "Team A",
    "github_team": "team-a-gh"
},
{
    "team_slug": "team-b",
    "team_name": "Team B",
    "github_team": "team-b_gh"
},
{
    "team_slug": "team_c",
    "team_name": "Team C"
}
...
]
```

So now instead of having the `tag_key` variable we have a prefix we can use before the team slug:

```terraform
locals {
  # teams json loaded into a map
  teams = jsondecode(file("teams.json"))

  # Key for the tag to be applied to repos
  tag_key_prefix = "repo-admin-"
}
```

## Resource Tagging

Now the Resource Tagging Rule is pretty much the same, but for the tag we set the key to the prefix and then the github team.

Something to note is that you can't have underscores in the key, but you can in the github team name so we'll need to swap these out for hyphens using `replace`. We can do the same in the project mapping to ensure they match up.

```terraform
resource "wiz_resource_tagging_rule" "repo_owning_team" {
  for_each = { for t in local.teams : t.team_slug => t if lookup(t, "github_team", "") != "" }

  # basic settings for the rule
  description = "Tag repos where the ${each.key} github team are admins"
  enabled     = true
  name        = "Repository Admin - ${each.key}"

  # Query finds all repos which have the given github team as admins
  query = jsonencode({
    relationships = [{
      type = [{
        reverse = true
        type    = "APPLIES_TO"
      }]
      with = {
        relationships = [{
          type = [{
            type = "PERMITS"
          }]
          with = {
            select = true
            type   = ["ACCESS_ROLE"]
            where = {
              name = {
                EQUALS = ["Admin"]
              }
            }
          }
          }, {
          type = [{
            type = "ASSIGNED_TO"
          }]
          with = {
            select = true
            type   = ["GROUP"]
            where = {
              name = {
                EQUALS = [each.value.github_team]
              }
            }
          }
        }]
        select = true
        type   = ["ACCESS_ROLE_BINDING"]
      }
    }]
    select = true
    type   = ["REPOSITORY"]
  })

  # Tag key and value, this will be prefixed with Wiz/
  tags {
    key   = "${local.tag_key_prefix}${replace(each.value.github_team, "_", "-")}"
    value = ""
  }
}

```

## Project Resource Link

Finally for the project mapping we just do the same, matching on just the key with an empty value, remembering to replace \_ with -.

```terraform
resource "wiz_project" "team_projects" {
  for_each = { for t in local.teams : t.team_slug => t }

  # Basic project values
  name = each.value.team_name
  slug = each.value.team_slug

  # Add a resource filter matching the tag applied above
  # only if the team have a github team specified
  dynamic "resource_filter_links" {
    for_each = lookup(each.value, "github_team", "") != "" ? [each.value.github_team] : []

    content {
      environment = "PRODUCTION"
      resource_tags_v2 {
        equals_all {
          key   = "${local.tag_key_prefix}${replace(each.value.github_team, "_", "-")}"
          value = ""
        }
      }
    }
  }
}
```

Once this has run in we'll see that repos in Wiz with multiple teams will have multiple tags, and can therefore be matched to multiple projects.
