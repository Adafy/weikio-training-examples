# Todos example – REST → Transform → REST

**Flow:** Timer → fetch comments (REST GET) → transform each comment into a todo → does the todo already exist? → create it (POST) or update it (PATCH)

Both the source and the target are the public test API [dummyjson.com](https://dummyjson.com), so you don't need any other services. Estimated time: about an hour. If you finish early, feel free to make your own changes to the integration.

The template is in the `todos-template` folder of the [Adafy/weikio-training-examples](https://github.com/Adafy/weikio-training-examples) repository. The integration in it is almost complete. The places you need to fill in are marked `TODO (1)`–`TODO (4)` in `example.camel.yaml`.

## Before you start

You need:

- the Weikio CLI and a login to a Weikio project and environment where you can publish integrations
- a secret named `training-api-token` in that project; any value will do, since dummyjson ignores it, but don't use a real credential because it is sent to a public service

In the commands below, replace:

| Placeholder | Meaning |
|---|---|
| `<name>` | your name or another identifier that is unique in the project, e.g. `alice`; your integration is then `todos-alice` |
| `<project>` | your Weikio project |
| `<env>` | your Weikio environment |

## Sample data

The source (`GET {{training-api-url}}/comments?limit=5&skip=251`) returns five comments in its `comments` list, ids 252–256. One comment looks like this:

```json
{
  "id": 252,
  "body": "Such a charming post.",
  "postId": 113,
  "likes": 7,
  "user": { "id": 57, "username": "nathand", "fullName": "Nathan Dixon" }
}
```

It is transformed into this todo:

```json
{
  "todo": "[nathand] Such a charming post.",
  "userId": 57,
  "completed": true
}
```

The comment id is saved in the `todoId` property, and the same id is used to check whether the todo already exists (`GET /todos/{todoId}`). Todos 252–254 exist (200) and are updated with PATCH. Todos 255–256 don't exist (404), so they are created with a POST to `/todos/add`.

## Steps

1. **Read and check the template:** Copy `todos-template` to a new folder `todos-<name>`. In `integration.yaml`, change the name `todos-example` to `todos-<name>`. Otherwise your integration clashes with everyone else's, and they can't run at the same time. The folder name doesn't affect anything, but it is good practice to name it after the integration. Then read through both routes and work out: which step fetches the comments, which transforms them, which creates or updates the todo, and how the existence check works? The check is in its own `direct:getTodo` route so that its 404 handling stays separate from the main flow.

2. **Split and transform:** The fetch returns several comments, and each must be handled on its own. At `TODO (1)`, write a Simple expression that returns the list in the response's `comments` field, e.g. `${body["…"]}`. `split` then processes each comment as its own message. At `TODO (2)`, add Groovy lines that set `todo.completed` to `true` when the comment has at least 5 likes (`body["likes"]`), and set `todo.userId` to the id of the comment's user (`body["user"]["id"]`). The existing `todo.todo` line shows the format.

3. **Fix create and update:** At `TODO (3)`, write the `choice` condition that is true when the todo was not found, i.e. when `${exchangeProperty.lookupStatus}` is `404`. Then fill in the `TODO (4)` places:
   - In the `when` branch, give the `to` step a `parameters` block that POSTs to `{{training-api-url}}/todos/add`. The `toD` step in the `getTodo` route shows the format.
   - In the `otherwise` branch, add `httpMethod: PATCH` to the `toD` step's parameters.

4. **Publish and start:** Validate, publish and deploy the integration with the commands below.

   ```powershell
   # 1. Validate – checks that the YAML is well-formed
   weikio validate -f .\todos-<name> --project <project>

   # (optional) run locally
   weikio dev run .\todos-<name> --env <env>

   # 2. Publish – uploads the integration to Weikio and builds it, does not start it
   weikio apply -f .\todos-<name> --project <project> --env <env>

   # 3. Deploy – installs the integration on the site and starts it
   weikio deploy <project>/todos-<name> --env <env>

   # 4. Monitor
   weikio status <project>/todos-<name> --env <env>
   weikio logs <project>/todos-<name> --env <env> --since 10m
   ```

   Running locally only works if your machine can reach every system the integration needs. It is useful for testing during development. The expected result in the log is three PATCH updates (252–254) and two POST creations (255–256). You can also deploy and check the logs in the Weikio UI.

5. **Cause an error on purpose:** Change the source `httpUri` to `https://dummyjson.com/http/404`, then run `apply` and `deploy` from step 4 again. The log should show `HttpOperationFailedException … statusCode: 404` from the source GET step. Think about it: why does this 404 stop the run when the 404 in `getTodo` does not? Finally, restore the original address by deploying the previous, working version again. The broken version stays in Weikio, but it no longer runs.

6. **Stop the integration:** Once the working version has run once, stop your integration with `weikio stop <project>/todos-<name> --env <env>`. Otherwise it keeps calling dummyjson every minute. Stopping deletes nothing: the versions stay in Weikio. If you have time, try deploying versions, stopping and starting in the UI too.

## You're done when

- The integration is published and has run.
- The log shows three PATCH updates (252–254) and two POST creations (255–256).
- You have caused an error by fetching from the 404 address and deployed the working version again.
- The integration is stopped.

## Good to know

- dummyjson only simulates changes and stores nothing: POST always returns id 255, and every run gives the same result.
- The timer runs once a minute, so the same lines appear in the log again every minute until you stop the integration.
- `{{training-api-url}}` is a property placeholder whose value (`dummyjson.com`) is in `application.properties`. `{{training-api-token}}` refers to the secret added to the project.
