# envoy-plugin-example-access-control

This is an example Access Control integration with Envoy's integration builder. You can copy this repository and make adjustments to run your own access control integration.

## How to copy the repository

If you are familiar with GitHub, you simply need to fork this repository into your organization, and you'll be ready to make the necessary changes.

To fork a GitHub repo, simply go to the repo's main page (in this case, https://github.com/envoy/envoy-plugin-example-access-control) and find the "Fork" button toward the top right of the screen.

![GitHub fork button](readme-files/fork.png?raw=true "GitHub fork button")

If you have multiple organizations you belong to, GitHub will ask you which organization you want to fork it to. Select the organization, and you are on your way!

## System requirements

Your host machine will need to have [node installed](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm). It will also need to be able to make API calls to an existing access control server for your organization.

## How it works

This server application takes requests from Envoy to configure options and handle various events like signing in and signing out of a facility. It connects those requests from Envoy with your access control server to track and authenticate individuals. Take a look at the top-level index.js file. You'll see that there are two key dependencies, `@envoy/envoy-integrations-sdk` and `express`.

Express is a framework for running a node server application for the web. You can learn more about it [here](https://expressjs.com/). You can use any framework you like, but we recommend Express because it is minimal and popular, with strong documentation and community support.

The Envoy Integrations SDK for Node is a library that simplifies working with Envoy's integration builder. You can check out its documentation [here](https://github.com/envoy/envoy-integrations-sdk-nodejs#readme). It goes through a lot of detail of what each component of the library does, what is required for setup, and available features.

In this application, the URL handlers in index.js (each line with `app.post`) are split up into three sections: setup steps, field populators, and events.

The endpoint `/setup-step-authenticate` registers an app key and secret for this server to store for subsequent authentication.  On `reqeust.envoy`, there is an `installStorage` where the app key and secret are stored as `accessControlAppKey` and `accessControlAppSecret`.

The field populator endpoints tell Envoy what options should be enabled for its interface to the users. They either return static information that you can reconfigure or they communicate back with the Envoy API.

The event endpoints handle invites, sign-ins, sign-outs, employee sign-ins, and employee sign-outs. For each event, the endpoint handler may send various jobs back to Envoy via the `request.envoy.jobs` object and also communicate with your access control server to gather information or notify about events.

## Required changes to lib/AccessControlClient.js

This file handles all calls to your access control server. After forking the repo, there are a few changes you need to make to the file to get it working.

### Update baseURL

First, you will need to change all references to `baseURL` within usages of `axios`, a popular library for sending API requests.

The first `baseURL` is inside of `axios.create`. Change it from `https://api.exampleaccesscontrol.com/manage` to the URL where your access control server is located. Whether it has `/manage` or some other common path for the `baseURL` is to your discretion, based on how you have configured your endpoints on your access control server.

The second `baseURL` is in the `orgList` method. It assumes that fetching the org list is an endpoint not under `/manage` and thus overrides `baseURL`. If your org list endpoint has a path common to all the other endpoints, this `baseURL` override is unnecessary and you can remove it. Otherwise, replace it with an appropriate URL to reach your org list.

The third `baseURL` is in the `scheduleStatusPerson` method. It assumes that updating the schedule status of a person is also an endpoint not under `/manage` and thus overrides `baseURL`. If your endpoint for updating the schedule status of a person has a path common to all the other endpoints, this `baseURL` override is unnecessary and you can remove it. Otherwise, replace it with an appropriate URL to perform the update.

### Align endpoint paths with access control server

`AccessControlClient.js` calls the access control server at the following endpoints:
- `GET /v1/orgs`
- `GET /manage/sites`
- `GET /manage/groups`
- `GET /manage/groups/:id`
- `GET /manage/persons`
- `POST /manage/persons/import`
- `POST /manage/persons/:id/resend`
- `POST /manage/groups/:id/persons/:id`
- `POST /mobile-access/persons/:id/scheduled-statuses`

For each endpoint, you must either make sure the endpoint is defined on your access control server with the path, query parameters, method, body, behavior, and response that `AccessControlClient.js` is expecting, or you must modify the methods in this file to match the endpoints you have defined on your access control server. (Note to Chris: This is an area where I would expand the documentation a lot more, given the time. What it really needs is a full documentation of how each endpoint should work.)

## Optional changes

In `lib/handlers/fieldPopulator`, both `fieldPopulatorDuration.js` and `fieldPopulatorProtectDuration.js` have duration options that you may reconfigure to anything you would like. Each object in the list has a label (string) and a value (string), where the value is in minutes.

Although it won't impede your application from running, you will likely want to update the repository name and various fields in `package.json` related to the repository name to something that is your own.
