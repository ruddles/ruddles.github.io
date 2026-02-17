---
title: Mapping Repos in Wiz
published: true
---

I've been playing with a few different strategies for automatically assigning repos to projects in Wiz. Given we use projects and SSO mappings to limit access so engineers can only see issues linked to resources they own, this will give us the ability to show admins the findings of their repos.

Assuming you have a way to map teams to github teams, one option is to look at which teams are admins on the repo. The Resource Scopes on the projects are understandably limited and can't do this out the box, but we can then use Resource Tagging Rules to apply tags to the repos, and we can use a resource scope on the project to match the tag to the project terraform.

This approach has some flaws, such as it only works if there's exactly one admin per repo. If you need to support multiple admins per repo you can switch things around a bit by changing the tag key per team and ignoring the value. I'll cover that in another post. If you do have multiple admins per repo the tags will overwrite each other and it appears the last one wins.

## The Project File

For this example I'm going to work with the following JSON which defines some properties for a team. We'll make the `github_team` property optional to show how we can tackle it in terraform.

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
    "github_team": "team-b-gh"
},
{
	  "team_slug": "team-c",
    "team_name": "Team C"
}
...
]
```

Given this we can load the json into a local variable using the following:

```terraform
locals {
  # teams json loaded into a map
  teams = jsondecode(file("teams.json"))

  # Key for the tag to be applied to repos
  tag_key = "repo-admin-team"
}
```

## Resource Tagging

What we want to do is set up a resource tagging rule for each team, applying it to every repo object where that team is an admin.

We can us the lookup function to get the `github_team` property or an empty string if the property doesn't exist.

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
    key   = local.tag_key
    value = each.value.github_team
  }
}

```

## Project Resource Link

Finally we can loop through the collection to create a project for each team with a resource_filter_links object matching the `Wiz/repo-ownership-team` tag (custom tags in Wiz start with `Wiz/`).

Because in our example the `github_team` isn't mandatory, we need to make the `resource_filter_links` dynamic so we can skip it if the team isn't specified.

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
          key   = "Wiz/${local.tag_key}"
          value = each.value.github_team
        }
      }
    }
  }
}
```

